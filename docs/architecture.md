# TLSOC Architecture

This document describes the architecture of the TLSOC platform across all its
repositories. Component-level internals are documented in each repository's own
`docs/` directory; this is the ecosystem-wide view.

## Contents

- [High-level architecture](#high-level-architecture)
- [Component architecture](#component-architecture)
- [Data flow](#data-flow)
- [Deployment topology](#deployment-topology)
- [Communication and ports](#communication-and-ports)
- [Kafka flow](#kafka-flow)
- [Parser pipeline](#parser-pipeline)
- [Reporting pipeline](#reporting-pipeline)

## High-level architecture

TLSOC is a pipeline of four cooperating systems: agentless collection, TLS-secured
transport and storage, ECS normalization, and reporting.

```mermaid
flowchart LR
    SRC["Monitored servers<br/>rsyslog + omkafka"] -- "TLS :9094" --> K["Apache Kafka"]
    K --> ENG["tlsoc-engine<br/>parse + normalize (ECS)"]
    ENG -- "NDJSON files" --> LS["Logstash"]
    LS --> ES["Elasticsearch"]
    ES --> KB["Kibana dashboards"]
    ES --> REP["tlsoc-reporting<br/>HTML / PDF reports"]
    REP --> MAIL["Email delivery"]
```

## Component architecture

```mermaid
flowchart TB
    subgraph src["Log sources (any Linux server)"]
        LOGFILES["Log files<br/>/var/log/…"]
        IMFILE["rsyslog imfile<br/>(inotify tail)"]
        OMKAFKA["omkafka output<br/>JSON envelope template"]
        LOGFILES --> IMFILE --> OMKAFKA
    end

    subgraph deploy["tlsoc-docker-deploy (Docker Compose, one host)"]
        KAFKA["Kafka broker<br/>internal :9092 / external TLS :9094"]
        LOGSTASH["Logstash<br/>file input → ES output"]
        ELASTIC["Elasticsearch<br/>TLS :9200"]
        KIBANA["Kibana<br/>TLS :5601"]
        PLUGIN["TLSOC Parser plugin<br/>(optional Kibana plugin)"]
    end

    subgraph engine["tlsoc-engine (host service)"]
        SUP["Worker supervisor (main.py)"]
        W["Worker pool<br/>one Kafka consumer group"]
        RULES["YAML rule registry<br/>rules/*.yaml (hot reload)"]
        REDIS["Redis<br/>stateful transaction store"]
        GEO["GeoIP / ASN enrichment<br/>(offline MaxMind DBs)"]
        OUT["Buffered NDJSON output<br/>+ per-source DLQ"]
        WEBUI["Web UI console :8600"]
        SUP --> W
        W --- RULES
        W --- REDIS
        W --- GEO
        W --> OUT
    end

    subgraph reporting["tlsoc-reporting (host service / cron)"]
        FRAME["Report framework<br/>YAML reports × YAML datasources"]
        PDF["PDF chain<br/>wkhtmltopdf → Chrome → WeasyPrint"]
        EMAILER["Report emailer<br/>(cron, SMTP)"]
        FRAME --> PDF --> EMAILER
    end

    OMKAFKA -- "TLS :9094" --> KAFKA
    KAFKA -- "consume" --> W
    OUT -- "bind-mounted files" --> LOGSTASH
    LOGSTASH --> ELASTIC
    ELASTIC --> KIBANA
    KIBANA --- PLUGIN
    PLUGIN -. "rule / config edits" .-> RULES
    WEBUI -. "rule / config edits" .-> RULES
    ELASTIC -- "aggregations" --> FRAME
```

## Data flow

The life of one log line, end to end:

```mermaid
sequenceDiagram
    participant S as Source server (rsyslog)
    participant K as Kafka
    participant E as tlsoc-engine
    participant L as Logstash
    participant ES as Elasticsearch
    participant R as tlsoc-reporting

    S->>K: JSON envelope {meta, raw} over TLS (topic keyed by program)
    K->>E: consumer group poll (at-least-once)
    E->>E: route by meta.source_program → YAML rule
    E->>E: parse → ECS fields, real event @timestamp, GeoIP/ASN enrich
    E->>E: buffer batch, write NDJSON, then commit Kafka offsets
    E-->>E: unparseable line → per-source DLQ file
    L->>ES: tail output file, index over TLS
    Note over ES: dashboards, search, detections (Kibana)
    R->>ES: scheduled aggregation queries (daily window)
    R->>R: render HTML → PDF → email
```

**The Kafka envelope contract** — every producer (the onboarding script, rsyslog
template, Filebeat, Vector, or a custom script) sends messages of this shape:

```json
{
  "meta": {
    "org": "example_org",
    "dept": "it",
    "env": "production",
    "server": "web_server_1",
    "source_host": "web01",
    "source_program": "nginx_access"
  },
  "raw": "the original log line"
}
```

`meta.source_program` is the routing key that selects the parsing rule;
everything else under `meta` is carried through to the indexed event untouched.

## Deployment topology

The reference deployment uses one central TLSOC server plus any number of
monitored servers. Every arrow crossing a machine boundary is TLS-encrypted.

```mermaid
flowchart TB
    subgraph net["Your network"]
        subgraph m1["Monitored server A"]
            A1["rsyslog + omkafka"]
        end
        subgraph m2["Monitored server B"]
            B1["rsyslog + omkafka"]
        end
        subgraph central["TLSOC server (Ubuntu)"]
            subgraph dockerc["Docker Compose"]
                CK["Kafka"]
                CL["Logstash"]
                CE["Elasticsearch"]
                CKB["Kibana"]
            end
            CENG["tlsoc-engine (systemd)"]
            CRED["Redis"]
            CREP["tlsoc-reporting (cron)"]
        end
        AN["Analyst browser"]
    end

    A1 -- "TLS :9094" --> CK
    B1 -- "TLS :9094" --> CK
    CK --> CENG
    CENG --- CRED
    CENG -- "output files" --> CL
    CL --> CE
    CE --> CKB
    CE --> CREP
    AN -- "HTTPS :5601" --> CKB
```

Scaling out: the engine's workers form a single Kafka consumer group, so you scale
by adding CPU cores on one machine or by running the engine on several machines
with the same `group_id`. Kafka topic partition counts set the upper bound on
parallelism (one partition is consumed by exactly one worker).

## Communication and ports

| From | To | Protocol / port | Purpose |
|---|---|---|---|
| Monitored servers | Kafka | TLS `:9094` | Log envelope delivery (external listener) |
| tlsoc-engine | Kafka | `kafka:9092` (internal) or TLS `:9094` | Topic consumption |
| tlsoc-engine | Redis | `:6379` (localhost) | Stateful transaction correlation |
| Logstash | Elasticsearch | HTTPS `:9200` | Event indexing |
| Kibana | Elasticsearch | HTTPS `:9200` | Search and dashboards |
| Analysts | Kibana | HTTPS `:5601` | Web access |
| tlsoc-reporting | Elasticsearch | HTTPS `:9200` | Aggregation queries |
| Operators | Engine Web UI | HTTP `:8600` (localhost by default) | Rule/config console |

## Kafka flow

```mermaid
flowchart LR
    subgraph producers["Producers"]
        P1["Server A — rsyslog<br/>key = %programname%"]
        P2["Server B — rsyslog<br/>key = %programname%"]
    end
    subgraph kafka["Kafka topic (e.g. cse_logs, N partitions)"]
        T1["partition 0"]
        T2["partition 1"]
        T3["partition …"]
    end
    subgraph group["Consumer group: soc-parser-group"]
        C1["engine worker 1"]
        C2["engine worker 2"]
        C3["engine worker …"]
    end
    P1 --> T1
    P1 --> T2
    P2 --> T3
    T1 --> C1
    T2 --> C2
    T3 --> C3
```

Key properties:

- Messages are **keyed by source program**, so all lines from one source land in one
  partition — preserving order, which stateful (multi-line transaction) parsing
  requires.
- Offsets are committed **only after events are durably written to disk** by the
  engine (at-least-once delivery: duplicates are possible after a crash, silent
  loss is not).
- Partition count caps parallelism: provision at least as many partitions per topic
  as total engine workers.

## Parser pipeline

Inside tlsoc-engine, every message passes through this pipeline
(full detail: `tlsoc-engine/docs/architecture.md`):

```mermaid
flowchart TB
    IN["Kafka message<br/>{meta, raw}"] --> ROUTE["Route: meta.source_program<br/>→ program_mapping → rule (or rule chain)"]
    ROUTE --> STRAT{"Rule strategy"}
    STRAT -->|stateless| RX["Single regex"]
    STRAT -->|multi_match| MM["Pattern list<br/>(prematch gate → regex)"]
    STRAT -->|stateful| SF["Redis transaction<br/>correlate by ID, TTL"]
    STRAT -->|json_map| JM["Dot-path JSON mapping"]
    STRAT -->|xml_xpath| XM["XML element mapping"]
    RX --> NORM
    MM --> NORM
    SF --> NORM
    JM --> NORM
    XM --> NORM
    NORM["ECS field mapping<br/>+ real event @timestamp (UTC)<br/>+ GeoIP / ASN enrichment"] --> OUT["Batched NDJSON output<br/>offset commit after flush"]
    ROUTE -->|no match| DLQ["Per-source DLQ file<br/>+ storm warning"]
    NORM -->|parse failure| DLQ
```

## Reporting pipeline

Inside tlsoc-reporting (full detail: `tlsoc-reporting/docs/architecture.md`):

```mermaid
flowchart LR
    DS["Datasource YAML<br/>index + logical field map"] --> GEN
    RPT["Report YAML<br/>queries + narrative + layout"] --> GEN
    SIG["Signature packs<br/>attack patterns"] --> GEN
    GEN["Generator<br/>report × datasource"] --> Q["Query engine<br/>ES DSL or local NDJSON"]
    Q --> ES[("Elasticsearch")]
    Q --> W["Widgets<br/>KPIs, charts, tables, insights"]
    W --> HTML["HTML report"]
    HTML --> PDFE["PDF engine chain<br/>wkhtmltopdf → Chrome → WeasyPrint"]
    PDFE --> OUTP["output/pdf/…"]
    OUTP --> EM["Report emailer (cron)"]
```

One report definition multiplies across every datasource with a matching profile —
adding a log source or a report type is a YAML file, never a code change.
