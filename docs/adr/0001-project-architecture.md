# ADR 0001: TypeScript Monorepo with a Deterministic Core

- **Status:** Proposed
- **Date:** 2026-07-18
- **Decision owners:** Project maintainer
- **Related:** [PRD](../product/PRD.md), [solution design](../architecture/SOLUTION_DESIGN.md), [test strategy](../testing/TEST_STRATEGY.md)

## Context

ContractGuard must compare OpenAPI 3.x documents consistently across a browser application, reusable library, CLI, GitHub Action, and MCP server. The public application must work without registration, keep specifications in the browser, and require neither a backend nor a database. Development, CI, distribution, and hosting must add no monetary cost. The system must remain understandable to one maintainer, and compatibility decisions must never depend on AI or another nondeterministic service.

The repository is pre-implementation. This ADR establishes the direction needed to evaluate tooling and begin the core engine; exact libraries and supported runtime versions remain follow-up decisions.

## Decision Drivers

- One authoritative implementation of compatibility semantics
- Reproducible and explainable output across all interfaces
- Browser and Node.js compatibility without separate engines
- No transmission or persistence of browser-provided specifications
- Low operational and maintenance burden
- No required paid service or runtime AI dependency
- Independently testable domain and adapter boundaries

## Decision

Adopt a TypeScript ESM monorepo using a proposed pnpm workspace, with dependencies directed inward to a pure `packages/core` package.

1. **Core authority.** `packages/core` will own string/object input validation, normalization, internal reference handling, comparison rules, diagnostics, stable finding types, ordering, and canonical serialization. Its public comparison path will perform no filesystem, network, DOM, process, clock, locale-sensitive, random, telemetry, or AI operations. Stable rule IDs and a versioned report schema will make decisions explainable and automation-safe.
2. **Thin adapters.** `packages/cli`, `packages/github-action`, and `packages/mcp-server` will translate their transports to and from the core API. `apps/web` will invoke the same core inside a Web Worker. Adapters may handle files, process exits, protocol messages, or presentation, but must not classify changes.
3. **Static, local-first web application.** The web build will contain only static assets. The browser will read selected/pasted documents into volatile memory, send them to a same-origin worker, render escaped results, and export only on an explicit user action. There will be no application backend, account system, database, analytics, or runtime third-party script.
4. **Offline reference policy.** Remote `$ref` retrieval will be disabled. Internal references are required for the MVP; explicitly supplied multi-document bundles remain a separate decision. Unsupported references will produce diagnostics rather than partial, silent comparison.
5. **Shared verification assets.** Synthetic or redistributable fixtures under `test/fixtures` will drive core rule tests and adapter parity tests. Architecture-specific code may depend on core contracts; core tests and source may not depend on adapter packages.
6. **No-additional-cost delivery.** The proposed baseline is a public GitHub repository using standard GitHub-hosted Actions runners, GitHub Pages for the static application, and the public npm registry for packages. These choices currently provide a no-charge path for public projects ([GitHub Actions billing](https://docs.github.com/en/actions/concepts/billing-and-usage), [GitHub Pages availability](https://docs.github.com/en/pages/getting-started-with-github-pages), [npm public package pricing](https://www.npmjs.com/products)). Larger paid runners, paid domains, private package features, and services that can incur overage are outside the baseline; CI must use zero-spend limits and short artifact retention.

The intended dependency direction is:

```text
apps/web ───────────────┐
packages/cli ───────────┤
packages/github-action ─┼──> packages/core
packages/mcp-server ────┘
```

No adapter may import another adapter merely to reach core behavior. A small transport-neutral shared package may be introduced later only if duplication is demonstrated and it does not weaken the core boundary.

## Consequences

### Positive

- Every surface receives the same classifications, diagnostics, and ordering.
- Most correctness tests run against pure functions without browsers, processes, or network setup.
- A static site has a small attack surface and no application data store to operate or breach.
- TypeScript types and fixtures can be reused across browser and Node.js builds.
- The project can be built, tested, hosted, and distributed through public-project services without an expected bill.
- Adapter delivery can be phased without coupling core correctness to every interface.

### Negative and Trade-offs

- Browser compatibility constrains core dependencies, parsing strategies, bundle size, and available APIs.
- A Web Worker adds message serialization, cancellation, and duplicate-memory concerns.
- pnpm workspace and multi-package releases add configuration before all packages exist.
- Disabling remote references rejects some real-world, unbundled specifications.
- GitHub-hosted services and npm are external dependencies whose terms and quotas can change.
- TypeScript cannot by itself guarantee runtime safety for hostile YAML/JSON; explicit limits and hardened parsing remain necessary.

## Alternatives Considered

### Server-side comparison service

Rejected for the initial product. It would simplify browser execution and centralized updates, but would upload sensitive contracts and introduce hosting, abuse prevention, privacy operations, persistence questions, and potential cost.

### Separate repositories or independently implemented tools

Rejected. Repository and release isolation would increase coordination work and make semantic drift between the web, CLI, Action, and MCP interfaces more likely.

### CLI-first implementation with comparison logic in the CLI

Rejected. Extracting a library later risks process and filesystem concerns leaking into the domain API. Core-first development provides a clean contract from the beginning.

### AI-assisted compatibility classification

Rejected for product behavior. Model output is not sufficiently deterministic or reproducible, may expose specifications, and can require paid infrastructure. AI may assist development, review, testing, and documentation without entering runtime decisions or test oracles.

### Server-side JavaScript plus a separate browser/WASM engine

Rejected initially. A polyglot or dual-engine design increases build complexity and parity testing. WASM can be reconsidered only if measured performance or parser safety cannot meet requirements in TypeScript.

### Automatic remote reference resolution

Rejected by default. Network resolution undermines offline determinism and privacy and introduces SSRF, authentication, availability, and content-drift risks. Users may bundle documents before comparison; safe explicit multi-file input can be designed later.

## Acceptance Conditions

Change this ADR to **Accepted** only after a discovery spike confirms that the chosen parser and validator work in maintained browsers and Node.js, a Web Worker can meet provisional performance limits, the proposed package/build setup stays maintainable, and the service choices still satisfy the cost policy. The solution design and threat model must also define resource limits and reference behavior precisely enough to begin implementation.

## Assumptions

- OpenAPI 3.0.x and 3.1.x YAML/JSON are the initial formats.
- Comparison is directional from baseline to candidate.
- The repository and published packages will be public.
- A worker-based static application is feasible for representative specifications.
- GitHub Pages, standard public-repository Actions usage, and public npm packages remain available without additional charge; this must be rechecked at each release.

## Unresolved Decisions

- Parser, validator, YAML, canonical serialization, and property-testing libraries
- Exact workspace/build/release tooling and whether pnpm is accepted as proposed
- Node.js and browser support matrices
- MVP rule catalog, severity taxonomy, report schema, and compatibility policy
- Multi-document local reference input, recursion semantics, and resource limits
- Web UI framework, worker cancellation protocol, fallback behavior, and whether GitHub Pages provides sufficient security controls
- Package scope, license, governance, release provenance, and version coordination
- GitHub Action baseline acquisition and whether MCP should ever add restricted path input beyond its content-only default
- A free-host migration plan if provider terms or quotas no longer meet the cost requirement
