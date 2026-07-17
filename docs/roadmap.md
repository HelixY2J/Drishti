# TLSOC Roadmap

The ecosystem-wide roadmap. Component-specific detail lives in each repository's
`docs/roadmap.md`; this page tracks the platform as a whole.

Items move top-to-bottom as they mature: **Long Term → Future → Next Release →
Current**. Dates are intentionally absent — scope is committed, timing is not.

At a glance:

| Status | Items |
|---|---|
| ✅ Shipped | tlsoc-engine · tlsoc-docker-deploy · tlsoc-reporting · agentless onboarding |
| 🚧 In progress | Ecosystem documentation & branding · versioned releases · parser library growth |
| Planned | tlsoc-dashboard · tlsoc-ml · tlsoc-parser-plugins · tlsoc-examples · tlsoc-integrations |

## Current

Shipped and actively maintained:

- **tlsoc-engine** — polymorphic parsing engine: five rule strategies (stateless,
  multi_match, stateful, json_map, xml_xpath), ECS-validated output, real event
  timestamps, offline GeoIP/ASN enrichment, per-source dead-letter queues,
  at-least-once Kafka delivery, worker-pool scaling, live rule reload, preflight
  and pipeline-replication validators, golden-sample regression CI, Web UI
  console, and a native Kibana plugin.
- **tlsoc-docker-deploy** — one-command TLS-secured Kafka + Logstash +
  Elasticsearch + Kibana stack for fresh Ubuntu servers, local CA generation, and
  the agentless `tlsoc-onboard.sh` source-onboarding script with end-to-end
  delivery verification.
- **tlsoc-reporting** — declarative report framework (YAML reports × YAML
  datasources), signature-based security analysis, offline preview backend,
  HTML/PDF rendering chain, daily cron automation, and email delivery.

## Next Release

- Unified ecosystem documentation and branding across all repositories (this
  refactoring).
- Consistent repository metadata, community health files, and issue/PR templates.
- Tagged, changelog-backed releases for every repository.
- Expanded parser rule library and golden-sample coverage in tlsoc-engine.
- Screenshot and demo assets for the ecosystem home.

## Future

- **tlsoc-dashboard** — a unified operational dashboard: pipeline health (Kafka
  lag, engine EPS, DLQ rates), source inventory, and report browsing in one place.
- **tlsoc-parser-plugins** — a community rule-pack repository with CI-enforced
  golden samples, so new log sources can be shared without touching the engine.
- **tlsoc-examples** — reproducible end-to-end labs: single-VM demo, multi-server
  deployment, and sample dashboards.
- **tlsoc-integrations** — alert and workflow connectors (email/webhook alerting,
  ticketing, threat-intelligence enrichment).
- Engine: optional direct-to-Elasticsearch output alongside file output.
- Deploy: multi-node Elasticsearch and Kafka profiles.

## Long Term Vision

- **tlsoc-ml** — machine-learning anomaly detection over normalized ECS events
  (behavioral baselines per source, alert scoring), feeding detections back into
  dashboards and reports.
- Detection-as-code: a shared, versioned detection rule format spanning the
  engine, dashboards, and reports.
- Turn-key appliance images for the full platform.
- A documentation website aggregating all repository docs.

## How the roadmap is maintained

- Proposals start as GitHub issues in the relevant repository (or in `tlsoc` for
  cross-cutting ideas) using the feature-request template.
- Accepted items are added here under **Future** or **Next Release** via pull
  request, and mirrored in the owning repository's `docs/roadmap.md`.
