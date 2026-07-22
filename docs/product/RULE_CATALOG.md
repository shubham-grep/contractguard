# ContractGuard Rule Catalog

| Field | Value |
| --- | --- |
| Status | Draft for maintainer review |
| Ruleset version | `0.1.0-draft.2` |
| Draft date | 2026-07-23 |
| Comparison direction | Baseline (existing contract) to candidate (proposed contract) |
| Supported inputs | OpenAPI 3.0.x and 3.1.x YAML or JSON; OAS 3.1 base dialect only |
| Rule count | 16 |

## Purpose and Authority

This catalog defines ContractGuard's initial deterministic compatibility policy. The [OpenAPI 3.0.4 specification](https://spec.openapis.org/oas/v3.0.4.html) and [OpenAPI 3.1.2 specification](https://spec.openapis.org/oas/v3.1.2.html) define document semantics; they do not define a universal meaning of “breaking change.” The classifications below are ContractGuard policy for an existing consumer moving from the baseline API to the candidate API.

A `breaking` finding means the isolated change can invalidate a request previously allowed, remove a documented capability or guarantee, widen a closed response contract beyond what an existing consumer accepts, or require authorization an existing consumer did not need. A `non-breaking` finding means the isolated change preserves existing consumer behavior under the stated guards. A finding-free result means only that no supported rule detected a change. Coverage marked `complete` is complete for this versioned ContractGuard policy, not for every OpenAPI feature, implementation behavior, or possible compatibility interpretation.

## Applicability and Comparison Model

Version 0.1 compares OpenAPI 3.0.x with 3.0.x or 3.1.x with 3.1.x. Patch versions within a supported line share one feature set. Mixed 3.0/3.1 comparisons and OpenAPI 3.2.x inputs are unsupported and return a diagnostic without a compatibility verdict.

For OpenAPI 3.1, version 0.1 supports only the OAS base dialect, `https://spec.openapis.org/oas/3.1/dialect/base`. A reachable unsupported `jsonSchemaDialect` or schema-resource `$schema` produces an `unsupported-schema-dialect` diagnostic; the engine must not silently substitute another dialect.

Before evaluating rules, the engine must:

1. Validate both documents and resolve only supported same-document references without network access.
2. Match endpoints by HTTP method and normalized path-template shape, not `operationId`. Normalization replaces only template-variable names; literal text, case, separators, and trailing slashes remain significant.
3. Match path parameters by their corresponding template-expression position. Match query and cookie parameters by exact `(in, name)` and headers by `(header, lowercase-name)`. Operation-level parameters replace path-level parameters with the same identity but cannot remove inherited parameters.
4. Ignore header parameters named `Accept`, `Content-Type`, or `Authorization`, using a case-insensitive comparison, as required by OAS.
5. Resolve operation-level security overrides and root security into effective authorization alternatives.
6. Expand explicit successful response coverage to concrete codes `200`–`299`. For a concrete code, select an exact key before `2XX`, then `default` as the fallback for otherwise undeclared responses.
7. Evaluate schemas at semantic request or response usage sites, not as context-free component diffs.
8. Compare sets canonically so map order and YAML versus JSON representation do not affect findings.

### Reference and schema subset

Version 0.1 supports same-document, fragment-only JSON Pointer `$ref` values with correct percent and `~0`/`~1` decoding. `$id`, `$anchor`, `$dynamicAnchor`, `$dynamicRef`, external references, and multi-document descriptions are deferred. OAS 3.0 Reference Object siblings are ignored. In OAS 3.1, Reference Object `summary` and `description` remain annotations; assertion-bearing Schema Object `$ref` siblings make the affected simple-rule comparison indeterminate. A Path Item containing both `$ref` and adjacent fields also produces a diagnostic because merge behavior is undefined. Cycles produce a diagnostic only when they prevent the affected comparison.

A **simple effective object schema** has an explicit, unchanged `type: object`, does not enable OAS 3.0 `nullable`, and has property-presence semantics determined only by `properties` and `required` after supported reference resolution. The engine must prove a valid object witness under the supported property subschemas; otherwise the affected comparison is diagnostic. Composition, conditionals, `dependentRequired`, property-name or additional/unevaluated-property assertions, parent-level `const`/`enum`, boolean schemas, custom vocabularies, or another assertion that can require, forbid, or make a property impossible are outside this subset.

A **simple effective string enum** is a reachable schema with unchanged `type: string`. Apart from `enum`, all assertion keywords must be equivalent, and no other assertion may filter the enum members. An omitted `enum` means all strings are allowed. Enum members must be unique strings and compare by exact string value. Broader JSON values, numeric equality, `nullable`, composition, open-enum extensions, and interacting assertions produce a diagnostic.

For version 0.1 schema rules, only `$comment`, `title`, `description`, `example`, `examples`, `default`, and `deprecated` are annotation-only and may differ without blocking a finding. `readOnly` and `writeOnly` follow the direction policy below. `format`, `xml`, `discriminator`, and unknown or custom keywords are not treated as annotation-only; an affected difference produces a diagnostic unless a rule explicitly supports it.

OAS 3.0 `readOnly` and `writeOnly` projections apply to both property-presence and enum rules: `readOnly` properties do not participate in requests, and `writeOnly` properties do not participate in responses. OAS 3.1 treats these fields as application-interpreted annotations, so an affected 3.1 comparison is diagnostic until ContractGuard adopts an annotation policy.

### Security model

Security alternatives are compared by accepted authorization meaning, not raw array entries. The outer array is OR; schemes within one object are AND; OAuth2 and OpenID Connect scope arrays are required sets. Root security that is absent or `[]` means no declared requirement. An absent operation-level field inherits root security, while operation `security: []` removes that inheritance. `{}` permits anonymous access.

Remove any alternative stricter than another already accepted alternative to form a canonical antichain. Adding or removing only a redundant stricter branch is not a change. Scheme `type` must match. Behavior fields are `name` and `in` for API keys; `scheme` for HTTP; flow names, authorization/token/refresh URLs, and declared scopes for OAuth2; and `openIdConnectUrl` for OpenID Connect. Mutual TLS has no additional behavior field. `description` and `bearerFormat` are annotations; changed security extensions or other unlisted fields are diagnostic. OAS 3.0 non-OAuth arrays with values are invalid. OAS 3.1 non-OAuth role arrays are valid but application-defined and therefore diagnostic in version 0.1.

### Findings, diagnostics, and coverage

Findings and diagnostics are separate:

| Condition | `comparisonStatus` | `coverage` | Findings |
| --- | --- | --- | --- |
| Valid, fully supported comparison | `completed` | `complete` | Supported findings |
| Valid comparison with a scoped unsupported construct or change | `completed` | `partial` | Valid findings outside the affected scope |
| Invalid input, unsupported version pair, or resource-limit failure | `failed` | `none` | None |

`hasBreakingChanges` is `true` whenever a breaking finding is detected. It is `false` only when coverage is `complete` and no breaking finding exists; it is `null` for a failed comparison or a partial comparison with none detected. A partial result must never be presented simply as “compatible.” Diagnostics require stable codes, document identity, a safe semantic location, and a reason. Initial top-level codes are `invalid-input`, `unsupported-version`, `unsupported-schema-dialect`, `unsupported-reference`, `ambiguous-semantics`, `unsupported-change`, and `resource-limit`. A versioned diagnostic registry must define condition-level `reasonCode` values before implementation.

Suppression is local. An added or removed endpoint suppresses descendants only for that endpoint. A removed response status, body, parameter, or media type suppresses findings only beneath that element. An unsupported sibling change does not hide an independently provable finding elsewhere.

## Versioning and Rule Stability

`rulesetVersion`, `engineVersion`, and `reportSchemaVersion` are independent and must all appear in machine-readable reports.

- Before the first accepted ruleset, review revisions increment the draft suffix even when behavior changes, for example `0.1.0-draft.2`. Approval removes the suffix and produces `0.1.0`.
- After the first accepted version and before 1.0, an output-affecting policy change increments the minor version; an editorial clarification increments the patch version.
- From 1.0, retiring a rule or changing covered behavior incompatibly increments the major version. Reclassification retires the old ID and introduces a new ID. Adding a rule or extending coverage compatibly increments the minor version. Editorial clarification increments the patch version.
- An accepted rule ID is never renamed, reused, or assigned a different meaning. Classification is deliberately not encoded in the ID.
- A code correction that restores documented behavior changes the engine version, not the ruleset version.

Draft records use **Proposed**. On approval, every accepted record changes to **Introduced: `0.1.0`**. Each accepted rule requires positive, negative, reverse-direction where applicable, OAS 3.0, OAS 3.1, YAML/JSON equivalence, map-order invariance, and supported internal-`$ref` fixtures.

## Record and Finding Format

Every rule records a stable ID, name, category, classification, supported OAS lines, deterministic trigger, policy rationale, example, guards, minimum fixtures, and proposed or introduced version. A finding carries the rule ID, stable variant when applicable, classification, `additionalImpacts`, comparison direction, stable semantic usage location, normalized before/after evidence, and ruleset version.

Semantic usage locations—not component names or source `$ref` paths—define finding identity and order. Optional source pointers support editor navigation but are excluded from canonical serialization. Evidence must be minimal and must not include descriptions, examples, schema `default` values, or complete schemas unless a future rule explicitly requires them.

## Rule Summary

| Rule ID | Name | Category | Classification | Additional impact |
| --- | --- | --- | --- | --- |
| `CG-ENDPOINT-001` | Endpoint removed | Paths and operations | Breaking | — |
| `CG-ENDPOINT-002` | Endpoint added | Paths and operations | Non-breaking | — |
| `CG-PARAM-001` | Parameter requirement tightened | Parameters | Breaking | — |
| `CG-PARAM-002` | Optional parameter added | Parameters | Non-breaking | — |
| `CG-PARAM-003` | Parameter became optional | Parameters | Non-breaking | — |
| `CG-BODY-001` | Request body became required | Request body | Breaking | — |
| `CG-MEDIA-001` | Request media type removed | Media type | Breaking | — |
| `CG-RESP-001` | Successful response status no longer covered | Response | Breaking | — |
| `CG-MEDIA-002` | Response media type removed | Media type | Breaking | — |
| `CG-SCHEMA-001` | Request property became required | Schema | Breaking | — |
| `CG-SCHEMA-002` | Required response property no longer guaranteed | Schema | Breaking | — |
| `CG-ENUM-001` | Request allowed values narrowed | Enum | Breaking | — |
| `CG-ENUM-002` | Request allowed values widened | Enum | Non-breaking | — |
| `CG-ENUM-003` | Response allowed values widened | Enum | Breaking | — |
| `CG-SEC-001` | Effective security tightened | Security | Breaking | — |
| `CG-SEC-002` | Effective security relaxed | Security | Non-breaking | Security weakening |

## Paths and Operations

### CG-ENDPOINT-001 — Endpoint Removed

- **Category:** Paths and operations
- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** A callable baseline endpoint, identified by normalized path template and HTTP method, has no candidate match.
- **Rationale:** Existing consumers can no longer call a documented operation. Prior deprecation does not make removal backward-compatible.
- **Example:** Baseline contains `GET /pets/{petId}`; the candidate has no matching `GET` operation.
- **Guards:** Top-level `paths` operations only. A pure template-variable rename is wire-equivalent and does not create an endpoint finding, but it produces a scoped diagnostic because generated client parameter names can change. Callbacks and webhooks are deferred. Emit one finding per removed method and suppress only that endpoint's descendants.
- **Minimum fixtures:** Removed method; metadata-only change; literal case and trailing-slash differences; pure variable rename diagnostic; reverse direction triggers `CG-ENDPOINT-002`.
- **Proposed:** `0.1.0-draft.1`

### CG-ENDPOINT-002 — Endpoint Added

- **Category:** Paths and operations
- **Classification:** `non-breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** The candidate contains a new HTTP method and normalized path-template pair with no baseline match.
- **Rationale:** An independent operation adds capability without removing an existing consumer's operation.
- **Example:** The candidate adds `POST /pets` while baseline endpoints remain unchanged.
- **Guards:** Analyze overlap only for the same HTTP method. A new concrete route that can shadow a retained baseline template or a genuinely intersecting new template produces a diagnostic. A new template beside a retained concrete path remains non-breaking because the concrete path has precedence. Invalid same-shape templates fail input validation. Suppress child additions only for the new endpoint.
- **Minimum fixtures:** New method; unambiguous path; different-method overlap; template beside retained concrete path; concrete shadowing retained template; ambiguous template intersection; reverse direction triggers `CG-ENDPOINT-001`.
- **Proposed:** `0.1.0-draft.1`

## Parameters

### CG-PARAM-001 — Parameter Requirement Tightened

- **Category:** Parameters
- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** An effective query, header, or cookie parameter is absent or optional in the baseline and `required: true` in the candidate.
- **Rationale:** A request that omitted the parameter was previously valid and is no longer valid.
- **Example:** Query parameter `limit` changes from optional to required; adding a required `X-Tenant-ID` header uses the `added` variant.
- **Guards:** Resolve path/operation inheritance first. Omitted `required` means `false`. Path parameters are excluded because valid OAS path parameters are always required. Ignored reserved headers never participate. Requirement tightening remains provably breaking even when another unsupported change on that parameter also produces a diagnostic.
- **Minimum fixtures:** Optional-to-required; newly required query/header/cookie; optional addition; case-only header rename; ignored reserved headers; reverse optional-to-required triggers `CG-PARAM-003`; reverse newly-required produces a parameter-removal diagnostic.
- **Proposed:** `0.1.0-draft.1`

### CG-PARAM-002 — Optional Parameter Added

- **Category:** Parameters
- **Classification:** `non-breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** The candidate adds an effective query, header, or cookie parameter whose `required` value is `false` or omitted.
- **Rationale:** Existing requests may continue omitting the new parameter.
- **Example:** The candidate adds optional query parameter `includeArchived`.
- **Guards:** Excludes path parameters, ignored reserved headers, invalid identity collisions, and inherited duplicates. Parameter removal is not the inverse rule and remains deferred.
- **Minimum fixtures:** Optional query/header/cookie additions; required addition triggers `CG-PARAM-001`; inherited duplicate; ignored reserved headers; reverse removal produces a diagnostic and no finding.
- **Proposed:** `0.1.0-draft.1`

### CG-PARAM-003 — Parameter Became Optional

- **Category:** Parameters
- **Classification:** `non-breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** The same effective query, header, or cookie parameter changes from `required: true` to `required: false` or omitted.
- **Rationale:** The candidate accepts every previously valid request and additionally permits omission.
- **Example:** Required header `X-Correlation-ID` becomes optional.
- **Guards:** Normalized `schema` or `content`, `style`, `explode`, `allowReserved`, and `allowEmptyValue` must be equivalent; `description`, `deprecated`, `example`, and `examples` may differ. A compound semantic change produces a diagnostic instead. Path parameters and ignored reserved headers are excluded. Removing the parameter remains deferred.
- **Minimum fixtures:** Required-to-optional; omitted-to-false; annotation-only change; serialization change diagnostic; parameter removal diagnostic; reverse direction triggers `CG-PARAM-001`.
- **Proposed:** `0.1.0-draft.1`

## Request Body and Media Types

### CG-BODY-001 — Request Body Became Required

- **Category:** Request body
- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** For `POST`, `PUT`, or `PATCH`, the request body is absent or optional in the baseline and is `required: true` in the candidate.
- **Rationale:** Existing consumers that send no body no longer satisfy the contract.
- **Example:** `POST /pets` changes from an optional body to `requestBody.required: true`.
- **Guards:** The candidate Request Body Object must be valid. Request-body changes on `GET`, `HEAD`, `DELETE`, `OPTIONS`, or `TRACE` produce a diagnostic because their semantics are unclear or method-specific. Empty or unsupported content produces a separate scoped diagnostic but does not suppress the independently provable required-body finding. A removed request body is deferred and suppresses media/schema findings beneath it.
- **Minimum fixtures:** Absent-to-required; optional-to-required; omitted-to-false; required body with empty or unsupported content emits the finding plus a diagnostic; each unsupported method diagnostic; reverse required-to-optional and removal diagnostics.
- **Proposed:** `0.1.0-draft.1`

### CG-MEDIA-001 — Request Media Type Removed

- **Category:** Media type
- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** For `POST`, `PUT`, or `PATCH`, both matched operations have supported, non-empty effective Request Body Objects, and the candidate no longer covers a media type covered by the baseline.
- **Rationale:** Existing consumers may send a representation the candidate no longer documents as accepted.
- **Example:** Baseline accepts `application/json` and `application/xml`; the candidate accepts only `application/json`.
- **Guards:** Version 0.1 supports unparameterized `type/subtype`, `type/*`, and `*/*`, compared case-insensitively by type and subtype. Use symbolic range inclusion and the most-specific applicable key. Parameterized or nonstandard partial-wildcard keys produce a diagnostic. Request-body changes on other methods produce one scoped diagnostic and suppress body descendants. A missing candidate body is a request-body-removal diagnostic, not a media finding. Suppress schema findings only beneath a removed representation.
- **Minimum fixtures:** Concrete removal; wildcard-preserved coverage; wildcard narrowed to concrete; key order; parameterized key diagnostic; missing body diagnostic; unsupported method diagnostic; media addition diagnostic.
- **Proposed:** `0.1.0-draft.1`

## Responses

### CG-RESP-001 — Successful Response Status No Longer Covered

- **Category:** Response
- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** At least one concrete success status explicitly covered by a baseline exact `200`–`299` key or `2XX` range has no effective candidate Response Object selected by exact key, `2XX`, or `default`.
- **Rationale:** ContractGuard treats each explicitly covered successful status as a distinct documented outcome. Removing one can change control flow or eliminate an outcome an existing consumer handles or depends on.
- **Example:** Baseline defines `200`; the candidate defines only `404` and has no `default`.
- **Guards:** Aggregate removed codes deterministically into one finding per operation and compact contiguous ranges. Candidate additions do not offset removals. Exact keys take precedence over `2XX`, and `default` covers otherwise undeclared codes. A candidate `default` therefore prevents a status-removal finding and becomes the effective object for nested comparison. Suppress descendants only for removed statuses. Added or replacement statuses, including a change from exact/`2XX` selection to `default`, also produce a partial-coverage diagnostic.
- **Minimum fixtures:** Last success removed; one of several successes removed under the closed-status policy; `200` to `2XX`; `2XX` to `200`; `200` to `201`; candidate `default` fallback plus diagnostic; exact-over-range precedence; key order; description-only change.
- **Proposed:** `0.1.0-draft.1`

### CG-MEDIA-002 — Response Media Type Removed

- **Category:** Media type
- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** For a concrete successful status explicitly covered in the baseline by an exact `200`–`299` key or `2XX` and covered by the candidate, the effective candidate Response Object no longer covers a media type covered by the effective baseline Response Object.
- **Rationale:** A consumer requesting or decoding that representation may no longer receive a supported response.
- **Example:** Response `200` previously offered `application/json` and `text/csv`; the candidate removes `text/csv`.
- **Guards:** Select the baseline response by exact key then `2XX`; use candidate exact, then `2XX`, then `default`. A baseline `default` alone does not establish successful-response media coverage. Unrelated status changes do not block this rule. Use the media subset from `CG-MEDIA-001`. Aggregate equivalent range-derived findings and suppress schemas only beneath removed representations. `HEAD`, `204`, and `205` body content produces a bodyless-response diagnostic. Error-response media changes are deferred.
- **Minimum fixtures:** Concrete removal; wildcard-preserved coverage; exact/range/default selection; unrelated status change; range aggregation; key order; parameterized key diagnostic; response media addition diagnostic; bodyless-response diagnostics.
- **Proposed:** `0.1.0-draft.1`

## Schemas and Enums

### CG-SCHEMA-001 — Request Property Became Required

- **Category:** Schema
- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** At a matched request usage site, a property absent from baseline effective `required` is present in candidate effective `required` within the simple object subset.
- **Rationale:** Existing request bodies may omit the property and become invalid under the candidate contract.
- **Example:** `required: [name]` becomes `required: [name, email]`.
- **Guards:** Match usage through the enclosing endpoint, parameter or request media type, and property path; component names are not identities. OAS 3.0 `readOnly` projection applies. An affected OAS 3.1 annotation, unsupported assertion, recursion, dialect, enclosing serialization change, or indeterminate parent comparison produces a scoped diagnostic.
- **Minimum fixtures:** New required property; optional-to-required; optional addition; component rename with equivalent use; shared request/response component; OAS 3.0 `readOnly`; OAS 3.1 annotation diagnostic; unsupported assertion diagnostic; reverse-direction diagnostic.
- **Proposed:** `0.1.0-draft.1`

### CG-SCHEMA-002 — Required Response Property No Longer Guaranteed

- **Category:** Schema
- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** At a matched response usage site, a name in baseline effective `required` is absent from candidate effective `required` within the simple object subset.
- **Rationale:** Existing consumers may rely on a baseline-required property and fail when the candidate permits it to be omitted.
- **Example:** Response `required: [id, name]` becomes `required: [id]`.
- **Guards:** JSON Schema `required` guarantees presence independently of `properties`; removing only a property definition does not trigger this rule. Match through endpoint, effective status, media type, and property path. OAS 3.0 `writeOnly` projection applies. OAS 3.1 annotation-dependent and other unsupported cases produce a scoped diagnostic.
- **Minimum fixtures:** Required name removed; required-to-optional; optional removal; property definition removed while still required; shared request/response component; OAS 3.0 `writeOnly`; OAS 3.1 annotation diagnostic; reverse-direction diagnostic.
- **Proposed:** `0.1.0-draft.1`

### CG-ENUM-001 — Request Allowed Values Narrowed

- **Category:** Enum
- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** At a matched request usage site, the candidate's simple effective string-enum set excludes at least one baseline-allowed value, including a change from unrestricted strings to an explicit enum.
- **Rationale:** An existing consumer may send a string the candidate no longer accepts.
- **Example:** Request enum `[draft, published]` becomes `[published]`; unrestricted `type: string` becoming `enum: [published]` is the `became-restricted` variant.
- **Guards:** Enclosing parameter serialization and other supported semantics must be equivalent. OAS 3.0 `readOnly` properties do not participate. OAS 3.1 annotation-dependent cases are diagnostic. If values are both added and removed, emit only this breaking request rule and include both finite deltas as evidence.
- **Minimum fixtures:** Value removed; unrestricted-to-enum; reorder; exact string case; mixed replacement; parameter/body/property use; OAS 3.0 `readOnly`; OAS 3.1 diagnostic; interacting assertion diagnostic; pure finite reverse triggers `CG-ENUM-002`.
- **Proposed:** `0.1.0-draft.1`

### CG-ENUM-002 — Request Allowed Values Widened

- **Category:** Enum
- **Classification:** `non-breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** At a matched request usage site, the candidate's simple effective string-enum set is a strict superset of the baseline set, including removal of an explicit enum restriction.
- **Rationale:** The candidate accepts every previously allowed request string and at least one additional value.
- **Example:** Request enum `[draft]` becomes `[draft, published]`; removing the enum entirely uses the `became-unrestricted` variant.
- **Guards:** Enclosing serialization and other supported semantics must be equivalent. If any baseline value is removed, suppress this rule and emit `CG-ENUM-001`. Direction-projection and unsupported-case handling match `CG-ENUM-001`.
- **Minimum fixtures:** Value added; enum-to-unrestricted; reorder; mixed replacement emits only `CG-ENUM-001`; request contexts; projection and unsupported-case diagnostics; pure reverse triggers `CG-ENUM-001`.
- **Proposed:** `0.1.0-draft.1`

### CG-ENUM-003 — Response Allowed Values Widened

- **Category:** Enum
- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** At a matched successful-response usage site, the candidate introduces a string value outside the baseline's simple effective enum, or removes the baseline enum restriction entirely.
- **Rationale:** A candidate response may contain a value rejected by baseline validation or unknown to an exhaustive generated client.
- **Example:** Response enum `[draft, published]` becomes `[draft, published, archived]`.
- **Guards:** ContractGuard treats supported response enums as closed. Enclosing status, media type, and other supported schema semantics must be matched and equivalent. OAS 3.0 `writeOnly` properties do not participate; affected OAS 3.1 annotations are diagnostic. A mixed replacement emits this rule for the added output values and a separate scoped diagnostic for the deferred narrowing.
- **Minimum fixtures:** Value added; enum-to-unrestricted; reorder; value removal only diagnostic; mixed replacement finding plus diagnostic; shared request/response component; OAS 3.0 `writeOnly`; OAS 3.1 diagnostic; changed parent status/media suppression.
- **Proposed:** `0.1.0-draft.2`

## Security

### CG-SEC-001 — Effective Security Tightened

- **Category:** Security
- **Classification:** `breaking`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** After inheritance and logical subsumption, at least one authorization accepted by the baseline is no longer accepted by the candidate: anonymous access is removed, a nonredundant OR alternative is removed, a scheme is added to an AND requirement, or an OAuth2/OpenID Connect scope is added.
- **Rationale:** An existing consumer may lack a newly required credential, scheme combination, or scope.
- **Example:** Effective security changes from `[{},{oauth:[read]}]` to `[{oauth:[read]}]`, removing anonymous access.
- **Guards:** Use the common security model and compare canonical antichains. Scheme renames or behavior changes are diagnostic. If a comparison both tightens and relaxes alternatives, emit this breaking rule and include both deltas rather than emitting `CG-SEC-002`.
- **Minimum fixtures:** Root absent/empty; operation inheritance/override; anonymous removed; OR branch removed; AND scheme added; OAuth scope added; order and duplicate scopes; redundant branch removal; scheme-change and OAS 3.1 role-array diagnostics; mixed tightening/relaxation.
- **Proposed:** `0.1.0-draft.1`

### CG-SEC-002 — Effective Security Relaxed

- **Category:** Security
- **Classification:** `non-breaking`
- **Additional impact:** `security-weakening`
- **Supported OAS:** 3.0.x and 3.1.x, same-line comparison
- **Trigger:** After inheritance and logical subsumption, the candidate's accepted authorization language is a strict superset of the baseline language by adding a nonredundant alternative or removing a scheme or OAuth2/OpenID Connect scope requirement.
- **Rationale:** Existing authorized consumers remain valid, although the change may weaken the API's security posture.
- **Example:** Effective security changes from `[{oauth:[read]}]` to `[{},{oauth:[read]}]`, adding anonymous access.
- **Guards:** Use the common security model and canonical antichains. If any baseline authorization is lost, emit `CG-SEC-001` instead. Adapters must present the additional security impact prominently; compatibility-non-breaking is not security approval.
- **Minimum fixtures:** Anonymous added; OR branch added; AND scheme removed; OAuth scope removed; order and redundant branches; simultaneous tightening suppresses this rule; security-impact rendering contract.
- **Proposed:** `0.1.0-draft.1`

## Deferred Decisions and Diagnostics

Version 0.1 assigns no rule IDs to the following until their compatibility policy is approved:

- Parameter removal, path-parameter rename source compatibility, request-body removal, optional-body addition, and required-to-optional body changes
- Response status additions and replacements beyond `CG-RESP-001`, error-response media changes, and response media additions
- Request media additions and parameterized or nonstandard wildcard media-type equivalence
- Response enum narrowing, non-string enums, numeric equality, nullability, and open-enum extensions
- Inverse non-breaking property-presence changes, property-definition removal, type/format changes, bounds, `additionalProperties`, and other schema assertions
- `allOf`, `oneOf`, `anyOf`, `not`, discriminators, conditionals, recursion, and dialect-specific keywords
- Server URL changes, callbacks, webhooks, links, response headers, and deprecation lifecycle policy
- External or multi-document references, `$id`, anchors, dynamic references, Path Item `$ref` siblings, mixed OpenAPI 3.0/3.1, and OpenAPI 3.2
- OAS 3.1 request/response projection when `readOnly` or `writeOnly` annotations affect a rule
- Security-scheme type/location/flow changes and OAS 3.1 non-OAuth role-array semantics

A detected deferred or ambiguous change returns `unsupported-change` or the more specific diagnostic, marks coverage `partial`, and does not borrow a nearby rule ID. An implementation may omit diagnostics only for fields explicitly declared annotation-only by this catalog.

## Maintainer Review Checklist

- [ ] Baseline-to-candidate compatibility and request/response variance are correct.
- [ ] OpenAPI 3.0/3.1 same-line scope, OAS 3.1 base-dialect restriction, and explicit 3.2 deferral are acceptable.
- [ ] Each of the 16 classifications, variants, guards, and additional impacts matches intended behavior.
- [ ] Endpoint matching, path-parameter identity, route-shadow diagnostics, and local suppression are acceptable.
- [ ] Response status/default precedence and closed string-enum policy are acceptable.
- [ ] The simple schema, enum, media-type, and internal-reference subsets are implementable by one maintainer.
- [ ] Partial coverage cannot be presented as full compatibility.
- [ ] Security relaxation remains compatibility-non-breaking and visibly security-weakening.
- [ ] The deferred list contains no must-have MVP rule.
- [ ] Versioning, semantic locations, evidence minimization, and fixture requirements are acceptable.

## Version History

| Version | Date | Status | Change |
| --- | --- | --- | --- |
| `0.1.0-draft.2` | 2026-07-23 | Draft | Tightened supported subsets and diagnostics, revised response and enum semantics, and added one response-enum rule. |
| `0.1.0-draft.1` | 2026-07-22 | Draft | Initial catalog containing 15 high-confidence rules. |
