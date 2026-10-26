# Architecture

For contributors and plugin authors. The core is an engine and assumes nothing about the user's
domain. Plugins bring the domain: frameworks, languages, platforms and workflows.

## Packages

All packages live in `packages/` and are versioned in lockstep.

| Package | Directory | Role |
|---|---|---|
| `@ihelpee/prumo` | `prumo` | gate, OII, rule DSL, waivers, ledger, plugin host, CLI, MCP server, core skills |
| `@ihelpee/prumo-plugin-node-nest` | `plugin-node-nest` | NestJS invariants (guards, DTO validation, module exports, feature boundaries) |
| `@ihelpee/prumo-plugin-node-next` | `plugin-node-next` | Next.js invariants |
| `@ihelpee/prumo-plugin-node-jest` | `plugin-node-jest` | Jest test-audit and test patterns |
| `@ihelpee/prumo-plugin-speckit` | `plugin-speckit` | spec interviewer, Prompt Zero catalog, phase status (experimental) |
| `@ihelpee/prumo-plugin-github-project` | `plugin-github-project` | thin GitHub Projects/Issues client (experimental) |
| `@ihelpee/prumo-plugin-pipeline` | `plugin-pipeline` | session timeline and preflight gates for GitHub mutations (experimental) |
| `@ihelpee/prumo-i18n-pt-br` | `i18n-pt-br` | pt-BR message catalog |

Each plugin's design spec, where one exists, is `DESIGN.md` in its package directory.

## Governance ownership

The monorepo root `.prumo/` owns rules that apply across packages: the constitution, roadmap,
workspace invariants, plugin-wide verification policy, and repository-level exceptions. The Prumo
core package owns its product intent and frozen contracts in `packages/core/.prumo/`, with its
own `prumo.config.ts`. CI runs both gates: the root gate aggregates workspace rules and intents,
and the core gate runs from `packages/core` so frozen-contract ledger paths remain relative to
their owner. Framework plugins keep their own `.prumo/` governance inside their package.

**Scope.** The gate, the OII and the framework plugins are the product. The workflow plugins
(speckit, pipeline, github-project) are experimental: new work there waits until the core
checks are validated on more real projects (see [verification.md](verification.md#external-validation)).

## Core layout (`packages/core/src`)

| Directory | Contents |
|---|---|
| `cli/` | commands, gate runner, checks (`checks.ts`), OII and budget (`oii.ts`), MCP server, badge |
| `core/` | config types and presets, ledger, waivers, i18n, plugin host and manifest, status/orientation, mutation engine |
| `invariants/` | AST extraction (`ast.ts`), rule DSL and engine, project rule loading, fabric laws |
| `context/` | AI context file audits (`check`, `budget`, `drift`, `coverage`, `fix`) |
| `skills/` | core skill definitions, `setup`, interview engine |
| `harness/` | harness sync, MCP config, Cursor hook installation, harness policy |

Rules of the core:

- The gate is deterministic: no model calls on the gate path.
- User-facing strings live in the English catalog (`core/i18n.ts`); translations come from i18n
  packages. Code, comments and identifiers are in English.
- The core ships nine domain-agnostic skills: `site-foreman`, `contract-author`, `oracle-author`,
  `reference-implementer`, `instantiator`, `invariant-auditor`, `gate-runner`, `test-pruner`,
  `flow-orchestrator`. Plugins add their own.
- `prumo interview` starts with an empty catalog; plugins register questions.

## Output language

Output is English unless the language pack for the requested locale is installed
(`@ihelpee/prumo-i18n-pt-br` for `pt-BR`). `resolveLocale` falls back to `en` otherwise, so a
plugin never prints a language the core cannot.

- **Core** strings live in the English catalog (`core/i18n.ts`); the language pack overrides
  them key by key, and any key it lacks falls back to English. Core rules take their translated
  message from the pack's `rules` section, by rule id.
- **Plugins** ship their own translations with `defineMessages` from `@ihelpee/prumo/core`.
  English is required for every key, and other locales are optional per key:

  ```ts
  // src/messages.ts
  export const messages = defineMessages({
    en: { 'my-rule': 'route without a timeout' },
    'pt-BR': { 'my-rule': 'rota sem timeout' },
  });

  // src/rules.ts
  rule('my-rule').message(messages.text('my-rule'))         // finding message, translated
  messages.t(locale, 'someError', { path })                  // CLI output and errors
  ```

  Plugin CLI commands resolve the locale with `resolveLocale({ cliFlag: opts.lang })`.
- **Finding targets are always English.** Waivers match targets exactly, so a target must not
  change with the output language. Only `message` is translated.

## What a plugin can contribute

The core knows no framework, language or toolchain. Everything framework-aware
comes from a plugin object listed in `plugins:` (`PrumoPlugin`, `invariants/rules.ts`):

| Field | What it adds |
|---|---|
| `rules`, `extractors` | invariant rules and the AST data they read |
| `layout`, `testFilePatterns`, `srcFileExtensions` | defaults merged when the project does not set them |
| `testAudit` | the redundant-test audit engine (`prumo test-audit`, the `test-redundancy` check). The core ships none |
| `mutationOperators` | framework-aware mutation operators for OII criterion 3 (e.g. `guard-removal` from `plugin-node-nest`) |
| `doctorChecks` | `prumo doctor` rows for the toolchain the plugin needs (e.g. `php`, `python3`) |

Packages also declare a manifest (below) for CLI commands, status providers and harness skills.

## Gate context

Rules and plugins receive a `GateContext` (`core/gate-context.ts`): `root`, `config`, `locale`,
`ledger`, `waivers`, `filesystem` (`exists`, `readFile`, `listFiles`), `env`, and
`extensions.<pluginId>` for plugin-specific data. AST data (`routes`, `dtos`, `imports`, …) is
populated only when a rule asks for it through `needsData`.

## Plugin manifest v2

A plugin declares a manifest in its `package.json`:

```json
{
  "name": "@ihelpee/prumo-plugin-example",
  "prumo": ">=0.13.0",
  "contributes": {
    "activate": "./dist/index.js",
    "rules": "./dist/rules.js",
    "cliCommands": "./dist/cli.js",
    "statusProviders": "./dist/status.js",
    "harness": { "skills": "./dist/skills.js" }
  }
}
```

| Field | Required | Meaning |
|---|---|---|
| `name` | yes | npm package name |
| `prumo` | yes | semver range of the supported core; checked at load |
| `contributes.rules` | no | invariant rules (or a factory) |
| `contributes.cliCommands` | no | commands attached to the `prumo` CLI |
| `contributes.statusProviders` | no | data added to `prumo status` |
| `contributes.harness.skills` | no | skills merged by `prumo harness sync` |
| `contracts`, `setup.scaffolds`, `gate.phases`, `dependencies` | no | contracts, setup templates, custom gate phases, required plugins |

Load sequence: plugins listed in `config.plugins` are resolved from `node_modules` or the
workspace, validated with `validateManifestV2`, then activated. Activation calls
`registerStatusProvider`, `registerCliContribution`, `registerHarnessSkills` and
`registerInterviewCatalog` (`core/plugin-host.ts`). During `prumo gate`, rules from all active
plugins run against the same context.

## Tests

The repository tests itself with **oracles**, not unit tests: scripts in `oracle/` that run the
built bin (`packages/*/dist`) as a subprocess against fixtures in `packages/core/fixtures/`, and never import
`packages/*/src`. `scripts/run-oracles.mjs` runs them by scope: `core` tests only the core;
the active plugin scopes are `nest`, `jest`, `next`, `speckit`, `pipeline` and `github-project`,
and `plugins` covers the cross-plugin contract. The Go, PHP and Python language-plugin scopes are
currently frozen and omitted until those plugins are revived. In CI, `core` runs in the build job
and every other active scope in its own job. The repository's own gate (`pnpm gate:dist`)
enforces this.

### Mutation engine

`core/mutation/` mutates only lines changed in the PR delta (operator set P0+P1). It has a
global budget (`timeoutMs`, `maxMutants`, `maxPerFile`) and fails open to `inconclusive` when the
budget runs out. The full design is in
[mutation.md](mutation.md).
