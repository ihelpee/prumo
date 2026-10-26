# @ihelpee/prumo-plugin-typescript

**Experimental TypeScript engine for [Prumo](https://www.npmjs.com/package/@ihelpee/prumo).**
It provides the TypeScript AST provider and mutation planner used by Prumo's
invariant engine. This package does not ship TypeScript-specific rules by
itself.

## Requirements

- Node.js `>=20.11`
- `@ihelpee/prumo` at the same release version as this plugin

## Installation

Install Prumo and this plugin together so their versions stay aligned:

```bash
pnpm add -D @ihelpee/prumo @ihelpee/prumo-plugin-typescript
```

## Configuration

Add the plugin to `prumo.config.ts`:

```typescript
import { defineConfig } from '@ihelpee/prumo';
import typescript from '@ihelpee/prumo-plugin-typescript';

export default defineConfig({
  plugins: [typescript()],
});
```

The package exports TypeScript AST helpers, the AST provider, mutation
operators, and the mutation planner for integrations that need those APIs.

## License

Proprietary software. See the license file included in this package.
