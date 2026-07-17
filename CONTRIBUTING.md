# Contributing to TLSOC

Thank you for your interest in improving TLSOC. This guide covers contributions to
the ecosystem as a whole; each repository additionally has its own `CONTRIBUTING.md`
with component-specific instructions (for example, how to write and test parser
rules in TLSOC Engine).

## Where to contribute

| I want to… | Go to |
|---|---|
| Fix or improve platform documentation, architecture, roadmap | this repository ([tlsoc](https://github.com/sankettaware16/tlsoc)) |
| Add or fix a parsing rule, engine feature, Web UI, Kibana plugin | [foss-soc-engine](https://github.com/sankettaware16/foss-soc-engine) |
| Improve the Docker stack, TLS setup, onboarding script | [TLSOCDockerDeploy](https://github.com/sankettaware16/TLSOCDockerDeploy) |
| Add a report type, datasource, signature pack, or reporting feature | [tlsoc-reporting-framework](https://github.com/sankettaware16/tlsoc-reporting-framework) |
| Propose something cross-cutting or a new ecosystem repository | this repository, via a feature-request issue |

## Ground rules

- Be respectful — all participation is governed by the
  [Code of Conduct](CODE_OF_CONDUCT.md).
- **Never include real deployment data** in issues, commits, or documentation:
  no real IP addresses, hostnames, organization names, credentials, or log
  samples. Use placeholders (`example_org`, `203.0.113.10`).
- Security vulnerabilities go through [SECURITY.md](SECURITY.md), never public
  issues.
- Contributions are accepted under the repository's
  [Apache-2.0 license](LICENSE).

## Workflow

1. **Open or find an issue first** for anything larger than a typo fix, so the
   approach can be agreed before you invest time. Use the issue templates.
2. **Fork and branch** — branch from `main`, one topic per branch
   (`docs/fix-architecture-diagram`, `feat/new-onboarding-check`).
3. **Make the change**, following the conventions in
   [docs/ecosystem.md](docs/ecosystem.md) and [docs/branding.md](docs/branding.md)
   for anything user-facing. Keep diffs focused; avoid drive-by refactoring.
4. **Validate** — run the repository's test or validation commands (each
   repository's CONTRIBUTING lists them; for documentation, check that links
   resolve and Mermaid blocks render in the GitHub preview).
5. **Open a pull request** using the PR template. Describe what changed and why;
   link the issue. Update `CHANGELOG.md` under `[Unreleased]` when the change is
   user-visible.

## Documentation contributions

Documentation is a first-class contribution in every TLSOC repository:

- Follow the [standard README layout](docs/ecosystem.md#standard-readme-layout)
  and [branding guide](docs/branding.md).
- Long-form content belongs in `docs/`, not the README.
- Keep terminology consistent with
  [the glossary](docs/ecosystem.md#documentation-conventions).

## Commit style

- Small, coherent commits with imperative subject lines
  ("Add deployment diagram", not "Added diagrams").
- Reference issues in the body (`Fixes #12`) where applicable.

## Questions

Open a [discussion or issue](https://github.com/sankettaware16/tlsoc/issues) in
this repository — if it belongs elsewhere, it will be routed to the right place.
