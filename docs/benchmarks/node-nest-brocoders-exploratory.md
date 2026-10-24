# Exploratory NestJS corpus: brocoders/nestjs-boilerplate

Status: **exploratory; excluded from plugin precision/recall and maturity metrics**.

This run checks Prumo's NestJS plugin against a public repository that was not
created as a Prumo fixture. It is evidence for rule behavior and noise, not a
benchmark: findings have not been exhaustively adjudicated, and the repository
does not provide ground-truth labels for Prumo's rules.

## Corpus identity

- Repository: <https://github.com/brocoders/nestjs-boilerplate>
- Evaluated commit: `9620f159eefe38f47747d02ab162852367c5472c`
- License declared by upstream: MIT
- Checkout used for this run: `/private/tmp/prumo-bench-nest-brocoders`
- Prumo version: `0.16.0` (local build)
- Plugin: `node-nest`

## Reproduction setup

The checkout was evaluated read-only, with this temporary config:

```ts
import { defineConfig } from '/Volumes/ia/test/prumo/packages/prumo/dist/index.js';
import nest from '/Volumes/ia/test/prumo/packages/plugin-node-nest/dist/index.js';

export default defineConfig({
  plugins: [nest],
  layout: {
    src: ['src'], test: ['test'], oracle: ['oracle'],
    intent: ['.prumo/intent'], contract: ['.prumo/contract'],
    invariants: ['.prumo/invariants'], waivers: [],
  },
  testFilePatterns: ['**/*.spec.ts', '**/*.e2e-spec.ts'],
  oii: { profile: 'fast' },
  budget: { enabled: false },
});
```

Command:

```sh
node /Volumes/ia/test/prumo/packages/prumo/dist/bin.js gate \
  --root /private/tmp/prumo-bench-nest-brocoders \
  --config /private/tmp/prumo-nest-bench.config.ts --json
```

The temporary config and checkout paths are machine-specific. Recreate them
before rerunning; this document does not claim a hermetic benchmark harness.

## Observations

At the evaluated commit, the gate reported:

- **DTO validation:** 2 findings, on `CreateUserDto.provider/socialId` and
  `UpdateUserDto.provider/socialId`. The repository has a global
  `ValidationPipe` configured with `whitelist: true`; these DTOs feed the
  admin-only `POST /users` and `PATCH /users/:id` operations; and the service
  forwards both fields to persistence. Neither property has a
  `class-validator` decorator, so whitelist may strip them before the service
  receives them. The other 10 `@Body()` DTO classes have validator metadata
  (including `@Allow()` for fields meant to pass through). **Adjudication is
  unresolved:** existing admin e2e tests do not submit `provider/socialId`, so
  repository behavior alone cannot establish whether those fields are
  intended writable inputs or stale declarations that should be removed. The
  upstream [authentication notes](https://github.com/brocoders/nestjs-boilerplate/blob/9620f159eefe38f47747d02ab162852367c5472c/docs/auth.md)
  confirm these fields are part of social-login account linking and are set
  from a verified provider identity; they do not say that admin DTOs should
  accept them.
- **Mutation route guard:** 9 findings on public email authentication routes
  (login, registration, confirmation, and password recovery/reset). Those
  operations need to be reachable before authentication. In this corpus they
  are **expected public routes**, so a blanket “all mutations need auth” label
  would be wrong. Route authorization needs intent-aware or explicit public
  route semantics.
- **Cross-feature imports:** 80 findings. These are architecture-policy
  signals, not 80 confirmed defects. No independent project policy was
  available to adjudicate them.
- **Family drift:** 9 similar pairs. The output includes variants that may be
  intentional; none were counted as confirmed defects.
- **Oracle isolation:** 1 test file imports production role/status enums.
  This is a concrete dependency observation, but not an adjudicated oracle
  quality defect without a project-specific testing policy.
- The fast profile reported 83.3% OII. **Do not interpret this as plugin
  accuracy, recall, or benchmark performance.** Test redundancy and typecheck
  were skipped because the temporary config did not enable those checks.

## What this supports

This corpus provides one candidate finding family for `dto-has-validation` and
exposes a clear false-positive class for `mutation-routes-guarded`. It does not
establish precision or recall: the DTO finding still needs intent adjudication,
and most other reported findings lack independent labels. Do not add this run
to `fixtures/bench/plugins.json`, do not raise `node-nest` maturity from it, and
do not use its aggregate OII as plugin quality evidence.

## DTO label status

| Label | Cases | Evidence |
|---|---|---|
| Negative candidates | The 10 other request DTO classes | Each has class-validator metadata on its fields (or `@Allow()`); route e2e tests exercise representative normal inputs. |
| Ambiguous | `CreateUserDto.provider/socialId`; `UpdateUserDto.provider/socialId` | The code declares and forwards the properties, but whitelist strips them and existing admin tests do not send them. Public auth documentation describes internal social-login use, not admin editing. |

No precision/recall number is computed from this partial label set. A maintainer-authored statement or contract establishing admin behavior would resolve the ambiguous cases.

## First RealWorld implementation: lujakob/nestjs-realworld-example-app

This second sample has an independently published API contract and is a better
candidate for a rule-specific labeled corpus.

- Repository: <https://github.com/lujakob/nestjs-realworld-example-app>
- Evaluated commit: `c1c2cc4e448b279ff083272df1ac50d20c3304fa`
- No `LICENSE` file was present in that checkout; the source is fetched
  temporarily for the CI oracle and is not copied into this repository.
- Upstream README states that the app adheres to the RealWorld API spec.
- External contract: [RealWorld backend endpoints](https://realworld-docs.netlify.app/specifications/backend/endpoints/)
- Same Prumo 0.16.0 local build and temporary config as above; root changed to
  `/private/tmp/prumo-bench-nest-realworld`.

The gate discovered 5 request DTOs and reported 3 `dto-has-validation`
findings:

| DTO | Candidate label | Independent contract evidence | Runtime path |
|---|---|---|---|
| `CreateArticleDto` | Positive | `POST /api/articles` requires `title`, `description`, `body`; `tagList` is an optional string array. | `@Body('article')`; no class-validator decorators and no route/global validation pipe. |
| `CreateCommentDto` | Positive | `POST /api/articles/:slug/comments` requires `body`. | `@Body('comment')`; no class-validator decorators and no route/global validation pipe. |
| `UpdateUserDto` | Positive | `PUT /api/user` accepts `email`, `username`, `password`, `image`, `bio`. | `@Body('user')`; no class-validator decorators and no route/global validation pipe. |
| `CreateUserDto` | Negative | Registration requires `username`, `email`, `password`. | Has `@IsNotEmpty()` decorators and a route-level validation pipe. |
| `LoginUserDto` | Negative | Login requires `email`, `password`. | Has `@IsNotEmpty()` decorators and a route-level validation pipe. |

The verified result is **3/3 positive
DTOs and 2/2 predicted negative DTOs** for this small repository slice. It is
recorded in `fixtures/bench/plugins.json`; it comes from one old, compact
implementation, so it cannot establish plugin-wide maturity or recall. The
labels measure the narrow rule “request DTO has no
class-validator metadata at all”; they do not claim that every property in the
two negative DTOs has complete domain validation. A concrete rule-quality issue
also surfaced: the plugin's finding message claims `ValidationPipe` with
`whitelist` strips fields, but the three
flagged routes do not install a validation pipe. The defect here is absent
runtime validation against the external contract, not whitelist stripping.

## Second RealWorld implementation: mikro-orm/nestjs-realworld-example-app

- Repository: <https://github.com/mikro-orm/nestjs-realworld-example-app>
- Evaluated commit: `e723ffb798f7c7c6cad0037b0ca25d05b93db8ef`
- License: MIT (`LICENSE` present in the checkout).
- The README says it adheres to the same RealWorld API specification, is a
  rewrite of the TypeORM implementation above, and points to the upstream
  RealWorld E2E suite.
- It reports the same five request DTOs and the same three positive candidates:
  `CreateArticleDto`, `CreateCommentDto`, and `UpdateUserDto`. The two DTOs
  with validator decorators (`CreateUserDto`, `LoginUserDto`) are clean.
- `main.ts` has no global pipe; the only local pipes are on registration and
  login. The three flagged article/comment/user-update routes do not bind one.

Each implementation separately measures precision 1.00 and recall 1.00 on
three positives and two negatives. Combined, the oracle reports 6 TP and 4 TN,
but marks the aggregate **correlated**: both implementations follow the same
RealWorld contract family, and MikroORM explicitly describes itself as a
rewrite of the TypeORM repository. This is evidence across two ORM-backed
implementations, not two independent specifications. `node-nest` remains
`experimental` until the corpus includes more independent contract families.

For a scored external benchmark, next define the rule-specific label policy,
adjudicate every in-scope finding and a sampled set of unflagged cases, and
make the corpus checkout/configuration reproducible in CI.
