# ContractGuard Rule Catalog

| Field | Value |
| --- | --- |
| Status | Draft for maintainer review |
| Ruleset version | `0.1.0-draft.1` |
| Draft date | 2026-07-19 |
| Comparison direction | Baseline (existing contract) to candidate (proposed contract) |
| Supported inputs | OpenAPI 3.0.x and 3.1.x YAML or JSON |
| Rule count | 15 |

## Purpose and Authority

This catalog defines ContractGuard's initial deterministic compatibility policy. The [OpenAPI 3.0.4 specification](https://spec.openapis.org/oas/v3.0.4.html) and [OpenAPI 3.1.2 specification](https://spec.openapis.org/oas/v3.1.2.html) define document semantics; they do not define a universal meaning of “breaking change.” The classifications below are therefore ContractGuard policy for an existing consumer moving from the baseline API to the candidate API.

A `breaking` finding means the isolated change can invalidate a request previously allowed, remove a documented capability or guarantee, or require authorization an existing consumer did not need. A `non-breaking` finding means the isolated change preserves existing consumer behavior under the stated guards. It is not a guarantee that the entire candidate implementation is compatible because line by line checking and syntax version matching is not performed.

## Applicability and Comparison Model

Version 0.1 applies only to same-line comparisons: 3.0.x to 3.0.x or 3.1.x to 3.1.x. Mixed 3.0/3.1 comparisons produce a diagnostic until dialect normalization is approved.

Before evaluating rules, the engine must:

1. Validate both documents and resolve supported internal `$ref` values without network access using the source OAS version's rules.
2. Match endpoints by HTTP method and normalized path-template shape, not `operationId`. Renaming `{id}` to `{petId}` alone does not create an endpoint change.
3. Merge path-level and operation-level parameters. Parameter identity is `(in, name)`; header names compare case-insensitively. Header parameters named `Accept`, `Content-Type`, or `Authorization` are ignored as OAS requires.
4. Resolve operation-level security overrides and root security into effective alternatives.
5. Normalize successful response coverage to concrete codes `200`–`299`. For each code, an exact response key overrides `2XX`; `default` is not explicit successful coverage.
6. Evaluate schemas at reachable request or response usage sites, not as context-free component diffs.
7. Compare sets canonically so key order, YAML versus JSON, and equivalent version-valid internal-reference layouts do not affect results.

Reference normalization is version-aware. OAS 3.0 Reference Object siblings are ignored. In OAS 3.1, Reference Object `summary` and `description` are annotations, while Schema Object `$ref` siblings participate in JSON Schema evaluation. A Path Item containing both `$ref` and adjacent fields has undefined merge behavior and produces a diagnostic. Cycles that cannot be compared without expansion also produce a diagnostic.

For schema-presence rules, a **simple effective object schema** is one for which the engine can prove that named-property presence is determined by `required` after resolving references. Plain `type`/`properties`/`required` objects and annotation-only fields qualify. Composition, conditionals, `dependentRequired`, parent-level `const`/`enum`, or another assertion that may guarantee or forbid property presence does not qualify. OAS 3.0 `readOnly`/`writeOnly` projections are honored; an OAS 3.1 case using those annotations is diagnostic until ContractGuard's annotation policy is approved.

Security alternatives are compared by accepted authorization meaning, not raw array entries. Each alternative is a set of required schemes and applicable OAuth2/OpenID Connect scopes. Remove any alternative that is stricter than another alternative already accepted, producing a canonical antichain; adding or removing only such a redundant branch is not a change.

When an endpoint is added or removed, emit only its endpoint finding and suppress nested parameter, body, response, and schema findings. Likewise, a removed media type suppresses findings beneath that representation. Unsupported references, ambiguous route overlap, uncertain schema composition, or invalid input produce diagnostics—not guessed findings.

## Versioning and Rule Stability

`rulesetVersion`, `engineVersion`, and `reportSchemaVersion` are independent and must all appear in machine-readable reports.

- Editorial draft revisions increment the prerelease suffix, for example `0.1.0-draft.2`. If this draft is approved unchanged, it becomes `0.1.0`.
- Before 1.0, an output-affecting draft or accepted-policy change increments the minor base version and resets any draft suffix; an editorial change to an accepted version increments the patch version.
- From 1.0, retiring a rule or changing covered behavior incompatibly increments the major version. Reclassification retires the old ID and introduces a new ID; it never mutates the accepted rule. Adding a rule or extending coverage without changing existing cases increments the minor version. Editorial clarifications increment the patch version.
- An accepted rule ID is never renamed, reused, or given a different meaning. Classification is deliberately not encoded in the ID.
- A code correction that restores documented rule behavior changes the engine version, not the ruleset version.

Each accepted rule requires positive, negative, reverse-direction where applicable, OpenAPI 3.0, OpenAPI 3.1, YAML/JSON equivalence, map-order invariance, and internal-`$ref` fixture coverage.

## Record Format

Every rule records a stable ID, name, category, classification, supported OAS lines, deterministic trigger, policy rationale, minimal example, guards, minimum fixtures, and introducing ruleset version. A resulting finding must carry the rule ID, classification, comparison direction, each available baseline or candidate location, normalized before/after evidence, and ruleset version. A rule may use a stable variant such as `added` or `became-required` when multiple syntactic changes have the same compatibility meaning.

## Rule Summary

| Rule ID | Name | Category | Classification |
| --- | --- | --- | --- |
| `CG-ENDPOINT-001` | Endpoint removed | Paths and operations | Breaking |
| `CG-ENDPOINT-002` | Endpoint added | Paths and operations | Non-breaking |
| `CG-PARAM-001` | Parameter requirement tightened | Parameters | Breaking |
| `CG-PARAM-002` | Optional parameter added | Parameters | Non-breaking |
| `CG-PARAM-003` | Parameter became optional | Parameters | Non-breaking |
| `CG-BODY-001` | Request body became required | Request body | Breaking |
| `CG-MEDIA-001` | Request media type removed | Media type | Breaking |
| `CG-RESP-001` | Successful response contract removed | Response | Breaking |
| `CG-MEDIA-002` | Response media type removed | Media type | Breaking |
| `CG-SCHEMA-001` | Request property became required | Schema | Breaking |
| `CG-SCHEMA-002` | Required response property no longer guaranteed | Schema | Breaking |
| `CG-ENUM-001` | Request enum value removed | Enum | Breaking |
| `CG-ENUM-002` | Request enum value added | Enum | Non-breaking |
| `CG-SEC-001` | Effective security tightened | Security | Breaking |
| `CG-SEC-002` | Effective security relaxed | Security | Non-breaking |

## Paths and Operations

### CG-ENDPOINT-001 — Endpoint Removed

- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** A callable baseline endpoint, identified by normalized path template and HTTP method, has no candidate match.
- **Rationale:** Existing consumers can no longer call a documented operation. Prior deprecation does not make the removal backward-compatible.
- **Example:** Baseline contains `GET /pets/{petId}`; the candidate has no matching `GET` operation.
- **Guards:** Top-level `paths` operations only. A path-variable rename is equivalent. Ambiguous route moves and callbacks/webhooks are deferred. Emit one finding per removed method and suppress its descendants.
- **Minimum fixtures:** Removed method; metadata-only change does not trigger; path-variable rename does not trigger; reverse direction triggers `CG-ENDPOINT-002`.
- **Introduced:** `0.1.0-draft.1`

### CG-ENDPOINT-002 — Endpoint Added

- **Classification:** `non-breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** The candidate contains a new HTTP method and normalized path-template pair with no baseline match.
- **Rationale:** An independent operation adds capability without removing an existing consumer's operation.
- **Example:** The candidate adds `POST /pets` while baseline endpoints remain unchanged.
- **Guards:** Do not classify a new path that ambiguously overlaps a baseline static or templated route; emit a diagnostic instead. Suppress all child additions for the new endpoint.
- **Minimum fixtures:** New method on an existing path; new unambiguous path; overlapping route does not classify; reverse direction triggers `CG-ENDPOINT-001`.
- **Introduced:** `0.1.0-draft.1`

## Parameters

### CG-PARAM-001 — Parameter Requirement Tightened

- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** An effective query, header, or cookie parameter is absent or optional in the baseline and `required: true` in the candidate.
- **Rationale:** A request that omitted the parameter was previously valid and is no longer valid.
- **Example:** `limit` in `query` changes from `required: false` to `required: true`; adding a new required `X-Tenant-ID` header is the same rule with an `added` variant.
- **Guards:** Resolve path/operation inheritance first. Omitted `required` means `false`. Path parameters are excluded because valid OAS path parameters are always required. Header identity is case-insensitive, and ignored reserved header parameters never participate.
- **Minimum fixtures:** Optional-to-required; newly required; optional addition does not trigger; case-only header rename does not trigger; each ignored reserved header does not trigger. Reversing optional-to-required triggers `CG-PARAM-003`; reversing newly-required produces a deferred parameter-removal diagnostic.
- **Introduced:** `0.1.0-draft.1`

### CG-PARAM-002 — Optional Parameter Added

- **Classification:** `non-breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** The candidate adds an effective query, header, or cookie parameter whose `required` value is `false` or omitted.
- **Rationale:** Existing requests may continue omitting the new parameter.
- **Example:** The candidate adds optional query parameter `includeArchived`.
- **Guards:** Excludes path parameters, ignored reserved headers, parameter identity collisions, and simultaneous serialization changes. Parameter removal is not the inverse rule and remains deferred.
- **Minimum fixtures:** Optional query/header/cookie additions; required addition triggers `CG-PARAM-001`; inherited duplicate and each ignored reserved header do not trigger; reverse removal produces a deferred diagnostic and no finding.
- **Introduced:** `0.1.0-draft.1`

### CG-PARAM-003 — Parameter Became Optional

- **Classification:** `non-breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** The same effective query, header, or cookie parameter changes from `required: true` to `required: false` or omitted.
- **Rationale:** The candidate accepts every previously valid request and additionally permits omission.
- **Example:** Required header `X-Correlation-ID` becomes optional.
- **Guards:** All other parameter semantics must remain equivalent. Path parameters and ignored reserved headers are excluded. Removing the parameter entirely is a different, deferred change.
- **Minimum fixtures:** Required-to-optional; omitted-to-false does not trigger; parameter removal does not trigger; reverse direction triggers `CG-PARAM-001`.
- **Introduced:** `0.1.0-draft.1`

## Request Body and Media Types

### CG-BODY-001 — Request Body Became Required

- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** An operation's request body is absent or has `required: false`/omitted in the baseline and has `required: true` in the candidate.
- **Rationale:** Existing consumers that send no body no longer satisfy the contract.
- **Example:** `POST /pets` changes from an optional body to `requestBody.required: true`.
- **Guards:** Apply only where the HTTP method has defined request-body semantics. Methods for which OAS describes body semantics as unclear produce a diagnostic.
- **Minimum fixtures:** Absent-to-required and optional-to-required trigger. Omitted-to-false does not. Reversing optional-to-required produces no v0.1 finding or diagnostic; reversing absent-to-required produces a deferred request-body-removal diagnostic.
- **Introduced:** `0.1.0-draft.1`

### CG-MEDIA-001 — Request Media Type Removed

- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** A matched operation's candidate request-body `content` no longer covers a media type covered by the baseline.
- **Rationale:** Existing consumers may send a representation the candidate no longer documents as accepted.
- **Example:** Baseline accepts `application/json` and `application/xml`; the candidate accepts only `application/json`.
- **Guards:** Normalize media types and ranges. Candidate `application/*` still covers baseline `application/json`. If range or parameter equivalence cannot be proven, emit a diagnostic. Suppress schema findings under a removed representation.
- **Minimum fixtures:** Concrete removal; wildcard-preserved coverage; key reordering does not trigger; media addition produces no v0.1 finding or diagnostic.
- **Introduced:** `0.1.0-draft.1`

## Responses

### CG-RESP-001 — Successful Response Contract Removed

- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** The baseline operation has an explicit `200`–`299` response code or valid `2XX` range, while the matched candidate operation has no explicit successful response coverage.
- **Rationale:** The candidate no longer documents any successful outcome on which an existing consumer can rely.
- **Example:** Baseline defines response `200`; the candidate defines only `404` and `default`.
- **Guards:** `default` alone is not treated as an explicit success contract. If both documents retain success coverage but their concrete coverage sets differ, emit a deferred response-status diagnostic instead. Response additions and replacements require the broader response-status policy.
- **Minimum fixtures:** Last success removed; description-only change does not trigger. `200` to `2XX`, `2XX` to `200`, and `200` to `201` produce a deferred diagnostic but no `CG-RESP-001` finding. Reversing the positive case by adding the first explicit success also produces a deferred response-status diagnostic and no finding.
- **Introduced:** `0.1.0-draft.1`

### CG-MEDIA-002 — Response Media Type Removed

- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** With identical concrete successful-status coverage, an effective candidate response `content` map no longer covers a media type covered by the effective baseline response.
- **Rationale:** A consumer requesting or decoding that representation may no longer receive a supported response.
- **Example:** Response `200` previously offered `application/json` and `text/csv`; the candidate removes `text/csv`.
- **Guards:** For each concrete status, select the exact response key before `2XX`; never use `default` as explicit success coverage. Normalize media types and wildcard coverage as for `CG-MEDIA-001`. Skip `HEAD`, `204`, and `205` body content with a bodyless-response diagnostic. Differing status coverage or ambiguous negotiation also produces a diagnostic instead of this rule.
- **Minimum fixtures:** Concrete removal; wildcard-preserved coverage; exact-over-`2XX` precedence; key reordering does not trigger; response media addition produces no v0.1 finding or diagnostic; `HEAD`/`204`/`205` content produces a diagnostic but no finding.
- **Introduced:** `0.1.0-draft.1`

## Schemas and Enums

### CG-SCHEMA-001 — Request Property Became Required

- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** In a simple effective request object schema, a property absent from baseline `required` is included in candidate `required`, whether the property is new or was previously optional.
- **Rationale:** Existing request bodies may omit the property and become invalid under the candidate contract.
- **Example:** `required: [name]` becomes `required: [name, email]`.
- **Guards:** Evaluate only simple effective object schemas at request usage sites. For OAS 3.0, ignore properties effective only as `readOnly`; for OAS 3.1, annotation-dependent direction produces a diagnostic. Other composition, property-presence assertions, ambiguous recursion, and mixed dialects are diagnostic.
- **Minimum fixtures:** New required property; optional-to-required; optional addition does not trigger. OAS 3.0 `readOnly` does not trigger; OAS 3.1 annotation-dependent input is diagnostic. Reverse direction produces no v0.1 finding or diagnostic.
- **Introduced:** `0.1.0-draft.1`

### CG-SCHEMA-002 — Required Response Property No Longer Guaranteed

- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** In a simple effective response object schema, a name in baseline `required` is no longer present in candidate `required`.
- **Rationale:** Existing consumers may safely assume the baseline-required property exists and fail when it is omitted.
- **Example:** Response `required: [id, name]` becomes `required: [id]`.
- **Guards:** JSON Schema `required` guarantees presence independently of `properties`; removing only the property definition does not trigger this rule. Evaluate only simple effective object schemas at response usage sites. For OAS 3.0, ignore properties effective only as `writeOnly`; for OAS 3.1, annotation-dependent direction is diagnostic.
- **Minimum fixtures:** Name removed from `required`; required-to-optional; optional removal does not trigger; property definition removed while its name remains required does not trigger this rule. OAS 3.0 `writeOnly` does not trigger; OAS 3.1 annotation-dependent output is diagnostic. Reverse direction produces no v0.1 finding or diagnostic.
- **Introduced:** `0.1.0-draft.1`

### CG-ENUM-001 — Request Enum Value Removed

- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** A simple request parameter, body, or property schema has explicit enums in both documents and at least one baseline enum value is absent from the candidate enum.
- **Rationale:** An existing consumer may send a value that the candidate no longer accepts.
- **Example:** Request enum `[draft, published]` becomes `[published]`.
- **Guards:** Compare enum values by canonical JSON equality, not text order. Other constraints must remain equivalent. If values are also added, emit only this breaking rule and include both deltas as evidence. Open-enum extensions, composition, unrestricted-to-enum changes, and response contexts are deferred.
- **Minimum fixtures:** One value removed; reorder does not trigger; duplicate-equivalent JSON representation does not trigger; pure reverse direction triggers `CG-ENUM-002`; a mixed replacement emits only `CG-ENUM-001` with added/removed evidence.
- **Introduced:** `0.1.0-draft.1`

### CG-ENUM-002 — Request Enum Value Added

- **Classification:** `non-breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** A simple request parameter, body, or property schema has explicit enums in both documents and the candidate enum is a strict superset of the baseline enum.
- **Rationale:** The candidate accepts every previously allowed value and at least one additional value.
- **Example:** Request enum `[draft]` becomes `[draft, published]`.
- **Guards:** Applies only in request context with otherwise equivalent constraints. If any baseline value is removed, suppress this rule and emit `CG-ENUM-001` instead. Response enum additions are deferred because exhaustive clients may not tolerate new output values.
- **Minimum fixtures:** One value added; reorder does not trigger; response enum addition produces a deferred diagnostic; pure reverse direction triggers `CG-ENUM-001`; a mixed replacement emits only `CG-ENUM-001`.
- **Introduced:** `0.1.0-draft.1`

## Security

### CG-SEC-001 — Effective Security Tightened

- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** After inheritance and logical subsumption, at least one authorization accepted by the baseline is no longer accepted by the candidate: anonymous access is removed, a nonredundant OR alternative is removed, a scheme is added to an AND requirement, or an OAuth2/OpenID Connect scope is added.
- **Rationale:** An existing consumer may lack a newly required credential, scheme combination, or scope.
- **Example:** Effective security changes from `[{},{oauth:[read]}]` to `[{oauth:[read]}]`, removing anonymous access.
- **Guards:** Operation security overrides root security; the outer array is OR, schemes inside an object are AND, `{}` permits anonymous access, and `security: []` removes inherited requirements. Compare canonical antichains, not raw branch membership. Referenced scheme definitions must be equivalent. Non-OAuth role arrays and scheme-definition changes are deferred.
- **Minimum fixtures:** Anonymous removed; nonredundant OR branch removed; AND scheme added; OAuth scope added; ordering and removal of a redundant stricter branch do not trigger; mixed tighten/relax emits this rule only.
- **Introduced:** `0.1.0-draft.1`

### CG-SEC-002 — Effective Security Relaxed

- **Classification:** `non-breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** After inheritance and logical subsumption, the candidate's accepted authorization language is a strict superset of the baseline language by adding a nonredundant alternative or removing a scheme/scope requirement.
- **Rationale:** Existing authorized consumers remain valid, although the change may weaken the API's security posture.
- **Example:** Effective security changes from `[{oauth:[read]}]` to `[{},{oauth:[read]}]`, adding anonymous access.
- **Guards:** Use the same canonical-antichain model as `CG-SEC-001`. If any baseline authorization is lost, emit `CG-SEC-001` instead, even when the candidate also adds alternatives. `non-breaking` is a compatibility classification, not security approval.
- **Minimum fixtures:** Anonymous added; nonredundant OR branch added; AND scheme removed; OAuth scope removed; ordering and addition of a redundant stricter branch do not trigger; simultaneous tightening does not emit this rule.
- **Introduced:** `0.1.0-draft.1`

## Deferred Decisions and Diagnostics

Version 0.1 assigns no rule IDs to the following until their compatibility policy is approved:

- Parameter removal, path-parameter renames, and request-body removal
- Replacement or addition of response status codes, including `default` and partial `2XX` coverage
- Response enum additions/removals and strict-versus-tolerant decoder assumptions
- Schema property-definition removal with unchanged presence, type/format changes, numeric/string/array bounds, nullability, `additionalProperties`, and open enums
- `allOf`, `oneOf`, `anyOf`, `not`, discriminators, conditionals, recursion, and dialect-specific keywords
- Server URL changes, callbacks, webhooks, links, response headers, and deprecation lifecycle policy
- External or multi-document references, ambiguous Path Item `$ref` siblings, and mixed OpenAPI 3.0/3.1 comparisons
- OAS 3.1 request/response projection when `readOnly` or `writeOnly` annotations affect a rule
- Security scheme type/location/flow changes and non-OAuth role-array semantics

An unsupported or ambiguous case must return a stable diagnostic with its location and reason. It must not silently pass or borrow a nearby rule ID.

## Maintainer Review Checklist

- [ ] The baseline-consumer compatibility perspective is correct.
- [ ] Same-line 3.0.x and 3.1.x scope is acceptable for the first ruleset.
- [ ] Each of the 15 classifications and guards matches intended product behavior.
- [ ] Endpoint matching and child-finding suppression are acceptable.
- [ ] The conservative response-status and schema-composition limits are acceptable.
- [ ] Security relaxation should remain non-breaking while clearly not implying security approval.
- [ ] The deferred list contains no must-have MVP rule.
- [ ] Versioning and fixture requirements are acceptable.

## Version History

| Version | Date | Status | Change |
| --- | --- | --- | --- |
| `0.1.0-draft.1` | 2026-07-22 | Draft | Initial catalog containing 15 high-confidence rules. |
