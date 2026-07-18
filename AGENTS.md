# Repository Guidelines

## Scope and Structure

Documents live under `docs/`; `ROADMAP.md` sequences delivery. The monorepo plan uses `apps/` for deployables, `packages/` for reusable code, colocated `*.test.ts` tests, and `test/fixtures` for sanitized examples.

## Architecture Boundaries

- `packages/core` owns parsing, normalization, rules, and result types. It must not depend on UI, Node-only APIs, I/O, telemetry, databases, or AI services.
- `apps/web` processes specifications in the browser. Never transmit specification contents from the app.
- CLI, GitHub Action, and MCP packages are thin adapters over `core`; do not duplicate comparison logic.
- Dependencies point inward toward `core`; presentation and transport concerns must not leak into it. Record cross-cutting choices in `docs/adr/`.

## Coding Expectations

Use strict TypeScript, ESM, two-space indentation, and explicit exports. Use `camelCase` for values/functions, `PascalCase` for types/components, and kebab-case filenames. Results must not depend on clocks, randomness, locales, or unstable iteration. Justify dependencies by maintenance, license, browser, and security impact.

## Testing Expectations

Test behavior at the lowest useful level: table-driven rule tests, adapter integrations, property invariants, and end-to-end journeys. Add regression fixtures for defects. Fixtures must be synthetic or redistributable, with no secrets or proprietary schemas. AI output is never a test oracle. Never weaken a gate to pass it.

## Security and Privacy

Treat specifications as untrusted and sensitive. Bound parsing work; reject malformed input safely; prevent prototype pollution and path traversal; never execute schema content. Do not log, persist, upload, or send specification data to telemetry. Never commit secrets. Production must need no registration, database, paid service, or AI API.

## Documentation and Change Discipline

Update documentation and ADRs with behavior. State assumptions and open decisions; never present plans as implemented. Change only files required by the task. Do not reformat, rename, or clean up unrelated files; preserve user work.

## Commits and Pull Requests

No commit convention exists. Use imperative subjects such as `docs: define reference policy`. Pull requests should explain the problem/approach, link issues/ADRs, list checks, note security, privacy, compatibility, or cost impact, and include screenshots for user-visible UI changes.

## Required Checks

For docs, run `git diff --check` and verify changed links and Mermaid syntax. After the toolchain exists, run these root scripts:

```sh
pnpm format:check
pnpm lint
pnpm typecheck
pnpm test
pnpm build
```

Also run `pnpm test:e2e` for user flows and `pnpm test:security` for trust-boundary changes. Report checks that cannot run; do not install tooling or bypass failures without approval.
