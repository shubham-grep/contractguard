# ContractGuard Roadmap

**Status:** Initial draft
**Last updated:** 2026-07-18
**Planning model:** Sequential, exit-criteria driven; no committed dates

## Roadmap Principles

ContractGuard will be built in small, releasable increments around one deterministic TypeScript comparison engine. Browser processing stays local, adapters stay thin, and no phase may introduce a database, paid AI dependency, or required paid infrastructure. Each phase begins only when its dependencies and relevant design decisions are sufficiently resolved. Scope should be reduced before operational complexity is added.

The proposed MVP milestone follows Phase 4: the core library, CLI, and registration-free static web app. The GitHub Action and MCP server extend the same public contracts. The production release phase hardens and publishes the complete initial toolset.

## Phase 1: Discovery and Design

### Deliverables

- Approved initial PRD, solution design, test strategy, architecture decision record, roadmap, contributor guide, and repository README.
- A prioritized OpenAPI compatibility rule matrix with examples, directionality, stable proposed IDs, and MVP/deferred labels.
- Representative public or synthetic OpenAPI 3.0 and 3.1 fixture plan, including malformed, adversarial, recursive, and cross-version cases.
- Decisions or time-boxed technical evaluations for parsing/validation, `$ref` handling, canonical output, monorepo tooling, browser execution, hosting, and package/release strategy.
- Threat model and privacy data-flow review confirming browser-local processing and no required backend.
- Definition of public contracts: finding model, diagnostics, engine options, CLI exit behavior, and compatibility policy draft.
- A zero-additional-cost checklist covering development, CI, releases, hosting, and optional services.

### Exit Criteria

- Maintainer accepts the MVP boundaries, non-goals, terminology, and baseline-to-candidate compatibility perspective.
- Every MVP rule has an expected classification and at least one planned positive and negative fixture.
- High-impact open decisions have an owner, decision deadline, and documented fallback; none blocks core implementation.
- Architecture and privacy review show no database, server-side specification processing, paid service, or AI runtime requirement.
- Work for the next phase is decomposed into small, independently testable issues.

### Dependencies

- Product constraints and user needs in the PRD.
- OpenAPI 3.0 and 3.1 specifications and JSON Schema behavior.
- Maintainer agreement on licensing, package naming, and supported runtimes.

### Risks

- Compatibility semantics are larger and more nuanced than the MVP can safely cover.
- Premature tool selection could add maintenance burden or constrain browser use.
- Ambiguous `$ref`, schema-dialect, and request/response variance rules could cause false confidence.
- Free service limits or terms could conflict with the cost requirement.

## Phase 2: Core Diff Engine

### Deliverables

- Minimal monorepo/workspace foundation and `packages/core` with explicit public API boundaries.
- Safe parsing and input diagnostics for supported OpenAPI 3.0.x/3.1.x YAML and JSON documents.
- Canonical internal representation sufficient for MVP rules, with documented handling of ignored fields and references.
- Deterministic traversal, rule registry, change model, stable ordering, canonical JSON serialization, and error model.
- Small vertical rule increments, starting with paths/operations and expanding through parameters, bodies, responses, media types, schemas, enums, required properties, and security rules.
- Versioned rule documentation and fixture corpus alongside unit, integration, property-based, mutation/regression, and performance tests.
- Package API documentation and initial release candidate of the reusable library.

### Exit Criteria

- Every approved MVP rule passes positive, negative, direction-inversion where relevant, and non-semantic-formatting tests.
- Identical semantic inputs produce byte-equivalent canonical results across repeated supported-runtime runs.
- Invalid, unsupported, cyclic, deeply nested, and resource-constrained inputs fail safely with actionable diagnostics.
- Coverage and quality gates in the test strategy pass; no release-blocking security or license issue remains.
- Performance and memory measurements meet the approved targets on representative small and large fixtures.
- Core has no browser-incompatible dependency and no interface-specific rendering, process exit, GitHub, or MCP logic.

### Dependencies

- Phase 1 rule matrix, public contract drafts, reference policy, fixture plan, and tooling decisions.
- Stable upstream OpenAPI/YAML/JSON Schema libraries that satisfy browser, license, security, and cost constraints.

### Risks

- JSON Schema variance and composition keywords can produce incorrect classifications.
- Parser or resolver behavior may differ between OpenAPI 3.0 and 3.1.
- Canonicalization might accidentally hide a semantic change or surface formatting noise.
- Large or adversarial documents may exhaust CPU, memory, or recursion limits.

## Phase 3: CLI

### Deliverables

- Thin `packages/cli` adapter over `packages/core` for explicit baseline and candidate file paths.
- Human-readable terminal output and versioned canonical JSON output.
- Documented flags, baseline/candidate order, severity filtering if approved, and stable exit codes for clean, breaking-change, and tool/input-error outcomes.
- Cross-platform packaging and smoke tests for supported Node.js versions on macOS, Linux, and Windows.
- CLI usage, automation examples, troubleshooting, and shell-safe handling guidance.
- An installable prerelease published only after package and provenance checks are ready.

### Exit Criteria

- End-to-end tests prove the CLI returns the same findings as direct core invocation for the fixture corpus.
- Output is deterministic, noninteractive by default, pipe-safe, and usable in CI.
- Errors do not expose unnecessary source content, overwrite files, or leave partial reports.
- Install, invocation, JSON-schema validation, help, version, and exit-code smoke tests pass on every supported platform/runtime.
- No comparison rule is implemented or forked in the CLI.

### Dependencies

- Stable Phase 2 core API, result schema, diagnostics, and initial compatibility policy.
- Decisions on package names, supported Node.js versions, executable packaging, and release registry.

### Risks

- Exit-code semantics or output changes could break early automation consumers.
- Cross-platform path, encoding, color, signal, and stream behavior may diverge.
- Packaging choices could inflate install size or duplicate dependencies.

## Phase 4: Web Application

### Deliverables

- Static `apps/web` application using `packages/core` directly in the browser.
- Clearly labeled baseline/candidate file selection and text-paste flows for YAML/JSON, plus clear/reset controls.
- Accessible summary, finding filters, detailed explanations, diagnostics, and explicit empty states.
- User-initiated canonical report export without server persistence.
- Dedicated Web Worker execution with cancellation and timeout behavior validated by performance testing.
- Host-compatible CSP/security controls, dependency isolation, safe rendering, input/resource limits, and a visible local-processing statement.
- Responsive design, automated and manual accessibility coverage, browser compatibility tests, and static deployment workflow.

### Exit Criteria

- Production-like network tests confirm that specification contents and findings never leave the browser.
- The full comparison workflow works without registration, accounts, cookies, a database, or an application backend.
- Browser results match core golden fixtures and satisfy agreed performance/resource targets.
- Keyboard-only and screen-reader smoke tests pass, automated accessibility checks have no serious/critical issue, and classifications do not rely on color.
- CSP and adversarial-input tests show no script/markup injection or unauthorized network destination.
- A reproducible static build deploys successfully on the selected no-additional-cost host.

### Dependencies

- Stable Phase 2 browser-compatible core and report schema; Phase 3 feedback on terminology and result presentation.
- Approved web framework/build tooling, hosting, browser support policy, design baseline, and privacy threat model.

### Risks

- Large files can freeze a tab or exceed browser memory.
- Unsafe rendering of specification text can create cross-site scripting risk.
- Framework or hosting defaults may introduce telemetry, third-party requests, or hidden server features.
- Accessible presentation of deeply nested findings may become visually complex.

## Phase 5: GitHub Action

### Deliverables

- Thin `packages/github-action` adapter that invokes the released core directly without reimplementing rules.
- Inputs for explicit baseline and candidate sources, documented permissions, output mode, and failure threshold.
- Check summary and downloadable machine-readable artifact; annotations are added only within documented platform limits.
- Secure handling for pull requests from forks and untrusted specifications, with no implicit execution of repository code.
- Pinned, reproducible Action bundle, action metadata, example workflow, release tags, and automated update process.
- Integration tests against representative pull request and repository layouts.

### Exit Criteria

- The Action reports the same findings and failure outcome as the core/CLI for shared fixtures.
- Default permissions are read-only and least privilege; fork and untrusted-input threat scenarios are tested.
- Baseline selection is explicit and documented, with actionable failures for missing, invalid, or inaccessible inputs.
- A sample public repository completes the workflow at no additional cost.
- Bundled dependencies, provenance, tags, and marketplace/repository documentation pass the release checklist.

### Dependencies

- Stable core result schema and CLI/engine behavior from Phases 2–3.
- GitHub permission, annotation, artifact, and public-repository CI constraints.
- Decision on baseline acquisition and the Action's report/annotation presentation policy.

### Risks

- Pull-request permission differences and fork security can make baseline access unsafe or unreliable.
- Git history or remote baseline retrieval may be ambiguous and slow.
- Annotation limits and generated Action bundles can complicate maintenance and review.
- Platform or free-tier policy changes may affect CI usage.

## Phase 6: MCP Server

### Deliverables

- Thin `packages/mcp-server` exposing explicit comparison and rule-information tools over the stable core API.
- Local stdio transport first, with schemas, capability metadata, size limits, timeouts, cancellation, and safe errors.
- Content-only input by default, no network behavior, and documented host/client configuration; any local-path mode is restricted to approved roots.
- Threat model covering path access, prompt-supplied content, symlinks, resource exhaustion, output volume, and accidental data disclosure.
- Protocol conformance, integration, and parity tests plus example configuration and troubleshooting documentation.

### Exit Criteria

- MCP results match direct core results for the same inputs and options.
- Tool schemas are versioned, bounded, deterministic, and return actionable structured errors.
- Default configuration cannot read arbitrary files outside explicitly permitted inputs or send contract data over the network.
- Resource-limit, cancellation, malformed-request, path-traversal, and client compatibility tests pass.
- Installation and local use require no account, database, AI API, hosted service, or additional monetary cost.

### Dependencies

- Stable Phase 2 core contracts and lessons from CLI adapter behavior.
- A supported MCP TypeScript SDK/protocol version and selected compatibility targets.
- Decisions on whether to add path inputs, allowed-root enforcement, transport scope, and package distribution.

### Risks

- The MCP ecosystem and SDK may change rapidly before interfaces stabilize.
- File access exposed through an agent host can cross trust boundaries.
- Large structured findings may exceed client context or message limits.
- Users may incorrectly infer that deterministic comparison requires or invokes AI.

## Phase 7: Production Release

### Deliverables

- Release candidates for the core library, CLI, static web app, GitHub Action, and MCP server with aligned versions and compatibility notes.
- Final security and privacy review, dependency/license audit, accessibility review, performance baseline, and zero-cost verification.
- Reproducible signed or provenance-attested artifacts where free platform support permits, plus checksums and a rollback procedure.
- Complete installation, rule catalog, limitations, API/report schema, CLI, Action, MCP, contribution, security-reporting, and release documentation.
- Public changelog, support policy, versioning/deprecation policy, issue templates, and maintainer runbook.
- Static production deployment, package releases, Action release tag, MCP package release, and post-release smoke tests.

### Exit Criteria

- All PRD acceptance criteria and CI quality gates pass from a clean checkout using documented commands.
- Published artifacts match reviewed source, pass consumer smoke tests, and can be reproduced without a paid service.
- The public app is available without registration and processes specifications locally with no database or runtime backend.
- No unresolved critical/high security, privacy, accessibility, licensing, correctness, or release-blocking issue remains.
- Rollback, vulnerability disclosure, dependency updates, and release ownership are documented and executable by one maintainer.
- Known limitations, deferred rules, and compatibility guarantees are public; release health is checked after publication.

### Dependencies

- Completion of Phases 1–6 and stability of shared public contracts.
- Access to the chosen free static host, package registry, and public-repository release/CI features.
- Final license, governance, versioning, branding, support, and security-contact decisions.

### Risks

- Coordinating multiple packages and channels can exceed one maintainer's sustainable release capacity.
- A late privacy, accessibility, supply-chain, or rule-correctness finding may delay release.
- Free hosting, registry, CI, or provenance features may change terms or availability.
- Early users may treat an incomplete rule catalog as a guarantee of full compatibility.

## Cross-Phase Controls

- Each phase updates the PRD, design, ADRs, rule catalog, threat model, tests, and user documentation when behavior or decisions change.
- No adapter may duplicate comparison rules; parity tests compare every interface with `packages/core`.
- New dependencies require license, maintenance, browser/runtime, security, bundle-size, and zero-cost review.
- Scope changes that add data storage, server processing, external calls, AI runtime behavior, or paid infrastructure require an explicit architecture decision and PRD review.
- Release notes must distinguish implemented capabilities from planned work.

## Assumptions

- One primary developer delivers phases mostly in order, using automation and focused external review where available.
- `packages/core` is the sole comparison authority; the CLI, GitHub Action, MCP server, and web app are adapters or consumers.
- The MVP milestone includes Phases 1–4; the first complete production release follows Phase 7.
- Public-repository CI, package hosting, and static hosting remain available at no additional cost, with local/reproducible fallbacks.
- Dates will be estimated only after Phase 1 reduces uncertainty; exit criteria take priority over calendar targets.

## Unresolved Decisions

- Final MVP compatibility-rule matrix and whether informational findings form a separate class.
- Monorepo/package manager, build, test, lint, formatting, release, and documentation tooling.
- OpenAPI parser/validator, local/bundled `$ref` policy (remote retrieval stays disabled), schema-dialect and mixed-version handling, and canonical report format.
- Performance budgets, input/resource limits, worker cancellation/fallback behavior, supported browsers, and supported Node.js versions.
- Package names, license, semantic-versioning policy, pre-1.0 compatibility guarantees, and release cadence.
- Static hosting provider and custom-domain strategy; no paid domain is required.
- GitHub Action baseline acquisition, annotations, bundling, and marketplace strategy.
- MCP SDK/version, transport, input model, filesystem boundary, and client support policy.
- Whether any aggregate telemetry will ever be introduced; current plan is none.
