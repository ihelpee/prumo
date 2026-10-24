# Specification — Load-Test Engine on the PR Delta (Surgical Mutation)

- **Task:** t_fa9254c4 · **Author:** AnalisIA (Principal Product Owner)
- **Status:** Specification for implementation (suggested assignee: @axia)
- **Vocabulary:** In Prumo, mutation testing is a **load test**: the engine induces a
  deviation in the code (a mutant) and checks whether a test notices it. A *killed* mutant
  is a deviation a test caught; a *surviving* mutant means the suite is **complacent** (a
  false green). The older glossary (archived, git bdb18a9:docs/GLOSSARIO.md) and
  Constitution Art. V define the original metaphor.
- **Scope:** Turn OII criterion 3 from a disabled `v2` placeholder into an ultralight
  load-test engine limited to the delta of files changed by the PR.
- **Explicit non-scope:** replacing Stryker as a full-repository mutation tool. This engine
  is a **PR alignment jig**, not a demolition.

---

## 1. Context and problem

The OII (Oracle Independence Index) is today the intersection of two active criteria
(`packages/prumo/src/cli/oii.ts:71-76`):

| # | Criterion | Source | Cheatable? | Status |
|---|---|---|---|---|
| 1 | test does not import `src/**` | AST | no, not without breaking the test | active |
| 2 | committed without touching `src/**` in the same commit | git | yes — a marker line | active |
| 3 | kills an induced mutant (load test) | AST | — | **v2, off** |

Current state verified in the code:

- `OiiInput.mutationPassing: Set<string> | null` (`oii.ts:38-40`) — only populated if the
  `runGate` caller injects it programmatically (`runner.ts:68-69`). There is no config
  field, no CLI flag and no engine (MANUAL (archived, git bdb18a9:docs/MANUAL.md)).
- `evaluateBudget` already receives `mutationScore`, but the runner always passes `null`
  (`runner.ts:230`) — the parameter exists and is dead.
- The report prints `passedMutation=desligado (v2)` (`runner.ts:209`).

**The business problem:** without criterion 3, the "plaster model / plastic spirit level"
test (a green suite that never notices when the wall is bent) cannot be detected (ABOUT
(archived, git bdb18a9:docs/ABOUT.md)). That is exactly the gap that lets shallow tests
grow as pure cost, which the budget tries to prevent.

**Why not Stryker:** Stryker shakes the whole repository, with instrumentation, a sandbox
and a worker pool — minutes per run. A PR gate that takes minutes changes the team's
behavior for the worse: the path of least resistance becomes skipping the gate. The
usefulness of criterion 3 is **proportional to its latency**. If it does not run in
seconds, it is not a gate; it is a report — and nobody reads reports.

---

## 2. Product decisions (binding on the implementation)

These are the PO's decisions. The implementation does not reopen them; disagreement goes
back as a comment on the card.

### D1 — Scope: criterion 3 is a **veto over the delta**, not a global measurement

Mutation on a delta cannot, by construction, produce an honest global OII: files outside
the delta were never mutated. If the engine marked "not evaluated" as "not killed", the
intersection in `oii.ts:76` **would push every pre-existing test out of the numerator and
the OII would collapse on every PR** — a false negative that destroys trust in the metric.

Rule:

```
criterion3(f) =
  ├─ f is NOT in the PR's test delta  → inherits the previous verdict (ledger) or PASS
  └─ f IS in the PR's test delta      → PASS if it killed ≥1 induced mutant in the src delta
```

- A test **added or changed** by the PR must prove it kills an induced mutant in the load
  test. It is the right question at the right moment: "does the test you just wrote have
  real structural support? Does it notice if the wall is forced?"
- A test outside the delta **inherits** the verdict persisted in the ledger; with no
  history it inherits PASS (informative, never retroactively punitive).
- Consequence: the global OII stays comparable between runs (the delta can only keep or
  improve the numerator, never degrade untouched files).

### D2 — OII semantics stay a **binary intersection**; mutation weight = full veto on the delta

The alternative considered and **rejected**: a weighted average
`OII = w1·isolation + w2·provenance + w3·mutation`. Reason: an average lets decorative
plastic tests into the score, diluted by the good ones, and the product invariant is "a
plaster model does not get into the plumb line". The binary veto on the delta keeps the
budget rule ("+tests without an OII increase = pure cost") and adds the load test and
structural rigor that were missing.

The `mutationScore` (killed mutants / induced mutants in the delta) becomes a **reported
metric** — in the `BudgetVerdict` (the field already exists, `types.ts:102`), in the report
and in the ledger — but **it does not enter the OII formula**. In the `strict` preset, a
suite that is complacent with every induced mutant in the delta is a direct FAIL of the
`mutation` check (D8).

### D3 — Mutation source: only `src/**` files changed in the PR delta

If the PR does not touch `src/**` (a tests-only PR), there is nothing to mutate → the
check is `skip` with an explicit reason, and criterion 3 **does not punish** (an
inapplicable veto ≠ a rejection).

### D4 — The working tree is untouchable

Mutation is applied **in memory / in an overlay**, never on the working disk. A gate that
dirties the developer's tree is rejected by users on day 1.

### D5 — The time budget is a first-class requirement, not an optimization

A configurable global deadline (default **5000 ms** of engine wall-clock, excluding the
test baseline the gate already runs). When the budget runs out → the check degrades to
`inconclusive` (status `skip` with a reason + warn), **fail-open**. An engine that fails CI
for being slow will be switched off by the team; fail-open preserves adoption.

---

## 3. Mutation operators (v1 menu)

Operator inclusion criteria: (a) a plausible mutation (changes observable behavior), (b)
cheap AST detection, (c) a business signal. The engine does **not** chase parity with
Stryker; it chases the structural construction defects the product fights.

| ID | Name | Mutation | AST target | Priority |
|---|---|---|---|---|
| `cond-inversion` | Condition inversion | `===`↔`!==`, `==`↔`!=`, `<`↔`>=`, `>`↔`<=`, `&&`↔`\|\|` | BinaryExpression | P0 |
| `bool-negation` | Boolean negation | `true`→`false`, `false`→`true` | BooleanLiteral | P0 |
| `guard-removal` | **Authorization removal** | removes the `@UseGuards(...)`/`@Roles(...)`/custom guard decorator from a mutation handler (POST/PUT/PATCH/DELETE) | Decorator on MethodDeclaration | P0 |
| `boundary-shift` | Boundary shift | `<`→`<=`, `>`→`>=`, `>=`→`>`, `<=`→`<` | BinaryExpression | P1 |
| `negation` | Expression negation | `if (x)`→`if (!x)`; flips the return of an early-return guard (`return true`↔`return false` in predicate functions) | IfStatement / ReturnStatement | P1 |
| `arith-flip` | Arithmetic swap | `+`↔`-`, `*`↔`/` | BinaryExpression | P2 |
| `block-removal` | Statement removal | removes a whole statement from a block | Statement | P2 (cap: 1/file) |

Business-rule notes:

- **Where operators live:** the core ships only the framework-agnostic operators. Framework-aware
  ones come from plugins through `PrumoPlugin.mutationOperators`; `guard-removal` is contributed by
  `plugin-node-nest`.
- **`guard-removal` is Prumo's signature operator.** It is the dynamic counterpart of the
  static invariant `mutation-routes-guarded` (which catches a *missing* guard; the induced
  mutant catches a *present guard that no test exercises*). A PR that removes a
  `@UseGuards` while the suite stays green is exactly the incident the product promises
  to prevent. This operator can never be turned off while the Nest plugin is active.
- Operators apply **only to nodes whose position falls inside the changed line ranges** of
  the hunk (see §4) — "surgical" means a node in the delta, not a file in the delta.
- Nodes in `.spec`/`.test` files are never mutated (mutation is only in `src/**`).
- The menu is configurable (§6.3); `cond-inversion` and `bool-negation` are always on.

---

## 4. Git diff filtering algorithm (delta → AST nodes)

```
INPUT:  root, config, baseRef (default: merge-base with the target branch)
OUTPUT: MutationPlan { mutants: Mutant[], deltaTests: string[] }

1. BASE      base = git merge-base HEAD <target>   (fallback: HEAD~1; no git → skip)
2. DIFF      changed = git diff --unified=0 --no-renames <base>...HEAD
             → per file: a list of [start,end] of lines added on the new side
             (parse of the hunk header `@@ -a,b +c,d @@`; --unified=0 gives the exact range)
3. FILTER    srcChanged  = changed ∩ config.layout.src      ← source of induced mutants
             testChanged = changed ∩ testFilePatterns        ← deltaTests (scope of the D1 veto)
4. AST       in-memory ts-morph Project (pattern already used in invariants/test-audit.ts)
             parse only srcChanged
5. CANDIDATES  for each file, a SyntaxKind walker per operator (§3):
                eligible node ⟺ node.start.line and node.end.line ⊆ some hunk range
             apply the per-file cap (default 10) and sort by operator priority
6. PLAN      apply the global cap (default 30) in the order: guard-removal → cond/bool → P1 → P2
             (cap reached → flag `capped=true` in the report; never FAIL because of the cap)
```

Points of attention (requirements, not suggestions):

- **Renames:** `--no-renames` on purpose — a rename shows up as delete+add; the "add" enters
  as delta and can be mutated. Document it in the code.
- **Files deleted in the delta:** ignored as a source (there is no content to mutate).
- **Dirty worktree / run outside a PR:** if no target branch can be resolved, base =
  `HEAD~1` and the check reports the base it used. Never assume a hardcoded `main`.
- **No git repository:** check `skip` (same behavior as criterion 2).

### 4.1 Test selection per induced mutant (the decision that sets the latency)

Running the whole suite per induced mutant is the mistake that makes Stryker cost minutes.
Rule:

```
tests(induced mutant m in file f) =
  1. if there is a coverage cache from the last baseline (coverage-final.json):
       the test files whose coverage maps include f
  2. otherwise, static fallback: the transitive import graph (1 level) between testFiles and f
     — for oracles that do not import src (criterion 1), use the layout's
     feature-to-oracle mapping (same boundary heuristic as no-cross-feature)
  3. last resort: the whole suite, but only when |selected tests| < the cost threshold
```

The induced mutant is **killed** if any selected test fails with the mutant applied. File
`f` is marked `mutationPassing` if ≥1 induced mutant of `f` was killed.

---

## 5. Performance budget (< 5 s)

Budget per phase (wall-clock, a standard CI machine, a medium-sized project):

| Phase | Budget | Note |
|---|---|---|
| git diff + merge-base | ≤ 100 ms | 2 `git` subprocesses, already standard in `core/git.ts` |
| AST parse (srcChanged only) | ≤ 300 ms | ts-morph without a full `createProgram`; parse per file |
| induced mutant generation | ≤ 50 ms | textual substitution at the node position |
| execution per induced mutant | ≤ 150 ms × N | the dominant cost; N ≤ 30 by the cap |
| **global deadline** | **5000 ms** | AbortController; exceeded → `inconclusive` (D5) |

Execution requirements:

- **M1.** The suite baseline runs **once** (the gate already runs tests; the engine reuses
  the result and the coverage cache, it does not re-run).
- **M2.** Execution per induced mutant runs **only the selected tests** (§4.1), with the
  runner warm. Recommended path: **programmatic vitest with a Vite plugin that resolves `f`
  to the mutated source in memory** (meets D4 with no copies and no cold start per mutant).
  Accepted alternative: jest with a temporary project in a shadow-copy directory. The final
  choice is Axia's, validated in a spike (§9 R1), but D4 and the deadline are not
  negotiable.
- **M3.** The induced-mutant cap (30) and the per-file cap (10) are configurable, and the
  report states when the cap truncated the plan (`capped=true`).
- **M4.** A timeout per individual induced mutant (default 2000 ms): a mutant that does not
  finish counts as **not killed** and is reported as `timeout` (a suspected infinite loop
  induced by the mutation — common in a `while` with an inverted condition).
- **M5.** A performance regression benchmark is required in Prumo's CI (§8 CA-7): a fixture
  with a 5-file PR / ~25 induced mutants must finish the engine at p95 < 5 s.

Why 5 s is reachable where Stryker fails: Stryker pays for (a) instrumenting the whole
project, (b) a sandbox per mutant, (c) the full suite per mutant, (d) hundreds to thousands
of mutants. This engine removes all four: a line delta (not a project), an in-memory
overlay (not a sandbox), selected tests (not the suite), ≤ 30 induced mutants (not
thousands).

---

## 6. Integration into the score and the gate

### 6.1 Data contract

```ts
// core/types.ts (additions)
export interface MutationResult {
  status: 'pass' | 'fail' | 'skip' | 'inconclusive';
  baseRef: string;                 // base used (merge-base or HEAD~1)
  mutantsGenerated: number;
  mutantsKilled: number;
  mutationScore: number | null;    // killed/generated; null if generated=0
  killedByFile: Map<string, string>; // testFile -> mutantId (who proved structural support)
  capped: boolean;
  elapsedMs: number;
  skipReason?: string;
}
```

- `runGate` now **produces** `mutationPassing: Set<string>` (a contract that already exists
  in `OiiInput`) from `killedByFile`, following D1: the set holds the delta tests that
  killed + the tests outside the delta that inherit a verdict.
- `computeOii` **does not change its formula** — the existing intersection (`oii.ts:76`)
  already implements the veto, as long as the set follows D1. This is a requirement: the
  change lives in producing the set, not in the index arithmetic.
- `evaluateBudget` gets the real `mutationScore` (replacing the fixed `null` in
  `runner.ts:230`) — the field already exists in `BudgetVerdict`.
- Ledger: persist `mutationScore` and `killedByFile` per run, for the D1 inheritance
  mechanism and the OII delta between runs.

### 6.2 Relative weight between criteria (answer to question 4 of the card)

| Criterion | Role in the OII | Effective weight |
|---|---|---|
| Isolation (1) | required boundary | full veto, always |
| Provenance (2) | git marker line | veto when git is present (unchanged) |
| Mutation (3) | evidence marker line **on the delta** | full veto **over the test delta**; informative outside it |

Executive rationale: the three criteria measure orthogonal things (structure, history,
behavior) and are not convertible into each other — weighting them would add incomparable
units. The right weight for mutation is binary where it has evidence (the delta) and zero
where it has none (the rest). Any future proposal of a fractional weight must first solve
ledger verdict inheritance; until then, the question is closed.

### 6.3 Configuration and CLI surface

```ts
// prumo.config.ts
mutation?: {
  enabled: boolean;          // default: on in CI when testCommand is set (CI env var), off locally
  base?: string;             // PR target ref; default: auto-detect (merge-base)
  timeoutMs?: number;        // default 5000
  maxMutants?: number;       // default 30
  maxPerFile?: number;       // default 10
  operators?: string[];      // default: P0+P1 from §3
}
```

- CLI: `--mutation` / `--no-mutation` override the config; `--mutation-base <ref>`.
- Presets: `strict` turns `mutation` into a direct FAIL check when the delta has induced
  mutants and none was killed.
- Report: the line `passedMutation=desligado (v2)` (`runner.ts:209`) now reflects the real
  state: `mutation 87% (13/15 induced mutants, base=origin/main, 2.1s)` or
  `mutation inconclusive (5s budget exceeded)` or `mutation skip (PR does not touch src/)`.
- i18n: new messages go into the English catalog with their pt-BR pair in
  `packages/i18n-pt-br` (`oracle/check-i18n.ts` validates).

---

## 7. Edge cases (required behavior)

| Scenario | Behavior |
|---|---|
| Repo without git | check `skip`; criterion 3 inapplicable (same as criterion 2) |
| Tests-only PR (delta without `src/**`) | `skip` with the reason `PR does not touch src/`; the veto does not apply (D3) |
| `src/**` delta with no eligible operator | `pass` with `mutantsGenerated=0`; never FAIL for lack of an induced mutant |
| Deadline exceeded | `inconclusive` (skip + warn); the OII runs with criteria 1–2; **never FAIL because of time** (D5) |
| Induced mutant with an individual timeout | counted as not killed, reported as `timeout` (M4) |
| Test baseline fails | the engine does not run; the gate already fails on the tests check (mutation runs after green) |
| File deleted in the delta | ignored as a source of induced mutants |
| `mutation.enabled=true` without a configured `testCommand` | `skip` with an explicit reason (no tests means no kill) |
| Global cap reached | `capped=true` in the report; prioritized plan (§4 step 6) |

---

## 8. Acceptance criteria (BDD)

**CA-1 — Veto on the delta (happy path)**
```
Given a PR that changes src/orders/fees.ts (the line with `if (amount >= FREE_LIMIT)`)
 And the delta includes oracle/fees.contract.spec.ts
 And the suite is green on the baseline
When the gate runs with mutation enabled
 And the test does not tell `>=` from `>` (every test tolerates the induced mutant)
Then the `mutation` check has status fail
 And the test does NOT enter `passedMutation`
 And the OII removes that test from the numerator
```

**CA-2 — A test with structural support (load test) passes**
```
Given the same PR
 And the oracle covers amount = FREE_LIMIT - 1 and FREE_LIMIT
When the gate runs
Then at least 1 induced mutant of fees.ts is killed
 And fees.contract.spec.ts enters `passedMutation`
 And the `mutation` check has status pass with a reported mutationScore
```

**CA-3 — A PR without src is not punished**
```
Given a PR that only adds files under oracle/
When the gate runs
Then the `mutation` check has status skip with the reason "PR does not touch src/"
 And the OII is computed with criteria 1–2 without a penalty
```

**CA-4 — Inheritance outside the delta (D1)**
```
Given a repository with 100 independent tests in the ledger
 And a PR that changes 1 src file and 1 oracle
When the gate runs mutation only on the delta
Then the 99 tests outside the delta stay in the OII numerator
 And the global OII does not drop because of mutation
```

**CA-5 — Fail-open on the budget (D5)**
```
Given an artificial delta whose induced mutants make test selection cost > timeoutMs
When the gate runs
Then the `mutation` check has status inconclusive (skip + warn)
 And the gate verdict is NOT decided by the mutation check
```

**CA-6 — Authorization operator**
```
Given a PR that removes @UseGuards(AuthGuard) from a POST handler in src/reports/reports.controller.ts
 And the suite stays green
When the gate runs
Then the `guard-removal` induced mutant is generated and survives
 And the `mutation` check finding references the handler and the `guard-removal` operator
```

**CA-7 — Performance benchmark (Prumo's CI)**
```
Given the benchmark fixture (5 changed src files, ~25 induced mutants, an oracle suite)
When the engine runs on a standard CI runner, 20 runs
Then the p95 of the engine's elapsedMs < 5000 ms
 And the working tree is intact after the run (git status --porcelain is empty)
```

**CA-8 — Contract and non-regression**
```
Given mutation disabled
When the gate runs
Then the behavior is byte-for-byte identical to before (passedMutation=null, "off (v2)"
 replaced by "off" — the text is the only change allowed)
 And the oracle fixtures (oracle/fixture-expectations.ts) keep passing
```

**CA-9 — i18n**
```
Given the pt-BR and en locales
When the mutation check reports any status
Then the messages exist in both languages and oracle/check-i18n.ts passes
```

---

## 9. Risks and pre-implementation spikes

- **R1 (high) — A warm runner per induced mutant.** Reaching < 5 s depends on running the
  selected tests without a cold start per mutant. A spike is required before the
  production code: programmatic vitest + an in-memory resolution plugin vs. a jest shadow
  copy. Spike criterion: 15 induced mutants × 3 test files in < 3 s on a Nest fixture
  project. If no path meets it, the product falls back to a "post-merge asynchronous
  engine" and the PO revisits the positioning — a slow gate does not ship.
- **R2 (medium) — False positives from equivalent mutants** (a mutation with no observable
  effect, e.g. `+0`). Mitigation: v1 only has high-signal syntactic operators;
  `block-removal` (the noisiest) enters at P2 with a cap of 1/file.
- **R3 (medium) — Coverage-based test selection needs a baseline with coverage.** If the
  project does not run coverage, the static fallback (§4.1 step 2) may under-select for
  oracles that do not import src. Mitigation: when static selection returns nothing, use
  the whole suite if it fits the remaining budget; otherwise `inconclusive`.
- **R4 (low) — Mutation induces an infinite loop.** Covered by M4 (a timeout per induced
  mutant).

## 10. Out of scope (v1)

- Mutating `node_modules`, JSON, CSS or non-TS files.
- String/regex/template-literal induced mutants.
- Incremental mutation with a cache between PRs (a v2.1 candidate — the ledger is already
  the place).
- Support for runners other than jest/vitest.
- Replacing Stryker for a full repository audit.

---

## 11. Suggested Kanban breakdown

1. **Spike R1** (Axia, no production PR): a proof of concept of the warm runner with
   measured numbers. *Gate: decides the execution architecture.*
2. **Core** (Axia, depends on the spike): `core/mutation/` — diff parser, induced-mutant
   plan, P0 operators, `MutationResult`, the `runGate`→`mutationPassing` integration without
   touching the OII formula. Covers CA-1..4, CA-8.
3. **`guard-removal` operator + Nest** (Axia, depends on the core): CA-6.
4. **CLI/config/presets/ledger/i18n** (Axia, depends on the core): §6.3, CA-9.
5. **CI benchmark + fixture** (Axia, depends on the core): CA-7, CA-5.
6. **Acceptance** (Cecília): the CA-1..9 suite against the build, including the CA-8
   byte-for-byte non-regression with mutation off.
