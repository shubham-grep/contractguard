# Test Strategy

> **Status:** initial pre-implementation draft. Tools, commands, thresholds, and paths below are the proposed quality contract, not current repository capabilities.

## Objectives and scope

Testing must demonstrate that ContractGuard is deterministic, correctly classifies OpenAPI changes, preserves user-supplied document privacy, and behaves consistently across `packages/core`, `packages/cli`, `apps/web`, `packages/github-action`, and `packages/mcp-server`. Tests should be fast enough for one maintainer: broad deterministic checks run on every pull request, while larger browser/property suites may run on `main`, releases, or a no-cost schedule.

The planned stack is Vitest for unit and integration tests, fast-check for property-based tests, Playwright for end-to-end tests, and axe-core for automated accessibility checks. Selection is finalized only when the toolchain is introduced.

AI may help propose test cases, review failures, or draft documentation, but a maintainer must review its output. AI-generated expectations cannot be an oracle; rule documentation, independently authored fixtures, and deterministic invariants remain authoritative.

## Taxonomy and naming

Tests live with the code that owns the behavior unless several surfaces consume them. Use `*.test.ts` for unit tests, `*.integration.test.ts` for package-boundary tests, `*.property.test.ts` for generated cases, `*.security.test.ts` for abuse cases, and `*.e2e.spec.ts` under `test/e2e` for black-box journeys. Shared specifications and expected results live only in `test/fixtures`; package-specific helpers stay inside that package's test directory.

Name tests by observable behavior, for example `operation-removed.breaking.test.ts` and `rejects-unsupported-version.integration.test.ts`. Tests should use arrange/act/assert structure and never depend on execution order. Comparison-rule tests use one explicit direction and expected rule IDs. Time, locale, random seed, and platform differences must be injected or fixed.

## Test levels

### Unit tests

Keep unit tests beside source and test pure behavior without network, filesystem, clock, locale, or uncontrolled random-state dependencies.

- Parsers: OpenAPI 3.0/3.1 YAML and JSON, malformed input, duplicate/unsupported constructs, and actionable diagnostics.
- Normalization: canonical keys, equivalent YAML/JSON documents, stable ordering, and preservation of source locations needed in reports.
- Diff rules: every rule has positive, negative, and boundary cases in the baseline-to-candidate direction. Breaking classification must never be inferred from display text.
- Reporting: stable rule identifiers, paths, severity, and byte-stable JSON serialization.
- Adapters: argument/input validation and mapping of core results to CLI exit codes, Action outputs, web messages, and MCP responses.

The core target is at least 95% line/function and 90% branch coverage. Other implemented packages target 80% line/function and 75% branch coverage. Every classification rule requires explicit behavior coverage even when aggregate thresholds pass; thresholds may be ratcheted upward, not silently reduced.

### Integration tests

Exercise boundaries using real parsers and synthetic files:

- parse → normalize → compare → render across JSON/YAML and OpenAPI 3.0/3.1;
- CLI baseline/candidate file handling, diagnostics, stdout/stderr separation, exit codes, and JSON output;
- web UI ↔ Web Worker transfer with network access disabled after application load;
- GitHub Action input/output and annotation behavior in a fixture repository;
- MCP tool schema, transport, error mapping, and deterministic response content from `packages/mcp-server`.

Core integration tests must prove it imports without DOM, Node filesystem/network, or process dependencies.

### End-to-end tests

Playwright will cover the registration-free web journey: select two local files, receive a classified report, inspect errors, and repeat offline after static assets load. Tests must fail if file contents are sent through `fetch`, XHR, WebSocket, or `sendBeacon`. Run Chromium smoke tests per pull request and the supported browser matrix before release.

Black-box CLI tests invoke the packed artifact in a temporary directory. Action tests run a representative comparison workflow without secrets. MCP tests use a real client/transport once the supported transport is chosen. E2E tests assert public behavior, not internal implementation details.

### Property-based tests

Generate bounded, valid OpenAPI fragments and retain every failing seed as a regression fixture. Required invariants include:

- comparing a document with itself yields no changes;
- repeated runs produce identical, stably ordered output;
- key-order and YAML/JSON serialization changes do not alter semantic results;
- adding and then removing the same supported element yields the corresponding directional rules;
- unrelated additions do not change existing findings;
- malformed or adversarial input terminates with a controlled error, never a crash or hang.

Pull requests use fixed seeds and bounded runs; extended runs use recorded seeds on `main` or a no-cost schedule.

## Fixtures and test oracles

Shared black-box fixtures belong under `test/fixtures`, organized by rule ID and case. A case contains `baseline.yaml` or `.json`, `candidate.*`, and an `expected.json` manifest recording rule IDs, classification, API location, and relevant details. Include equivalent-format pairs, invalid documents, `$ref` cases, security regressions, minimal edge cases, and representative multi-path specifications.

Expected semantic records are the primary oracle and must be authored independently of the engine. Snapshots are allowed only for presentation and diagnostics; reviewers must not accept unexplained bulk updates. Findings are compared structurally and in canonical order. All fixtures must be synthetic, minimized from a public report, or licensed for redistribution—never copied from private APIs.

Each approved diff rule needs fixtures for detection, non-detection, boundary values, and inverse direction when request/response variance changes the classification. Invalid-input fixtures cover syntax, unsupported OpenAPI versions, unresolved references, and resource limits. A manifest records the OpenAPI versions, options, expected diagnostics, and canonical output. The same corpus must run through the library and adapters so acceptance criteria are not proven by separate, inconsistent examples.

## Security and privacy checks

- Run dependency vulnerability review, secret scanning, static analysis, and lockfile integrity checks in CI using no-cost tooling.
- Fuzz parser and reference resolution boundaries for YAML alias bombs, deep nesting, prototype pollution, path traversal, remote fetches, oversized input, and resource exhaustion.
- Treat descriptions and names as hostile text; verify UI escaping, safe downloads, terminal control-character handling, GitHub output safety, and MCP argument validation.
- Keep reference resolution offline by default unless a later ADR explicitly permits another mode. Tests must reject unexpected network and filesystem access.
- Assert that browser inputs, filenames, findings, and contents do not enter telemetry, logs, storage, URLs, or outbound requests. No test fixture may contain secrets.

## Accessibility checks

Target WCAG 2.2 AA for the web application. Each pull request runs axe checks on primary states plus keyboard-only tests for file selection, focus order, result navigation, error recovery, and visible focus. Test accessible names, headings, status announcements, contrast, zoom/reflow, and reduced motion. Before a production release, complete a manual screen-reader pass on at least one supported desktop browser; automation does not replace it.

## Performance and reliability

Create a versioned corpus at small, medium, and provisional 5 MB input sizes. Measure parse, normalize, compare, render, peak memory, and Web Worker responsiveness separately on documented reference hardware. The provisional PRD target is two 5 MB documents in under three seconds without freezing interaction; discovery must validate both the target and maximum input size before they become blocking gates.

Regression benchmarks compare against an approved baseline and flag statistically meaningful degradation rather than noisy single runs. Add hard tests for nesting, cycles, reference fan-out, finding count, and time/memory limits once those limits are decided. Cancellation, worker failure, invalid input, and resource exhaustion must produce controlled diagnostics without partial success, stale results, or leaked document content.

## PRD acceptance traceability

| PRD acceptance criteria | Primary evidence |
| --- | --- |
| AC 1, registration-free web comparison | Playwright local-file and paste journeys against the deployable static build |
| AC 2, no transmitted content | Browser request interception plus storage, URL, telemetry, and export privacy assertions |
| AC 3, approved rule matrix | Rule-level positive/negative/inverse fixtures reused by core, CLI, and web tests |
| AC 4, byte-equivalent output | Unit, property, cross-format, cross-process, and cross-platform canonical JSON checks |
| AC 5, safe failures | Parser, reference, resource-limit, fuzz, and UI/CLI diagnostic tests |
| AC 6, CLI behavior | Packed-artifact E2E tests on macOS, Linux, and Windows CI runners |
| AC 7, accessibility | axe, keyboard automation, and documented screen-reader release evidence |
| AC 8, CI gates | Required checks and retained test/coverage reports |
| AC 9, static zero-cost deployment | Build inspection, offline smoke test, and release checklist showing no backend/database |
| AC 10, accurate documentation | Release review against supported-rule, privacy, limitation, and usage docs |

## CI quality gates

The planned root command contract is `pnpm format:check`, `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build`, `pnpm test:e2e`, and `pnpm test:security` as the relevant suites are implemented. These scripts do not exist yet. Required pull-request gates are:

1. formatting, linting, strict type checking, and clean build;
2. unit/integration tests and coverage thresholds;
3. fixed-seed property suite and determinism checks;
4. Chromium E2E smoke and automated accessibility checks once the web app exists;
5. dependency, secret, and static security checks;
6. package/artifact smoke tests and confirmation that generated files are reproducible.

Release gates add the complete supported browser matrix, packed CLI test, Action/MCP tests when those surfaces exist, manual accessibility review, and the extended property suite. Required failures block merging; retries must not conceal flakes. A quarantined flaky test requires a tracked issue, owner, and removal deadline. CI must remain within public GitHub-hosted free allowances and must not require paid services or secrets for routine validation.

## Release testing

Release candidates must be built from a clean checkout with the committed lockfile. Verify reproducible artifacts, package contents, source maps, licenses, checksums/provenance where supported, and absence of fixtures, secrets, or development-only files. Smoke-test the published-shape core package and packed CLI from a temporary consumer project; validate the static web artifact offline and on the configured base path. Later phases add least-privilege fixture-workflow tests for the GitHub Action and protocol compatibility tests for the MCP server.

A release checklist records the full browser/OS matrix, manual screen-reader result, dependency and license review, privacy/network evidence, performance result, documentation review, and confirmation that build, test, release, and hosting need no paid service or AI API. Any accepted exception needs a linked issue, owner, rationale, and expiry date.

## Assumptions and unresolved decisions

Assumptions: strict TypeScript/ESM, pnpm workspaces, a pure core, synthetic fixtures, GitHub Actions CI, deterministic offline comparison, and no production telemetry. Remote `$ref` retrieval is disabled. Vitest, fast-check, Playwright, axe-core, and the listed coverage thresholds are proposals until adopted with the implementation toolchain.

Decisions still required include the supported Node/browser matrix, parser and reference-resolution policy, local/bundled `$ref` behavior, whether mixed OpenAPI 3.0/3.1 comparisons are supported, maximum document/complexity budgets, exact rule catalog and severity model, canonical report schema, Action test harness, MCP transport, performance reference hardware and thresholds, fuzzing/static-analysis tools, release provenance mechanism, and whether initial coverage levels need adjustment after a measured baseline. Each resolution must update this strategy and its relevant fixtures, plus an ADR when the decision is architectural.
