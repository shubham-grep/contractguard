# ContractGuard

> **Project status:** design and planning. ContractGuard is not yet installable, and no application or CLI commands are available.

ContractGuard is a planned open-source developer tool for comparing a baseline and candidate OpenAPI 3.x document and reporting breaking and non-breaking API changes. Its deterministic comparison engine will power several interfaces without relying on AI at runtime.

The intended workflow is simple: supply an earlier specification as the baseline and a proposed specification as the candidate, validate and normalize both documents, then receive a stable list of classified changes with machine-readable rule identifiers and useful API locations. The initial focus is trustworthy change detection rather than API linting, documentation generation, traffic analysis, or AI-authored compatibility judgments.

## Planned interfaces

| Interface | Purpose | Delivery target |
| --- | --- | --- |
| TypeScript core | Reusable parsing, normalization, comparison, and reporting | MVP |
| CLI | Local and CI-friendly comparisons with machine-readable output | MVP |
| Web application | Registration-free comparison of files in the browser | MVP |
| GitHub Action | Pull-request checks and change summaries | Post-MVP |
| MCP server | Structured access for compatible developer tools | Post-MVP |

## Design commitments

- **Deterministic:** identical inputs and configuration produce identical, stably ordered results. AI may assist development, review, testing, and documentation, but never decides results or test expectations.
- **Private by default:** the web application processes specifications locally in a Web Worker. It will not upload document contents and needs no account, database, or application backend.
- **Zero additional cost:** development, public-repository CI, and static hosting must use open-source tools and no-cost services or allowances. No paid AI API is required in production.
- **Library first:** the core remains independent of the DOM, filesystem, network, process globals, and interface-specific presentation.
- **Maintainable:** the architecture favors a small pnpm workspace and workflows manageable by one maintainer.

## Repository documentation

- [Product requirements](docs/product/PRD.md)
- [Solution design](docs/architecture/SOLUTION_DESIGN.md)
- [Test strategy](docs/testing/TEST_STRATEGY.md)
- [Architecture decision 0001](docs/adr/0001-project-architecture.md)
- [Roadmap](ROADMAP.md)
- [Contributor and agent guidance](AGENTS.md)

## Proposed repository structure

The planned pnpm workspace separates deterministic product logic from delivery adapters:

```text
apps/web/                 Static browser application and Web Worker
packages/core/            Pure OpenAPI parsing, normalization, and diff rules
packages/cli/             Command-line adapter
packages/github-action/   GitHub workflow adapter
packages/mcp-server/      MCP protocol adapter
test/fixtures/            Shared baseline, candidate, and expected-result cases
docs/                     Requirements, architecture, testing, and ADRs
```

`packages/core` must not depend on browser, filesystem, network, process, or interface-specific presentation APIs. Each surface calls the same public engine so a change is classified consistently everywhere. See the solution design for detailed package responsibilities and data flow.

## Contributing at this stage

Contributions should currently focus on reviewing requirements, architecture, rule semantics, and unresolved decisions. Do not infer working setup commands from the design documents; root `pnpm` checks are a planned command contract and will be documented as available only after the toolchain exists. Follow `AGENTS.md`, keep changes narrowly scoped, and record decisions that alter public behavior in an ADR.

There is currently nothing to install, build, or run. Once implementation begins, setup and verification commands will be added here together with supported runtime versions. Changes that introduce behavior will need focused tests and corresponding documentation; changes to architecture or durable policy will need an ADR.

Never commit confidential API specifications, credentials, or customer data. Test documents must be synthetic or explicitly redistributable.

## Assumptions and unresolved decisions

Current assumptions are directional comparison from baseline to candidate, support for OpenAPI 3.0 and 3.1 YAML/JSON, static hosting on GitHub Pages, no remote reference retrieval, and no server-side document processing. Still to decide are the exact diff policy and rule identifiers, local/bundled reference handling, input-size targets, UI framework, supported Node.js/browser versions, license, and release/versioning policy. The PRD and ADR are authoritative as these decisions are resolved.
