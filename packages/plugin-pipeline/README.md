# @ihelpee/prumo-plugin-pipeline

Deterministic governance for agent work tracked in GitHub issues: a phase timeline per issue
and a **preflight** check that authorizes (or blocks) each mutation before it happens.

```ts
// prumo.config.ts
import pipeline from '@ihelpee/prumo-plugin-pipeline';
export default defineConfig({ plugins: [pipeline] });
```

## Commands

```bash
prumo pipeline session   --issue owner/repo#N [--json]                     # current phase, checkpoints, sub-issues
prumo pipeline preflight --issue owner/repo#N --action <action> [--json]   # exit 0 = allowed, 1 = blocked
```

A blocked preflight returns `blockedReason`, `lawfulNext` (the next legitimate step) and
`currentPhase`.

| Action | Typical requirement |
|---|---|
| `issue-edit-body` | an approval status needs the `confirmed` phase first |
| `issue-comment` | allowed for progress and checkpoints |
| `issue-close` | sub-issues done and a final board status |
| `project-move-status` | phases listed in `boardRules.requiredPhases` for the target status |
| `issue-add-sub-issue` | kind listed in `issueKinds` |
| `issue-edit-assignees` | valid session state |
| `decision-file-write` | path inside `decisionsGlob` |

## Timeline

Phases are recorded as machine-readable HTML comments on the issue:

```html
<!-- pipeline:phase:v1 {"phase":"confirmed","author":"reviewer","at":"2026-10-23T10:00:00Z"} -->
```

Phases, in order: `briefing` → `draft` → `verified` → `confirmed` → `persisted`. Legacy
`<!-- odin-checkpoint v1 {...} -->` markers are read but never written.

## Configuration: `.prumo/pipeline.yaml`

```yaml
projectId: 13                       # GitHub Projects v2 number
statusMap:
  APPROVER: approver
  DONE: done
issueKinds: [epic, task, sub-issue]
boardRules:
  requiredPhases:
    APPROVER: [briefing, draft, verified, confirmed]
    DONE: [briefing, draft, verified, confirmed, persisted]
decisionsGlob: changelog/decisions/**
externalGates: []
timelineMarkers:
  prefix: 'pipeline:'
```

## Agent skill

The plugin adds the `pipeline/foreman` skill (synced by `prumo harness sync`): load the session
at the start (`prumo pipeline session`) and run `prumo pipeline preflight` before every
mutation. It works alongside the repository's own skills.

## Offline mode

Set `PIPELINE_FIXTURE=<path/to/fixture.json>` to read issue data from a file instead of the
network (used by CI and `oracle/check-pipeline.ts`). Without a fixture, `GH_TOKEN` or
`GITHUB_TOKEN` is required; the command fails with an explicit message if neither is set.

GitHub access goes through `@ihelpee/prumo-plugin-github-project`.
