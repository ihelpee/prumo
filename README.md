<p align="left"><img src="docs/assets/logo.svg" alt="Prumo" height="48" /></p>

# Prumo

A deterministic CI gate for codebases written with AI agents. It finds tests that only repeat
what the code already does, and it enforces structural rules that unit tests cannot see.

[![npm](https://img.shields.io/npm/v/@ihelpee/prumo?label=npm)](https://www.npmjs.com/package/@ihelpee/prumo)
[![License](https://img.shields.io/badge/License-proprietary-lightgrey.svg)](LICENSE)

[Português (Brasil)](README.pt-BR.md)

## The problem

When the same agent writes the code and its tests, the tests inherit the code's assumptions.
If the code is wrong, the test is wrong in the same way, and the suite stays green.

This test reads its expectation from the implementation. It is a **mirror**:

```ts
// test/orders.service.spec.ts
import { OrdersService } from '../src/orders/orders.service.js';

it('totals the items', async () => {
  const order = await new OrdersService().create({ items: [{ quantity: 2, unitPriceCents: 1500 }] });
  expect(order.total).toBe(3000);   // whatever the code computes, the test agrees
});
```

The gate reports it:

```text
FAIL  oracle-isolation  testFiles=2 mirrors=1 clean=1
      x oracle-no-src (1) test imports the implementation: it is a mirror, not an oracle.
          · test/orders.service.spec.ts -> ../src/orders/orders.service.js
```

An **oracle** takes its expectation from somewhere the code did not come from: the written
intent, a frozen contract, or real infrastructure. It can fail when the code is wrong:

```ts
// oracle/orders.contract.spec.ts
import type { OrderContract } from '../.prumo/contract/orders.contract.js';
// intent rule 1: every monetary value is an integer number of cents
```

The **Oracle Independence Index (OII)** is the share of test files that are oracles. The
`budget` check fails the build when the suite grows without the OII growing.

## Quick start

```bash
npm install -D @ihelpee/prumo
npx prumo setup          # agent skills, MCP config, .prumo/ scaffold
npx prumo gate           # exit 1 on failure; --json for CI
```

Node ≥ 20.11. No build step: `prumo.config.ts` is loaded directly, in ESM or CommonJS projects.

## What it checks

| Check | What it catches | Blocks |
|---|---|---|
| `oracle-isolation` | tests importing `src/` (relative paths, tsconfig aliases, literal dynamic imports) | yes |
| `invariants` | framework rules: unguarded mutation routes, unvalidated DTOs, cross-feature imports, … | on `error` |
| `contract-frozen` | contract changed after its freeze (hash ledger); contract changed in the same commit as the implementation (warns) | yes |
| `intent-schema` | intent without `id`/`owner`/`status`, duplicated ids, malformed `R-<domain>-NNN` rule ids | yes |
| `trace` | intent rules with no frozen contract or no oracle (warns); contracts citing undeclared rules | yes |
| `provenance` / OII | which tests are independent (isolation + git history, + mutation if enabled) | — |
| `budget` | OII below the floor, or test count up without OII up | yes |
| `context-integrity` | broken paths in `CLAUDE.md`, `AGENTS.md`, skills and rules | yes |
| `typecheck` | your `typecheckCommand` | yes |

The gate uses no LLM and sends no telemetry.

**Limits:**
- Prumo does not write intent or oracles for you. `prumo contract draft` turns your intent rules into a contract draft; a person freezes it.
- Git provenance is a commit-hygiene signal, not proof.
- Mutation testing (the strongest evidence) is off by default.
- Skill read barriers are enforced only with an IDE hook.
- Every framework plugin is `experimental`: Nest and Next rules were checked on four public repositories, Laravel, FastAPI and Go/Gin on none ([external validation](docs/verification.md#external-validation)).

## Configure

```ts
// prumo.config.ts
import { defineConfig } from '@ihelpee/prumo';
import nest from '@ihelpee/prumo-plugin-node-nest';

export default defineConfig({
  plugins: [nest],
  testFilePatterns: ['**/*.spec.ts', 'oracle/**/*.ts'],
  oii: { profile: 'standard' },            // fast | standard | strict
  budget: { enabled: true, minOii: 0.5, minOiiDelta: 0 },
});
```

Starting on an existing codebase: `npx prumo gate --preset starter`.

Layout:

```text
.prumo/intent/      what the system must do (human, non-executable)
.prumo/contract/    frozen boundaries: DTOs, interfaces, schemas
.prumo/invariants/  your own rules (every .ts file is loaded)
.prumo/waivers/     explicit debt, each entry with an expiry date and an approver
oracle/             black-box tests; never import src/
src/                implementation
```

## CI

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0        # required: OII reads git history
- uses: actions/setup-node@v4
  with:
    node-version: 22
- run: npm ci
- run: npx prumo gate --json > prumo-report.json
```

## Plugins

Install the same version as the core; all packages release in lockstep.

**Framework plugins** — rules for the gate:

| Package | Adds |
|---|---|
| `@ihelpee/prumo-plugin-node-nest` | NestJS: guards (decorators, APP_GUARD, auth middleware), DTO validation, module exports, feature boundaries |
| `@ihelpee/prumo-plugin-node-next` | Next.js App Router: Server Action auth, client boundary leaks, input validation, `NEXT_PUBLIC` secrets |
| `@ihelpee/prumo-plugin-node-jest` | Jest: mock tautology analysis, redundant-test audit |
| `@ihelpee/prumo-plugin-php-laravel` | Laravel: guarded routes, FormRequest, parameterized SQL |
| `@ihelpee/prumo-plugin-python-fastapi` | FastAPI: auth dependencies, Pydantic validation |
| `@ihelpee/prumo-plugin-go-gin` | Gin: route guards, error handling, payload binding |
| `@ihelpee/prumo-i18n-pt-br` | Brazilian Portuguese output |

**Workflow plugins (experimental)** — agent workflow around the gate, not part of its verdict.
Their interfaces may change between minor versions:

| Package | Adds |
|---|---|
| `@ihelpee/prumo-plugin-speckit` | spec interviewer and Prompt Zero planning flow |
| `@ihelpee/prumo-plugin-pipeline` | phase timeline and preflight checks for GitHub issue mutations |
| `@ihelpee/prumo-plugin-github-project` | GitHub Projects/Issues client (used by pipeline) |

## Commands

`gate` · `report` · `findings --promote <id>` · `status` · `start` · `allowed` · `setup` ·
`harness sync` · `interview` · `contract draft|freeze|extract|ratify` · `trace` · `badge` ·
`doctor` · `loopcheck` · `mcp`. See `npx prumo --help` and the
[reference](docs/reference.md#commands).

## Documentation

| Doc | For |
|---|---|
| [Concepts](docs/concepts.md) | why mirrors fail, OII, the test pyramid, usage modes, limits |
| [Reference](docs/reference.md) | config, checks, rules, waivers, commands, CI, troubleshooting, upgrading |
| [Verification](docs/verification.md) | each claim with the command that reproduces it |
| [Harness](docs/harness.md) | enforcing skill read barriers in the IDE |
| [Architecture](docs/architecture.md) | packages, core layout, plugin manifest |

## Contributing

See [CONTRIBUTING.md](SUPPORT.md). This repository runs its own gate: oracles are committed
apart from the code they verify, and `pnpm selftest` must pass.

## License

Proprietary software: [Prumo Software License](LICENSE), from version 0.17.0. Earlier versions keep
the license they were published with (MIT or Apache-2.0). The Prumo name and brand: see
[TRADEMARK](docs/TRADEMARK.md). Maintained by [Alex Gotardi](https://github.com/alexgotardi) ·
IHELPEE TECNOLOGIA LTDA.
