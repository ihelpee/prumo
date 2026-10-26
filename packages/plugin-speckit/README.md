# @ihelpee/prumo-plugin-speckit

Experimental planning and interview plugin for Prumo. Speckit helps an agent turn an unclear
request into a human-approved intent before implementation. It does not write application code
or approve decisions on the person's behalf.

## What it adds

- The `prumo/spec-interviewer` and `prumo/speckit-foreman` skills, available after the project
  harness is synchronized.
- An interview catalog with starter questions for common architecture and validation findings.
- A status provider that reports `speckitPhase` as `sem-intent`, `intent-rascunho`,
  `intent-congelado` or `implementando`.
- An optional `speckit-intent-mapped` rule when the plugin is configured with code globs.
- A starter grill-me intent template for use with the plugin's scaffold API.

The catalog is a starting point, not a substitute for decisions specific to the project. The
human must approve the resulting intent; Speckit does not freeze contracts or implement the
feature.

## Installation

Install the plugin alongside Prumo at matching versions:

```bash
pnpm add -D @ihelpee/prumo @ihelpee/prumo-plugin-speckit
```

## Configuration

Register the plugin in `prumo.config.ts`:

```ts
import { defineConfig } from '@ihelpee/prumo';
import speckit from '@ihelpee/prumo-plugin-speckit';

export default defineConfig({ plugins: [speckit] });
```

Then synchronize the agent harness and start an interview:

```bash
npx prumo harness sync
npx prumo interview
```

The plugin contributes to Prumo's existing interview and status commands. It does not replace
the core gate or its verdict.

## Extension API

The package exports its catalog, skills, status provider and rule factory for integrations.
`createSpeckitRules({ codeGlobs })` returns no rule when no globs are supplied; with globs, it
reports matching source files when the project has no `.prumo/intent/*.md` file. The scaffold
helper writes the starter `speckit_grill_me.md` under `.prumo/intent/` in the selected locale.

## Compatibility

This plugin is experimental. Keep it on the same release version as `@ihelpee/prumo`; its
catalog and skill APIs may change between minor releases.
