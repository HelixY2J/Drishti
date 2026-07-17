# Why TLSOC?

The design rationale behind the platform: what problem it solves, why it is built
the way it is, what it deliberately does not do, and how it compares to the
alternatives you are probably considering.

## Contents

- [What problem are we solving?](#what-problem-are-we-solving)
- [Design philosophy](#design-philosophy)
- [Why agentless?](#why-agentless)
- [Why Kafka?](#why-kafka)
- [Why ECS?](#why-ecs)
- [Why not plain ELK?](#why-not-plain-elk)
- [Why not Elastic Security?](#why-not-elastic-security)
- [Trade-offs](#trade-offs)
- [Where this is going](#where-this-is-going)

## What problem are we solving?

Organizations such as universities, research institutions, and self-hosted
environments need centralized security monitoring across a fleet of heterogeneous
Linux servers — but commonly face constraints that mainstream offerings assume
away:

- **No agents allowed or wanted.** Server owners across departments will not
  install a third-party agent; machines are diverse, long-lived, and often not
  centrally managed.
- **Data must stay on premises.** Logs contain personal and institutional data;
  shipping them to a SaaS is not an option.
- **Small teams.** There is no dedicated engineering team to assemble, harden,
  and babysit a log pipeline from parts.
- **Zero licensing budget.** The platform must be fully open source.

TLSOC packages a complete pipeline for exactly this situation: agentless
collection with stock `rsyslog`, TLS-secured transport, ECS-normalized parsing,
Elastic-based storage and dashboards, and automated executive reporting — each
piece deployable and upgradable on its own.

## Design philosophy

TLSOC deliberately builds **on** proven open-source infrastructure instead of
reinventing mature components. Development effort is focused on the gaps that
generic log analytics platforms do not solve for this use case.

Instead of reimplementing Kafka, Elasticsearch, Kibana, or Logstash, TLSOC
integrates them and concentrates its own engineering where it adds the most
value:

- Agentless onboarding and one-command deployment automation
- YAML-driven parsing and ECS normalization, with validation and golden-sample
  testing
- A modular parser framework (five strategies, hot reload, dead-letter queues)
- A declarative executive reporting framework
- The repository ecosystem and operational workflows
- Future analytics and anomaly detection

The advantages of this approach:

- Reuse proven, production-tested infrastructure.
- Stay compatible with the broader Elastic ecosystem (ECS, index templates,
  Kibana tooling).
- Reduce maintenance burden — TLSOC maintains its logic, not a database engine.
- Let organizations upgrade underlying components independently.
- Focus engineering effort where it delivers the greatest value.

TLSOC builds upon mature infrastructure while addressing deployment, parser
management, and operational workflows that are outside the scope of the
underlying projects.

## Why agentless?

Agents give richer telemetry, but they carry costs that are disqualifying in
TLSOC's target environments: something to install, update, and secure on every
machine; resistance from server owners; and incompatibility with unmanaged or
legacy systems.

TLSOC's collection layer is **stock `rsyslog` with the `omkafka` module** — both
from the distribution's own package repository. The onboarding script only writes
one rsyslog configuration file, validated before activation, with rotation-safe
tailing and buffered retry. If you ever remove TLSOC, you delete one config file;
nothing else was ever on the machine.

The trade-off is explicit: no endpoint telemetry (process trees, memory,
EDR-style visibility) — see [Trade-offs](#trade-offs).

## Why Kafka?

A monitoring pipeline's transport must absorb failure without losing data:

- **Buffering** — if the parsing engine or Elasticsearch is down, logs accumulate
  in Kafka instead of being dropped; rsyslog itself buffers and retries if Kafka
  is unreachable.
- **Backpressure and replay** — consumers process at their own pace and can
  re-consume after a crash; the engine commits offsets only after events are
  durably written (at-least-once delivery).
- **Ordering where it matters** — messages are keyed by source program, so all
  lines from one source stay in one partition, which is exactly what multi-line
  transaction parsing (e.g. Postfix) requires.
- **Scale-out for free** — partitions spread across the engine's worker pool and
  across machines in the same consumer group, with no code changes.

## Why ECS?

[Elastic Common Schema](https://www.elastic.co/guide/en/ecs/current/index.html)
gives every event the same field vocabulary: `source.ip` is `source.ip` whether
the event came from NGINX, Postfix, or a firewall. That single decision is what
makes one set of dashboards, one signature pack, and one reporting framework work
across all sources — and keeps TLSOC data compatible with the wider Elastic
ecosystem's tooling. The engine enforces it: every rule field is validated
against ECS, and rules with invalid fields cannot ship.

## Why not plain ELK?

The ELK stack is the foundation TLSOC stands on — but a stock ELK install is a
toolkit, not a monitoring platform. Standing one up for the TLSOC use case means
building, by hand: TLS everywhere with a managed CA, a transport/buffering layer,
per-source parsing and normalization, source onboarding that survives log
rotation, index templates, dashboards, and scheduled reporting — then maintaining
all of it.

TLSOC is that missing layer, prebuilt and opinionated:

- One-command TLS-secured deployment ([TLSOCDockerDeploy](https://github.com/sankettaware16/TLSOCDockerDeploy))
- Verified agentless onboarding per server, in one command
- A parser framework with validation, testing, and hot reload
  ([foss-soc-engine](https://github.com/sankettaware16/foss-soc-engine))
- Executive reporting out of the box ([tlsoc-reporting-framework](https://github.com/sankettaware16/tlsoc-reporting-framework))

If you already run a tuned ELK pipeline you are happy with, TLSOC's components
still compose: the engine works with any Kafka + any consumer of NDJSON, and the
reporting framework works against any Elasticsearch 8.x with ECS-style data.

## Why not Elastic Security?

Elastic Security is an excellent platform and provides comprehensive SIEM
capabilities, particularly when deployed with Elastic Agent. TLSOC was designed
for a different deployment model and a different set of operational constraints.

| | Elastic Security | TLSOC |
|---|---|---|
| Collection model | Designed around Elastic Agent | Native Linux logging (`rsyslog` + `omkafka`), agentless |
| Endpoint coverage | Endpoint agents required for full capability | No endpoint telemetry by design |
| Scope | General-purpose security platform | Opinionated deployment for infrastructure log monitoring |
| Setup | Flexible, but requires significant configuration | Ready-to-deploy architecture with integrated components |
| Parsing / detection content | Managed within Elastic (integrations, detection rules) | YAML-driven parser and normalization framework, YAML signature packs |
| Target environment | Built primarily for enterprise environments | Optimized for universities, research labs, and self-hosted environments |

TLSOC does **not** attempt to replace Elastic Security — Elastic Security offers
capabilities TLSOC currently does not, such as endpoint telemetry, mature
detection content, and integrated case management. TLSOC provides an alternative
architecture for organizations that prefer agentless log collection,
infrastructure-native deployment, and modular open-source components, with custom
logic only where it adds the most value.

## Trade-offs

Honest limits of the current platform — deliberate scope decisions, not
oversights:

- **No endpoint telemetry.** Agentless means log files only: no process trees,
  memory analysis, or EDR-style response. If you need endpoint depth, pair TLSOC
  with (or choose) an agent-based tool.
- **Detection is signature- and report-driven today.** Threat patterns live in
  the reporting framework's signature packs and Kibana; there is no streaming
  correlation/alerting engine yet (planned — see the roadmap).
- **No case management or SOAR.** TLSOC produces normalized data, dashboards,
  and reports; ticketing and incident workflow live in your existing tools
  (an `tlsoc-integrations` repository is planned).
- **Single-node reference deployment.** The stack targets one server; multi-node
  Elasticsearch/Kafka profiles are on the roadmap. The engine itself already
  scales horizontally via Kafka consumer groups.
- **Linux-first sources.** Bundled parsers target Linux infrastructure services;
  other sources arrive as YAML rules (any syslog/JSON/XML producer works today).

## Where this is going

The [roadmap](roadmap.md) covers what is shipped, in progress, and planned —
including the dashboard, ML-based anomaly detection, community parser packs, and
integrations. Proposals and questions are welcome as
[issues](https://github.com/sankettaware16/tlsoc/issues).
