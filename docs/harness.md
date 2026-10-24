# Agent harness

Prumo skills are defined by what the agent **must not read**. The gate mechanically enforces one
barrier: tests must not import `src/**`. Every other read barrier (`forbiddenReads` in
`packages/prumo/src/skills/definitions.ts`) is declared, listed by `prumo_skills`, and enforced
only if you install a harness.

Without enforcement, greenfield work is fragile; guardrail and brownfield use still get value
from the gate.

## Levels

| Level | Effort | Effect |
|---|---|---|
| L0 — honor system | none | skills and rules in context; the gate catches mirrors after the fact |
| L1 — process | low | separate commits (contract → oracle → src); CI with `fetch-depth: 0` |
| L2 — convention | medium | switch to the `oracle-author` skill before writing tests; don't mention `src/` files in that turn |
| L3 — read guard | medium | a hook denies reads matching the active skill's `forbiddenReads` |
| L3 — intent guard | medium | a hook warns on (or blocks) writes to `src/**` when no intent file exists |

## What `prumo setup` installs

When a `.cursor/` directory exists, `prumo setup` copies and merges:

| File | Role |
|---|---|
| `.cursor/rules/prumo-foreman.mdc` | always-on rule: orient with `prumo status`, use the right skill, run the gate |
| `.cursor/hooks/prumo-forbidden-read.js` | read guard for the skill in `PRUMO_ACTIVE_SKILL` |
| `.cursor/hooks/prumo-intent-guard.js` | `preToolUse` guard for writes to `src/**` without intent |
| `.cursor/hooks.json` | registers both hooks |

Both hooks are offline (no model) and warn-only by default:

| Variable | Effect |
|---|---|
| `PRUMO_ACTIVE_SKILL=oracle-author` | read guard blocks `src/**` |
| `PRUMO_ACTIVE_SKILL=contract-author` | read guard blocks `src/**` and `oracle/**` |
| `PRUMO_FORBIDDEN_STRICT=1` | read guard exits 1 on a violation instead of warning |
| `PRUMO_INTENT_GUARD_STRICT=1` | intent guard blocks the write instead of warning |
| `PRUMO_INTENT_GUARD=off` | disables the intent guard |

For other IDEs, wire the same scripts into their hook API, or read the skill list from the MCP
tool `prumo_skills`.

## Keeping the harness current

| Command | When | Edits `prumo.config.ts` | Writes `.prumo/prumo.json` |
|---|---|---|---|
| `prumo setup` | first install | yes (scaffold) | yes |
| `prumo harness sync` | after every `@ihelpee/prumo` upgrade | no | yes (version stamp, MCP pin) |
| `prumo gate` | before merge, every agent turn | no | no |
| `prumo sync` | sync, then gate | no | yes |

The gate reports `harness-stale` (warn) when the harness meta is missing or older than the
running bin. Set `prumoVersion` in the config to make a version mismatch an error. `prumo doctor`
prints the declared, running and stamped versions.

## What the harness will not do

- Write intent, contracts or oracles.
- Replace code review or production monitoring.
- Make OII criterion 2 a security boundary: git provenance is a process signal.
