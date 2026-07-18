# ContractGuard Product Requirements Document

**Status:** Initial draft
**Last updated:** 2026-07-18
**Product stage:** Pre-implementation

## Product Summary

ContractGuard is an open-source developer tool that compares a baseline OpenAPI 3.x specification with a candidate specification and produces a deterministic report of breaking and non-breaking API changes. It is intended to give API producers and consumers useful compatibility feedback before a change reaches production.

The first public product consists of a reusable TypeScript comparison library, a command-line interface (CLI), and a registration-free static web application. A GitHub Action and Model Context Protocol (MCP) server are planned after the MVP. AI may assist development, review, testing, and documentation, but it must not decide product results or serve as a test oracle.

## Problem Statement

OpenAPI files are often reviewed as text, where formatting noise and document size obscure contract changes. Generic diffs cannot determine whether removing an operation, adding a required parameter, or narrowing a schema affects compatibility. Existing approaches may also require account creation, upload confidential API definitions, add paid infrastructure, or be difficult for one maintainer to operate.

Users need a fast, explainable, and automation-friendly comparison that treats the older specification as the compatibility baseline, keeps sensitive specifications local when used in the browser, and returns the same result for the same inputs.

## Goals and Success Outcomes

- Detect a documented, useful set of breaking and non-breaking OpenAPI 3.0 and 3.1 changes.
- Explain every finding with a stable rule identifier, severity, location, and human-readable reason.
- Support interactive browser use and repeatable local/CI use from one comparison engine.
- Require no registration, database, paid AI API, or paid runtime service.
- Remain understandable and maintainable by one primary developer.

Initial success will be evaluated through fixture coverage, deterministic test results, successful public use, GitHub issues, and adoption signals such as package downloads and repository usage. Privacy-preserving telemetry is not an MVP requirement.

## Target Users

| User | Primary need |
| --- | --- |
| Backend engineers | Verify that an API change will not unexpectedly break consumers. |
| Frontend and BFF engineers | Understand whether a new contract remains compatible with generated or handwritten clients. |
| API platform teams | Apply consistent compatibility policy across repositories and CI pipelines. |
| Open-source maintainers | Review contributed API changes without operating a hosted service. |

## User Stories

- As an engineer, I can select an old and a new YAML or JSON specification in a browser and see compatibility findings without either file leaving my device.
- As a reviewer, I can tell which document is the baseline and which is the candidate before running a comparison.
- As an API producer, I can see whether each finding is breaking or non-breaking and why the rule classified it that way.
- As a developer, I receive actionable parsing, validation, and unsupported-feature diagnostics rather than a partial or misleading report.
- As a CLI user, I can compare two local files, choose human-readable or machine-readable output, and use a documented exit code in automation.
- As a library consumer, I can invoke the same deterministic engine used by every ContractGuard interface.
- As a platform maintainer, I can identify rules by stable IDs and link findings to documentation.
- As a user with a disability, I can complete the browser workflow with a keyboard and assistive technology.

## MVP Scope

The MVP includes:

- A TypeScript core library that parses supported OpenAPI inputs, normalizes comparison-relevant structures, evaluates a versioned rule catalog, and returns structured findings.
- OpenAPI 3.0.x and 3.1.x input in YAML or JSON, with clear errors for invalid documents and unsupported versions.
- Directional comparison from **baseline (old)** to **candidate (new)**.
- Initial rules for paths and operations, parameters, request bodies, responses, media types, schemas, enums, required properties, and security requirements. The exact v1 rule matrix must be approved during discovery.
- Breaking and non-breaking findings with stable rule ID, category, JSON Pointer or equivalent location, summary, explanation, and relevant before/after values where safe and useful.
- Deterministic ordering and a versioned machine-readable result schema.
- A CLI for local files, human-readable and JSON output, and CI-friendly exit codes.
- A responsive static web application with file selection, text paste, clear/reset, summary and filtering, and report export. Processing occurs entirely in the browser.
- Documentation of supported changes, known limitations, privacy behavior, CLI use, and library use.

Local or bundled references are supported only to the extent defined by the approved MVP reference policy. Remote reference retrieval is disabled.

## Explicit Non-Goals

- Swagger/OpenAPI 2.0, AsyncAPI, GraphQL, gRPC, or proprietary contract formats in the MVP.
- Runtime traffic validation, API monitoring, conformance testing, or availability checks.
- Proving business-level or behavioral compatibility that is not represented in the specifications.
- Automatically editing specifications or recommending AI-generated fixes.
- Authentication, user accounts, teams, comments, saved projects, or report history.
- A server-side upload, comparison, proxy, or persistence service.
- A database, paid analytics, paid AI API, or other paid product dependency.
- Code generation, API mocking, documentation hosting, or a general-purpose OpenAPI editor.
- Exhaustive semantic interpretation of every vendor extension.
- GitHub Action and MCP server delivery as conditions of the MVP; they are later roadmap phases.

## Functional Requirements

| ID | Requirement |
| --- | --- |
| FR-01 | The system must accept exactly one baseline and one candidate document in supported YAML or JSON form. |
| FR-02 | It must identify valid OpenAPI 3.0.x and 3.1.x documents and reject unsupported versions with actionable diagnostics. |
| FR-03 | It must never silently compare an input that cannot be parsed or normalized reliably. |
| FR-04 | It must compare the candidate against the baseline using documented, versioned deterministic rules. |
| FR-05 | Each finding must include classification, stable rule ID, affected location, concise explanation, and comparison direction. |
| FR-06 | Results must be returned in a stable order independent of source key ordering and non-semantic formatting. |
| FR-07 | Unsupported or ambiguous constructs must produce explicit diagnostics and must not be silently classified. |
| FR-08 | The web app must perform parsing and comparison locally and work without registration. |
| FR-09 | The web app must not transmit, persist, or include specification contents in telemetry, URLs, or error reports. |
| FR-10 | Users must be able to filter web findings by classification and export a machine-readable report. |
| FR-11 | The CLI must support documented exit codes for success, breaking changes, and tool/input errors. |
| FR-12 | The library, CLI, and web app must use the same rule implementations and result schema. |
| FR-13 | A rule catalog must document the rationale, supported OpenAPI versions, and examples for every rule. |
| FR-14 | Identical semantic inputs must produce byte-equivalent canonical JSON output for a given ContractGuard version and options. |

## Non-Functional Requirements

### Determinism and correctness

- No compatibility classification may call an AI model, depend on network state, current time, locale, or nondeterministic traversal.
- Rules must account for request and response compatibility direction where it differs.
- Rule behavior and result-schema changes must be versioned and covered by regression fixtures.

### Performance and compatibility

- The browser UI must remain responsive during a comparison; long work should not block interaction once an agreed threshold is exceeded.
- A provisional target is to compare two 5 MB specifications in under 3 seconds on a representative maintained desktop browser. This target and input limit require measurement during discovery.
- The web app should support the current and previous major versions of Chromium, Firefox, and Safari. The exact browser policy is unresolved.

### Quality and maintainability

- Packages must have explicit boundaries, typed public contracts, minimal dependencies, and no duplicated comparison logic.
- Automated tests must cover rule behavior, adapters, privacy controls, accessibility, and representative end-to-end paths.
- Public APIs, rule IDs, exit codes, and report schemas must follow an explicit compatibility and deprecation policy before v1.0.

### Usability and accessibility

- The baseline/candidate direction and result classifications must not rely on color alone.
- The public web workflow must target WCAG 2.2 AA and support keyboard-only use, visible focus, meaningful labels, and screen-reader status announcements.

### Security

- Treat every specification as untrusted input. Parsing, rendering, reference resolution, and export must not execute or inject input content.
- Dependencies and release artifacts must be auditable; privileged CI permissions must use least privilege.

## Privacy Requirements

- The web app must parse and compare user-provided specifications entirely within the browser after static assets load.
- Specification contents and derived findings must remain in volatile browser state unless the user explicitly downloads a report.
- No input may be sent to a ContractGuard server, third-party API, analytics provider, crash reporter, or AI service.
- The app must avoid trackers and third-party runtime scripts. Any future telemetry requires a separate privacy review and must never capture contract data.
- Content must not be placed in query strings, fragments, browser history, logs, or persistent web storage by default.
- Error messages must minimize echoed source content. Documentation and the UI must plainly state the local-processing model and any browser limitations.
- Automated tests must verify that the comparison flow makes no network request containing user input.

## Zero-Additional-Cost Requirement

ContractGuard must be buildable, tested, released, and hosted without requiring the maintainer or users to purchase a subscription, API usage, compute, storage, database, or domain. The implementation should use open-source dependencies, GitHub's capabilities available to public repositories, and free static hosting. A free platform quota may be used only when the product remains functional without paid upgrades and has a documented migration or local-run path if the quota or terms change.

The production app must not require a paid AI API or any AI API. Optional developer tools may assist maintainers, but CI and release correctness must remain reproducible with no paid tool. Cost compliance is a release gate and must be rechecked whenever architecture, hosting, analytics, or automation changes.

## MVP Acceptance Criteria

1. A user can compare two valid, supported YAML or JSON OpenAPI specifications through the deployed web app without registering, and receive an ordered report.
2. Browser network inspection and automated tests show that no specification content or derived report is transmitted during selection, parsing, comparison, or export.
3. The approved MVP rule matrix has positive, negative, inverse-direction where applicable, and regression fixtures, and all fixtures pass in library, CLI, and web integration tests.
4. Repeated runs with semantically equivalent inputs produce byte-equivalent canonical JSON for the same engine version and options.
5. Invalid syntax, unsupported versions, unresolved references under the approved policy, and resource-limit failures produce safe, actionable errors without a misleading partial success.
6. The CLI compares local inputs, emits human and JSON reports, and returns the documented exit codes on macOS, Linux, and Windows CI runners.
7. The public workflow passes agreed automated accessibility checks and a documented keyboard/screen-reader smoke test.
8. CI passes formatting, linting, type checking, unit, integration, property-based, end-to-end, security, and build gates defined by the test strategy.
9. The deployable web output is static, requires no database or application backend, and can be hosted at no additional monetary cost.
10. Documentation accurately describes supported OpenAPI versions, rules, limitations, privacy, CLI and library usage, contribution workflow, and security reporting.

## Definition of Done

The MVP is done when all acceptance criteria are evidenced in CI or a release checklist; the core library, CLI, and static web app are versioned and reproducibly released; the documented fixture corpus and supported-rule matrix pass; security, dependency, accessibility, privacy, and license reviews have no unresolved release-blocking finding; the public app is available without registration; and a clean environment can build, test, and deploy the project without a database, paid AI API, or additional monetary cost. Known limitations and deferred GitHub Action/MCP work must be documented rather than hidden.

## Assumptions

- “Breaking” is evaluated from the perspective of an existing consumer moving from the baseline contract to the candidate contract; individual request/response rules may have different variance semantics.
- The MVP comprises `packages/core`, a thin CLI adapter, and a static browser app. The GitHub Action and MCP server reuse the core in later phases.
- Most MVP inputs are self-contained OpenAPI documents; safe local-reference behavior will be specified before implementation.
- A single maintainer can rely on public-repository CI and static hosting tiers without creating a paid operational dependency.
- English is the initial UI, CLI, and documentation language.
- ContractGuard reports compatibility under its documented rules; it does not guarantee that an implementation or client is behaviorally compatible.

## Unresolved Decisions

- The authoritative MVP rule matrix, including how informational changes, deprecations, and OpenAPI 3.0/3.1 dialect differences are represented.
- Whether a baseline and candidate using different OpenAPI minor lines (3.0 versus 3.1) may be compared directly or require an explicit diagnostic.
- Whether `$ref` support includes only internal references or explicitly supplied local bundles; remote retrieval remains disabled.
- The parser, validator, property-based testing library, monorepo tooling, package manager, and static hosting provider.
- Input size, nesting, recursion, and execution-time limits, plus the worker cancellation, timeout, and fallback behavior.
- The canonical JSON report schema, SARIF support, CLI flag names, exit-code values, and compatibility/versioning policy.
- Whether non-breaking additions and purely informational changes need separate display categories while retaining the required breaking/non-breaking report.
- Supported browser and Node.js version ranges and the measured performance baseline hardware.
- License, package scope/names, governance model, and initial release/versioning policy.
- Whether privacy-preserving aggregate telemetry is ever desirable; the MVP assumes none.
