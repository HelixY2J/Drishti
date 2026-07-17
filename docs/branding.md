# TLSOC Branding

One visual identity across every repository. Everything a README or document needs
is defined here — copy, do not improvise.

## Name

- The platform is **TLSOC** — always uppercase in prose ("TLSOC", never "Tlsoc" or
  "tlsoc" outside repository names and code).
- Repository names are lowercase (`tlsoc-engine`) per the
  [naming convention](ecosystem.md#naming-convention).

## Logo

- File: `docs/images/tlsoc-logo.jpeg` (the blue "TL" mark), present in every
  repository at that exact path.
- README usage — centered, 110 px wide, above the project title:

```html
<p align="center">
  <img src="docs/images/tlsoc-logo.jpeg" alt="TLSOC logo" width="110"/>
</p>

<h1 align="center">TLSOC Engine</h1>
```

## Color palette

Drawn from the logo:

| Role | Color | Hex |
|---|---|---|
| Primary (brand blue) | Royal blue | `#4a6edb` |
| Secondary | Slate blue | `#7b96d4` |
| Dark accent (text/diagrams) | Navy | `#1f2d50` |
| Elastic-related badges | Elastic teal | `#005571` |
| Neutral | Light grey | `#d9d9d9` |

Use the primary blue for the ecosystem badge and brand accents; use standard
shields.io colors elsewhere (badges must stay recognizable).

## Badges

Badge row order in every README: **ecosystem badge, license, then repo-specific
badges** (CI, language/runtime, platform).

```markdown
[![TLSOC Ecosystem](https://img.shields.io/badge/TLSOC-Ecosystem-4a6edb)](https://github.com/sankettaware16/tlsoc)
[![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-blue)](LICENSE)
```

Repo-specific examples:

```markdown
![CI](https://github.com/sankettaware16/tlsoc-engine/actions/workflows/ci.yml/badge.svg)
![Python](https://img.shields.io/badge/python-3.8%2B-3776AB?logo=python&logoColor=white)
![Docker Compose](https://img.shields.io/badge/docker-compose-2496ED?logo=docker&logoColor=white)
![Elastic Stack](https://img.shields.io/badge/elastic%20stack-8.x-005571?logo=elastic)
```

## Markdown style

- ATX headings (`##`), sentence-case section titles exactly as listed in the
  [standard README layout](ecosystem.md#standard-readme-layout).
- One blank line between blocks; no trailing whitespace; wrap prose around
  90 characters.
- Tables for enumerable facts (ports, files, repositories); prose for explanation.
- Fenced code blocks always carry a language tag (`bash`, `yaml`, `json`,
  `mermaid`).
- Links to sibling ecosystem repositories always use the full GitHub URL; links
  within a repository are relative.
- Emoji are not used in headings or prose, with two exceptions: status markers
  (`✅` shipped, `🚧` in progress) in roadmaps and status lists, and the heart in
  the standard footer.

## Standard footer

Every README ends with this footer (after the License section, separated by a
horizontal rule):

```html
---

<p align="center">
  Built with ❤️ by <b>TrustLab, IIT Bombay</b><br/>
  Part of the <a href="https://github.com/sankettaware16/tlsoc">TLSOC Ecosystem</a>
</p>

<p align="center">
  <a href="https://github.com/sankettaware16/tlsoc">TLSOC</a> •
  <a href="https://github.com/sankettaware16/tlsoc-engine">Engine</a> •
  <a href="https://github.com/sankettaware16/tlsoc-docker-deploy">Deploy</a> •
  <a href="https://github.com/sankettaware16/tlsoc-reporting">Reporting</a>
</p>
```

Add new repositories to the footer link row as they are published.

## Positioning language

- TLSOC is described as an **"open-source Security Operations Platform delivering
  SIEM capabilities"** — not as "a SOC" (a SOC is people and processes, not
  software) and not as only "a SIEM" (TLSOC also ships deployment automation,
  onboarding, parsing, and reporting).
- Spell out **Security Information and Event Management (SIEM)** on first use in
  a document.
- Frame the relationship to underlying technology as *extending, not replacing*:
  never claim to improve on Kafka/Elasticsearch/Kibana themselves, and never
  disparage alternatives — explain the different constraints TLSOC optimizes
  for. See [why-tlsoc.md](why-tlsoc.md) for the canonical wording.

## Diagrams

- Mermaid only (fenced code blocks with the `mermaid` language tag) — rendered
  natively by GitHub.
- Direction: `flowchart LR` for pipelines, `flowchart TB` for component stacks,
  `sequenceDiagram` for message flows.
- Node labels name the repository (`tlsoc-engine`) or the concrete technology
  (`Apache Kafka`), not abbreviations.

## Voice and tone

- Concise, technical, factual. No marketing superlatives, no informal asides.
- Claims about performance or guarantees state their conditions ("measured on…",
  "at-least-once: duplicates possible, silent loss not").
- Consistent terminology per the [documentation conventions](ecosystem.md#documentation-conventions).
