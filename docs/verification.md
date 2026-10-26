# Verification

Every claim Prumo makes, where it lives in the code, and the command that reproduces it. If a
command gives a different result than recorded here, this document is wrong. Open an issue.

Measured on `@ihelpee/prumo 0.13.0`, Node 22, macOS arm64. Build first: `pnpm install && pnpm build`.

## 1. Tests that import the implementation fail the gate

`oracle-isolation` reports any test file whose imports resolve into `layout.src`, as error.
It resolves relative specifiers, tsconfig/jsconfig `paths`/`baseUrl` (nearest config, following
`extends`), literal `import()`/`require()` (template literals included), `jest.mock`/`vi.mock`
and their `requireActual`/`importActual` variants, and loaders made with `createRequire`.

- Code: `checkOracleIsolation` in `packages/core/src/cli/checks.ts`; `extractImports` in
  `packages/core/src/invariants/ast.ts`.
- Reproduce:
  ```bash
  pnpm --filter @ihelpee/prumo-plugin-node-nest fixture:drift-sample:gate # package Nest fixture: mirrors=1 clean=1, FAIL
  npx tsx oracle/check-isolation-alias.ts # 8 mirror shapes, 1 opaque import, 1 clean file
  ```

A computed specifier (`import(p)`) cannot be resolved statically: it is a warning
(`oracle-opaque-import`), and the file does **not** pass OII criterion 1, because its isolation
is unproven.

**Not covered:** a path built at runtime and passed to a literal-free API (e.g. `fs` + `eval`).
The check follows the config nearest to the file, while `tsx` uses the one at the working
directory.

## 2. OII is an intersection of criteria, not an average

- Code: `computeOii` in `packages/core/src/cli/oii.ts`; `computeOiiScore` in
  `packages/core/src/core/ledger.ts` (an empty suite scores 0, not 100%).
- Reproduce:
  ```bash
  node packages/core/dist/bin.js gate --root packages/plugin-node-nest/fixtures/drift-sample --config prumo.config.ts
  ```
  Measured: `OII 0% 0/2`, `isolation 1 · provenance 0`. One of two files passes isolation, and
  the score is still 0 because criterion 2 rejects both. An average would give 25%.

Criterion 2 is anchored on **content**: the anchor of a test is the oldest commit that added the
blob it had when it was last added, under any path. Deleting a test and re-adding the identical
file, or `git mv`, keeps the original provenance. A re-add with changed content anchors anew.

- Code: `provenanceAnchors` in `packages/core/src/core/git.ts`.
- Reproduce: `npx tsx oracle/check-provenance-anchor.ts` (both tricks; only the honest test counts).

Criterion 2 still depends on the environment: a shallow clone or a missing `.git` changes the
number (no `.git` drops the criterion and inflates it). For that reason the oracles assert
`passedIsolation` and mirror counts, never `oiiScore`.

## 3. Only `error` blocks; `--fail-on-warn` changes the exit code, not the verdict

- Code: `statusFromFindings` in `packages/core/src/core/types.ts`; verdict and exit code in
  `packages/core/src/cli/runner.ts` and `cli.ts`.
- Reproduce: on a project whose only finding is a `warn` (e.g. `family-drift`),
  `prumo gate` exits 0 and `prumo gate --fail-on-warn` exits 1, both with `verdict: PASS`.

## 4. Waivers require reason, expiry and approver

- Code: `loadWaivers` and `matchWaiver` in `packages/core/src/core/waivers.ts`; application in
  `packages/core/src/invariants/engine.ts`.
- Behavior: entries missing `rule`, `target`, `reason`, `expires` or `approvedBy` are dropped;
  expired waivers stop exempting and the target is reported again.

## 5. Framework invariants find planted drift

`packages/plugin-node-nest/fixtures/drift-sample` plants drift file by file (each marked in a comment). Measured:

| Rule | Findings |
|---|---|
| `mutation-routes-guarded` | 5 |
| `dto-has-validation` | 1 |
| `no-cross-feature-import` | 1 |
| `providers-exported-when-cross-module` | 1 |
| `guards-resolve` | 1 |
| `family-drift` (warn) | 3 |

- Reproduce: `pnpm --filter @ihelpee/prumo-plugin-node-nest fixture:drift-sample:gate`. Expected counts are asserted in `oracle/fixture-expectations.ts`.
- The same rules load in a `"type": "commonjs"` project: `packages/plugin-node-nest/fixtures/commonjs-sample`, asserted by
  `oracle/check-loader.ts` (`rules=7 ranRules=7`).
- Other stacks: `pnpm oracle --scope next`.

`family-drift` compares files in the same feature directory by line-shingle Jaccard similarity,
threshold 0.6. On the fixture, a real family scores 0.73 and unrelated files about 0.02.

## 6. Loop detection

- Code: `packages/core/src/core/repetition.ts` (defaults `minRepeats: 4`, `maxPeriod: 24`).
- Reproduce:
  ```bash
  printf 'x\nx\nx\nx\n' | node packages/core/dist/bin.js loopcheck   # loop, exit 1
  printf 'x\nx\nx\n'    | node packages/core/dist/bin.js loopcheck   # no loop, exit 0
  pnpm oracle:loop                                                    # real cases in oracle/loop-cases/
  ```
- Limit: only cycles at the end of the input, up to `--max-period` lines, are detected.

## 7. The published artifact works without a build step

- `packages/*/package.json` exports carry `types`, `import` and `require`, because `jiti`
  resolves through `require.resolve`. The bin reads its version from `package.json`.
- Reproduce: `npx tsx oracle/check-loader.ts` runs the built bin as a subprocess against the
  CommonJS fixture; `npx tsx oracle/check-npm-pack.ts` checks what the tarball ships.
- Not automated: installing the published tarball from the registry into a clean project.

## 8. The repository passes its own gate

`prumo.config.ts` scopes the OII to `oracle/**/*.ts`; fixture specs are data for the gate, not
evidence about it.

```bash
pnpm gate:dist
```

Measured with content-anchored provenance: `OII 52.9% 18/34`, above the `minOii` of 0.5. Fifteen
oracles were once re-added in commits without `src/` to raise criterion 2; anchored on content,
they count as provenance gaps again. The previous `100%` was produced by that re-anchoring.

## 9. Skills declare forbidden reads

- Code: `packages/core/src/skills/definitions.ts` (`forbiddenReads` per skill); shape asserted by
  `oracle/check-skills.ts`.
- Limit: barriers are **declared**. They are enforced only through a harness hook; see
  [harness.md](harness.md).

## 10. Orientation is read-only

`prumo status`, `prumo orient` and their MCP tools never write files and never change a gate
verdict.

- Code: `packages/core/src/core/status.ts`, `orientation.ts`.
- Reproduce: `npx tsx oracle/check-fabric-status.ts`, `npx tsx oracle/check-orientation.ts`.

## External validation

Recorded 2026-10-23; the numbers are from that run and change with each repository's default
branch. The gate was run, unchanged except for the fixes below, on four public repositories cloned at
their default branch: two NestJS apps (`nest.config.ts`: node-nest plugin, `src/`) and two
Next.js apps (`next.config.ts`: node-next plugin, `app/ lib/ components/ pages/`). Each run took
under 1.2 s. Every finding below was read against the source.

| Repository | Files | Test files | Findings (after fixes) | Assessment |
|---|---|---|---|---|
| `lujakob/nestjs-realworld-example-app` | 35 | 1 | `oracle-no-src` 1 file; `mutation-routes-guarded` 3; `dto-has-validation` 3; `no-cross-feature-import` 16 | the one test is a mirror; `DELETE users/:slug` has no auth (real bug); register/login are public by design; the 3 DTOs have no validation at all |
| `brocoders/nestjs-boilerplate` | 157 | 13 | `oracle-no-src` 1 file; `mutation-routes-guarded` 9; `dto-has-validation` 2; `no-cross-feature-import` 80; OII 8/13 | the 9 routes are public auth endpoints (login, register, reset) without `@Public()`: by rule design, needs `@Public()` or a waiver; the 2 DTO fields have no decorator |
| `vercel/commerce` | 64 | 0 | `server-action-mutation-guarded` 4; `server-action-input-validated` 4; `client-boundary-leak-prevented` 3 | anonymous cart actions: by design for a storefront; the 3 boundary findings read non-`NEXT_PUBLIC` env in client code, which Next leaves undefined (a latent bug, not a leak) |
| `shadcn-ui/taxonomy` | 116 | 0 | none | — |

**Prumo bugs found by this run, and fixed** (each covered by a fixture and an oracle):

| Plugin | Bug | Effect on the four repos |
|---|---|---|
| node-nest | `@Controller({ path, version })` rendered the object text as the route path | route ids readable |
| node-nest | guards imported from a package (`AuthGuard` from `@nestjs/passport`) reported as missing | `guards-resolve` 13 → 0 |
| node-nest | auth applied with `consumer.apply(AuthMiddleware).forRoutes(...)` not seen | `mutation-routes-guarded` 13 → 3 on realworld |
| node-nest | the closed list of validators missed `MinLength`, `IsUUID`, `ValidateNested`…; now any decorator imported from `class-validator` counts | `dto-has-validation` 4 → 2 on boilerplate |
| node-nest | `utils/`, `helpers/`, `lib/` not treated as shared | `no-cross-feature-import` 127 → 80 |
| node-next | the client graph was followed through `"use server"` modules | `client-boundary-leak-prevented` 10 → 3 on commerce |
| node-next | type-only imports (`import { Post } from "@prisma/client"` used as a type) counted as leaks | taxonomy 4 → 0 |
| node-next | auth checked through a same-file helper was not seen | taxonomy 2 → 0 |

- Reproduce the fixes: `npx tsx oracle/check-nest-routes.ts`, `pnpm oracle --scope next`.

**Still noisy:** `no-cross-feature-import` flags every import between features (80 on the
boilerplate, mostly `auth-*` providers importing `auth`). It enforces an architecture many Nest
apps do not follow; treat it as a policy to opt into. `mutation-routes-guarded` cannot tell a
deliberately public route without `@Public()`.

**Not validated:** projects with
oracle suites written to Prumo's model (none of the four has one).

## What does not exist

| Not available | State |
|---|---|
| Full-repo mutation testing | the engine mutates only the PR delta, with caps and a time budget; use Stryker for audits |
| TypeORM, OpenAPI/Swagger rules | none |
| Read-barrier enforcement without a hook | `forbiddenReads` is declarative |
| Batch context isolation for instantiators | not implemented |
| Typed rule templates for every finding | `findings --promote` generates a route-shaped template |

## Audit it yourself

```bash
pnpm install --frozen-lockfile
pnpm selftest            # build, typecheck, doc paths, all oracles, self-gate
```
