# Reference

Configuration, checks, rules, waivers, commands, CI and troubleshooting. For the reasoning
behind them, see [concepts.md](concepts.md).

## Install

```bash
npm install -D @ihelpee/prumo        # or: pnpm add -D @ihelpee/prumo
npx prumo setup                      # skills, MCP config, .prumo/ scaffold
npx prumo gate
```

Requires Node ≥ 20.11. Works in `"type": "module"` and `"type": "commonjs"` projects:
`prumo.config.ts` and rule files are loaded with `jiti`, so no `tsx` or build step is needed.

## Configuration

`prumo.config.ts` (or `.mts` / `.js`) at the project root, default export:

```ts
import { defineConfig } from '@ihelpee/prumo';
import nest from '@ihelpee/prumo-plugin-node-nest';

export default defineConfig({
  plugins: [nest],
  testFilePatterns: ['**/*.spec.ts', 'oracle/**/*.ts'],
  typecheckCommand: 'tsc --noEmit -p tsconfig.json',
  oii: { profile: 'standard' },
  budget: { enabled: true, minOii: 0.5, minOiiDelta: 0 },
});
```

A partial config is merged over the defaults; `layout`, `budget`, `testAudit` and `context` are
merged key by key. Precedence: defaults < `preset` in the file < the file < `--preset` on the CLI.

| Field | Default | Effect |
|---|---|---|
| `layout.src` | `['src']` | implementation roots. A test importing these is a mirror |
| `layout.oracle` | `['test', 'oracle', 'e2e']` | used by `src-no-oracle` (implementation importing test code). Does **not** select test files |
| `layout.contract` | `['.prumo/contract']` | frozen contract; empty → `contract-frozen` skips |
| `layout.invariants` | `['.prumo/invariants']` | every `.ts` file under these dirs is loaded as rules |
| `layout.intent` | `['.prumo/intent']` | human intent; read by `status`, `interview` and context checks |
| `layout.waivers` | `['.prumo/waivers']` | conventional location for waiver files |
| `layout.context` | `.prumo/intent`, `.cursor/rules`, … | dirs scanned by `context-integrity` (root `CLAUDE.md`, `AGENTS.md`, `.cursorrules` are always included) |
| `testFilePatterns` | `**/*.spec.ts`, `**/*.test.ts`, `**/*.e2e-spec.ts`, `test/**/*.ts`, `oracle/**/*.ts` | the OII denominator. Oracle files under `oracle/` count even without a `.spec`/`.test` suffix |
| `ignore` | `[]` | globs excluded from every check |
| `plugins` | `[]` | framework plugins (rules, extractors, CLI commands, skills). The TypeScript engine, `@ihelpee/prumo-plugin-typescript`, gives the rules and checks the source model (AST) and the mutation planner. The `setup` template includes it. Without it, a rule that needs source data does not run and the gate reports `ast-provider-missing` |
| `invariants` | — | inline rules, merged with plugin rules and rule files |
| `onDuplicateRule` | `'error'` | `error` \| `project-wins` \| `plugin-wins` when two sources define the same rule id |
| `typecheckCommand` | — | run with `sh` at the root; absent → `typecheck` skips |
| `testCommand` | — | used only by the mutation engine |
| `oii.profile` | `'standard'` | `fast` \| `standard` \| `strict` (see [concepts.md](concepts.md#oracle-independence-index-oii)) |
| `budget` | `{ enabled: true, minOii: 0.35, minOiiDelta: 0 }` | OII floor and growth rule. Optional `minIntentCoverage` (0..1) turns low intent coverage into a `trace` error; optional `minContextCoverage` (0..1) turns low context coverage into a `context-integrity` error |
| `approval` | `{ profile: 'standard' }` | how a contract freeze proves a person approved it: `standard` (default) asks the person to type the approver's name at the terminal; `fast` records the freeze as unverified and the gate warns (`approval-unverified`); `strict` is not available in the core |
| `pluginConcurrency` | cores − 1, at least 1 | how many plugin processes run at once in a dispatch. Plugins run together and one failure never cancels the others |
| `contextCoverage` | — | which folders count for context coverage: `include` keeps only the folders matched, `exclude` drops them. Globs over project-relative paths; a plain path also covers the folders inside it. Without it, the default folders count, up to two levels deep |
| `failOnWarn` | `false` | exit 1 on `warn` findings; `--fail-on-warn` overrides |
| `mutation` | on in CI when `testCommand` is set, off locally | delta mutation engine (`CI` env var decides the default): `enabled`, `base`, `timeoutMs` (5000), `maxMutants` (30), `maxPerFile` (10), `perMutantTimeoutMs` (2000), `operators` |
| `testAudit` | `{ enabled: true, failOnRedundant: false }` | redundant-test audit. The engine comes from a plugin (e.g. `plugin-node-jest`); without one, `test-redundancy` skips |
| `context` | `{ enabled: true }` | AI context file audit; `ignore` for exceptions |
| `ledgerFile` | `'.prumo/ledger.json'` | last report (baseline for deltas) and promotion history |
| `locale` | `'en'` | `'pt-BR'` needs `@ihelpee/prumo-i18n-pt-br` |
| `prumoVersion` | — | the gate errors if the running bin differs |
| `preset` | — | `starter` \| `strict` |
| `extractors` | `[]` | contract extractors for legacy code, used by `prumo contract extract` (see [Intent to contract](#intent-to-contract)) |
| `checks` | `[]` | extra gate checks (extension point `check`) |
| `reporters` | `[]` | receive the finished report after every gate run (extension point `reporter`) |
| `oiiCriteria` | `[]` | extra independence criteria: a test counts toward the OII only if each accepts it |
| `pipeline` | — | `disabled` check ids (reported as skipped) and per-check `levels` (`warn` downgrades errors). `oracle-isolation` cannot be disabled |

Presets:

| Preset | Budget | `failOnWarn` | OII profile | Mutation |
|---|---|---|---|---|
| `starter` | `minOii 0.2`, `minOiiDelta -0.05` | off | `fast` | off |
| `strict` | `minOii 0.6`, `minOiiDelta 0` | on | `strict` | on |

## The gate

`prumo gate` runs these checks in order (cheap and structural first):

| Check | What it does | Blocks on |
|---|---|---|
| `typecheck` | runs `typecheckCommand` | non-zero exit |
| `oracle-isolation` | test files importing `layout.src` (relative, tsconfig `paths`/`baseUrl`, literal `import()`/`require()`); `src` importing test code; non-literal dynamic imports (warn). Needs the TypeScript plugin for the source model | `oracle-no-src`, `src-no-oracle`, `ast-provider-missing` |
| `contract-frozen` | every contract matches the hash in `freeze-ledger.json`; contract files added in a commit that also touched `src/` (warn); unfrozen contracts (warn). `_draft/`, `_observed/` and `_ratified/` are not contracts | a contract changed after its freeze |
| `trace` | intent rule ↔ frozen contract ↔ oracle; missing links warn | a contract citing an undeclared rule; coverage below `budget.minIntentCoverage` |
| `intent-schema` | intent frontmatter (`id`, `owner`, `status`), unique intent ids, `R-<domain>-NNN` rule ids unique and well formed | any violation |
| `context-integrity` | broken paths and case mismatches in AI context files; coverage below `budget.minContextCoverage` | errors |
| `harness-policy` | harness meta missing or stale; `prumoVersion` mismatch | version mismatch |
| `test-redundancy` | tests that only re-check the framework | when `failOnRedundant` |
| `invariants` | all rules from plugins, inline config and rule files. A rule that needs source data runs only with the TypeScript plugin | `error` findings, `ast-provider-missing` |
| `mutation` | delta mutation (when enabled). The operators come from the TypeScript plugin; without it the check skips with a reason | per preset |
| `provenance` | OII breakdown; lists `provenance-gap` files not covered by a waiver | — (warn) |
| *extension* | each `config.checks` entry, under its own id | its `error` findings |
| `budget` | OII floor and growth rule | below floor, or growth without OII growth |

Statuses: `ok`, `FAIL`, `skip`. A `skip` is never silent: it always carries a reason, and it
never counts as a pass. Only `error` findings fail a check; `warn` findings are shown, counted
and promotable, and fail the exit code only with `--fail-on-warn` (the JSON `verdict` does not
change). In human output, `x` marks an error and `~` a warning.

Side effects: writes `.prumo/last-report.json` and updates `ledgerFile`. The JSON report
(`--json`) is the stable interface: parse it, not the human output.

Exit codes: `0` on PASS, `1` on FAIL (or on warnings with `--fail-on-warn`), `2` on usage
errors such as an unknown `--preset`.

## Rules

Rules come from plugins (see the table in the [README](../README.md#plugins)) and from `.ts`
files under `layout.invariants`. A rule returns only the violators:

```ts
// .prumo/invariants/route-timeout.ts
import { rule, type RuleContext } from '@ihelpee/prumo/invariants';

export default [
  rule('route-timeout')
    .severity('warn')                 // 'error' (default) | 'warn'
    .needsData('routes')              // AST data the rule consumes
    .message('external route without a declared timeout')
    .waivers('.prumo/waivers/timeout.json')
    .assert((ctx: RuleContext) =>
      ctx.routes
        .filter((r) => /* the pattern, not one instance */ false)
        .map((r) => ({ target: `${r.id} (${r.controller}.${r.handler})`, location: `${r.file}:${r.line}` })),
    )
    .build(),
];
```

`needsData` values: `routes`, `dtos`, `imports`, `contractFields`, `modules`, `injectables`. The
engine extracts each source once, and only if some rule asks for it.

Guidelines: filter for the *pattern*, not a specific instance (that is a test, not an
invariant). Make `target` readable in review, because waivers match it exactly. Use `warn` for
heuristics. A rule that flags correct code is the rule a team disables.

A rule that throws does not fail the gate: it is counted in `metrics.erroredRules`, with the
error in the JSON. Check that metric in CI.

### Promoting findings

```bash
npx prumo findings                       # promotable findings from the last report
npx prumo findings --promote <id>        # writes .prumo/invariants/<id>.ts
```

The generated rule is narrow by construction: it lists the targets known at promotion time. If
the same class of finding comes back, widen the filter to the class predicate; do not add a
waiver. The generated template is route-shaped, so rewrite `assert` for non-route findings.
`prumo report` shows recurrence per promoted class.

## Waivers

A waiver is countable debt, not an invisible exception. It is a JSON array (or
`{ "waivers": [...] }`) attached to rules with `.waivers(file)` or `withWaivers(rules, file, ids?)`:

```json
[
  {
    "rule": "monetary-is-integer",
    "target": "Order.total: number",
    "reason": "legacy column; migration to cents in ORD-412",
    "expires": "2026-12-31",
    "approvedBy": "alex"
  }
]
```

| Field | Required | Meaning |
|---|---|---|
| `rule` | yes | rule id, or `"*"` |
| `target` | yes | exact match with the finding target; with `prefix: true`, a path prefix |
| `reason` | yes | why the violation exists |
| `expires` | yes | ISO date; after it, the violation counts again |
| `approvedBy` | yes | who approved it |
| `prefix` | no | match by directory prefix |

Waivers for `provenance-gap` live in any JSON file under `layout.waivers` and match the test
file path. A waived gap still does not count toward the OII; only the warning is lifted.

An entry missing any required field is dropped silently: check all five first when a waiver
"does not work". An expired waiver stops exempting: the target is reported again, annotated with
the expiry date, at the rule's own severity. Active waivers are reported as
`summary.waiversActive`.

## Commands

| Command | Purpose |
|---|---|
| `prumo gate` | run the gate. `-c/--config`, `-r/--root`, `--json`, `--fail-on-warn`, `--preset starter\|strict`, `--mutation` / `--no-mutation`, `--mutation-base <ref>`, `--lang` |
| `prumo report` | print the last report without running anything, plus the trace matrix. `--json`, `--diff` |
| `prumo findings` | list promotable findings; `--promote <id>` writes a rule |
| `prumo status` | read-only next-step orientation. `--json`, `--as engineer\|stakeholder\|qa\|po` |
| `prumo orient` | read-only end-to-end flow (stage, plan, evidence) |
| `prumo setup` | checklist of agents (or `--target`): skills, MCP config, instructions block, hooks where the agent has them; scaffold `.prumo/` |
| `prumo harness sync` | refresh skills, hooks and `.prumo/prumo.json` after upgrading |
| `prumo sync` | `harness sync` then `gate` |
| `prumo interview` | brownfield intent and rule discovery. `--batch`, `--dry-run`, `--interactive` |
| `prumo start` | detect the stage (`greenfield`, `legacy`, `intent-needs-draft`, `draft-needs-freeze`, `steady`) and print the next step. `--init-intent <name> --owner <person>` writes a draft intent skeleton |
| `prumo allowed [path]` | whether an agent may write a path at this stage, and why; `--list` prints readable, writable and forbidden files. Exit 1 when blocked |
| `prumo contract draft` | write `_draft/<intent>.contract.ts`, one `@intent` constant per `R-` rule (deterministic) |
| `prumo contract freeze <name> --by <person>` | freeze a reviewed draft, ratified or existing contract and record its hash in `freeze-ledger.json` |
| `prumo contract extract` | run `config.extractors` and write observed contracts to `_observed/` |
| `prumo contract ratify <extractor> <name>` | copy an observed contract to `_ratified/` for review |
| `prumo trace` | print the intent ↔ contract ↔ oracle matrix. `--json` |
| `prumo gaps [path]` | what is missing for the module of a path (or the whole project): no intent covering it, a draft intent, rules without a frozen contract or an oracle, with interview questions seeded from the code. `--json` |
| `prumo badge` | SVG badge from the last report (see below) |
| `prumo doctor` | environment diagnosis: Node, config, versions, harness, plus the checks each active plugin contributes (e.g. `php`, `python3`) |
| `prumo check` / `budget` / `drift` / `coverage` / `fix` | AI context file audits |
| `prumo node-jest test-audit` | find redundant tests (needs `@ihelpee/prumo-plugin-node-jest`; `prumo test-audit` remains available as a root alias) |
| `prumo loopcheck` | detect degenerate repetition at the end of stdin |
| `prumo plugin [name]` | inspect available plugins |
| `prumo mcp` | stdio MCP server for IDEs. Agent tools: `prumo_start`, `prumo_gaps`, `prumo_allowed`, `prumo_trace`, `prumo_intent_init` (writes a draft, never approves), `prumo_contract_draft`, plus `prumo_gate`, `prumo_status`, `prumo_orient`, `prumo_interview` and the context and test-audit tools. No MCP tool approves an intent or freezes a contract |

### badge

`prumo badge` reads `.prumo/last-report.json` (it never runs the gate) and writes a
self-contained shields.io-style SVG. `--type oii` (default) shows `OII <floor(pct)>%`: green
≥ 80%, yellow ≥ 50%, red below. `--type gate` shows passing/failing; `--type invariants` shows the
finding count. A red badge still exits 0: the gate blocks, the badge reports. `-o -` or `--json`
prints the SVG to stdout.

### loopcheck

```bash
git diff --unified=0 | npx prumo loopcheck
npx prumo loopcheck --json --min-repeats 6 --max-period 40 < agent-output.txt
```

Flags a block of up to `--max-period` lines (default 24) repeated at least `--min-repeats` times
(default 4) **at the end** of the input. Repetition in the middle is not a loop. A cycle longer
than `--max-period` is invisible. Exit 1 on loop.

## Plugin timeouts

Each plugin process has a timeout. The value is taken from the first of these that is set: the project config (`plugins: [{ id, transport: 'jsonl-v2', timeoutMs }]`), the plugin's own `prumo.timeoutMs` in its `package.json`, or 120 000 ms. A plugin that exceeds it is a technical error for that plugin only; the other plugins still report.

## Intent to contract

Greenfield: `prumo start --init-intent billing --owner <person>` writes a draft intent. The owner
writes its rules as `<!-- prumo:rule id="R-billing-001" -->` followed by the statement, and
approves them. Then:

```bash
npx prumo contract draft                         # .prumo/contract/_draft/billing.contract.ts
npx prumo contract freeze billing --by <person>  # .prumo/contract/billing.contract.ts + ledger
npx prumo trace                                  # each rule with its contract and oracles
```

Every intent file starts with frontmatter (`id`, `owner`, `status`: `draft` | `approved` |
`deprecated`, and optionally `covers: <module>, <module>` for the code it specifies). A draft is generated, never authority: no file outside `_draft/` may import one
(the `no-draft-import` rule in this repo is an example). The freeze ledger,
`<contract>/freeze-ledger.json`, is committed: it records the file, its SHA-256, the approver, the
date and the `@intent` rule ids. Editing a frozen contract without a new freeze fails
`contract-frozen`.

Legacy: configure extractors and keep their output observed until someone ratifies it.

```ts
import { readFileSync } from 'node:fs';
import { dtoInterfaces } from '@ihelpee/prumo-plugin-node-nest';

export default defineConfig({
  extractors: [{
    id: 'nest-dto',
    extract: (root) => {
      const file = 'src/orders/dto/create-order.dto.ts';
      return dtoInterfaces(file, readFileSync(`${root}/${file}`, 'utf8'));
    },
  }],
});
```

`prumo contract extract` writes `_observed/nest-dto/…`; `prumo contract ratify nest-dto <name>`
copies one to `_ratified/`; `prumo contract freeze <name> --by <person>` freezes it.

Agents: `prumo harness sync` installs hooks for every detected editor.

- Cursor: a `preToolUse` hook asks `prumo allowed` before a write. Implementation waits for a
  frozen contract at the `greenfield`, `intent-needs-draft` and `draft-needs-freeze` stages; with
  `PRUMO_INTENT_GUARD_STRICT=1` the write is denied, otherwise it warns.
- Claude Code (`.claude/settings.json`): a `UserPromptSubmit` hook adds the output of
  `prumo gaps` to the agent's context, so it interviews the owner before changing a module
  nobody specified; a `PreToolUse` hook on Edit/Write asks `prumo allowed` and turns a held-back
  write into a question for the human (`ask`), or a denial with `PRUMO_WRITE_GUARD_STRICT=1`.
  `PRUMO_GAPS_CONTEXT=off` and `PRUMO_WRITE_GUARD=off` disable them.

Legacy code: `prumo gaps <path>` is where an interview starts. Its questions come from the
module's exported functions and from the optional fields of its observed contracts. Each answer
becomes a rule in a draft intent with `covers: <module>`; the owner approves it.

## CI

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0          # required: OII criterion 2 reads git history
- uses: actions/setup-node@v4
  with:
    node-version: 22
- run: npm ci
- run: npx prumo gate --json > prumo-report.json
```

Without full history, criterion 2 is wrong. A shallow clone under-reports independence. A
missing `.git` drops the criterion entirely and **over-reports** it.

The growth rule compares against the ledger. CI runs start without one, so only `minOii`
applies unless you persist `.prumo/ledger.json` between runs (commit it, or restore it from the
last `main` run).

To publish the OII badge, run `prumo gate` then `prumo badge` in a job on `main`, and commit
`.prumo/badge.svg` with `[skip ci]` in the message so the commit does not retrigger the workflow.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `oracle-isolation` skips: no test files found | `testFilePatterns` matches nothing | match your layout, e.g. add `oracle/**/*.ts` |
| `no-test-files-matched` warning | `testFilePatterns` matches nothing, but the oracle directories hold code | match your layout; the check no longer passes silently |
| `OII 0% 0/0` and budget FAIL | no test files | as above; 0/0 is zero evidence, not 100% |
| OII suddenly higher | git missing or history truncated | `fetch-depth: 0`; check `passedProvenance` in the JSON |
| `provenance-gap` warnings | tests added in the same commit as `src/` | commit oracles separately from implementation, or use `oii.profile: 'fast'` while adopting |
| `oracle-opaque-import` warning | `import(x)` / `require(x)` with a computed specifier | use a literal specifier so isolation can be checked |
| `invariants` passes with `routes=0` | `layout.src` does not cover your controllers | fix `layout.src`; rules over an empty set prove nothing |
| `erroredRules > 0` | a rule threw | read the error in `--json` output and fix the rule |
| `invariants` skips: no rules | no plugins and no rule files | add a plugin or a `.ts` under `layout.invariants` |
| waiver ignored | missing field, or `target` not an exact match | check all five fields and the exact target |
| `harness-stale` warning | harness older than the installed bin | `npx prumo harness sync` |
| `report`/`findings`/`badge` exit 1 | no `.prumo/last-report.json` | run `prumo gate` with the same `--root` |

## Upgrading

- **0.13** moved domain features out of the core. If you use the spec interviewer / Prompt Zero
  flow, add `@ihelpee/prumo-plugin-speckit`. Pipeline governance is
  `@ihelpee/prumo-plugin-pipeline`, and GitHub Projects access is
  `@ihelpee/prumo-plugin-github-project`. Plugins declare a v2 manifest (`prumo`, `contributes`);
  see [architecture.md](architecture.md#plugin-manifest-v2).
- **Renamed plugins:** `@ihelpee/prumo-plugin-nest` → `@ihelpee/prumo-plugin-node-nest`,
  `@ihelpee/prumo-plugin-jest` → `@ihelpee/prumo-plugin-node-jest`.
- All `@ihelpee/prumo*` packages are versioned in lockstep: install the same version of the core
  and every plugin. After upgrading, run `npx prumo harness sync`.
- Per-release details: [Core package changelog](../CHANGELOG.md).
