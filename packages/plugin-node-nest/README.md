# @ihelpee/prumo-plugin-node-nest

<p align="center">
  <a href="https://www.npmjs.com/package/@ihelpee/prumo-plugin-node-nest"><img src="https://img.shields.io/npm/v/@ihelpee/prumo-plugin-node-nest?logo=npm&color=cb3837&label=npm" alt="npm version" /></a>
  <a href="https://github.com/ihelpee/prumo/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-proprietary-lightgrey" alt="Proprietary license" /></a>
  <a href="https://www.npmjs.com/package/@ihelpee/prumo"><img src="https://img.shields.io/badge/telemetry-zero-brightgreen?logo=ghost&logoColor=white" alt="Zero telemetry" /></a>
  <a href="https://nodejs.org/"><img src="https://img.shields.io/badge/node-%3E%3D20.11-339933?logo=node.js&logoColor=white" alt="Node >= 20.11" /></a>
</p>

> Official NestJS architectural invariant plugin for [Prumo](https://www.npmjs.com/package/@ihelpee/prumo) — the deterministic gatekeeper and business guardrails for AI-assisted engineering.

**Renamed:** `@ihelpee/prumo-plugin-nest` is discontinued; use **`@ihelpee/prumo-plugin-node-nest`** ([guide](https://github.com/ihelpee/prumo/blob/main/docs/reference.md#upgrading)).

Provides static AST analysis rules and guardrails for NestJS codebases:
- **`mutation-routes-guarded`**: Enforces authentication / authorization guards on mutation routes (`@Post`, `@Put`, `@Patch`, `@Delete`).
- **`dto-has-validation`**: Verifies that DTO properties have class-validator decorators.
- **`providers-exported-when-cross-module`**: Validates dependency injection exports across module boundaries.
- **`no-cross-feature-import`**: Prevents architectural leakages between decoupled domain modules.
- **`monetary-is-integer`**: Prevents floating-point rounding errors for financial/monetary entities.

## Installation

```bash
pnpm add -D @ihelpee/prumo @ihelpee/prumo-plugin-node-nest
# or
npm install --save-dev @ihelpee/prumo @ihelpee/prumo-plugin-node-nest
```

## Usage

In your `prumo.config.ts`:

```typescript
import { defineConfig } from '@ihelpee/prumo';
import nestRules from '@ihelpee/prumo-plugin-node-nest';

export default defineConfig({
  invariants: [
    ...nestRules,
  ],
});
```

## Author & Maintainer

**Alex Gotardi**  
- 📧 Email: [alexgotardi@gmail.com](mailto:alexgotardi@gmail.com)  
- 🐙 GitHub: [@alexgotardi](https://github.com/alexgotardi)  
- 💼 LinkedIn: [linkedin.com/in/alexgotardi](https://www.linkedin.com/in/alexgotardi/)  

## Corporate Ownership & License

Copyright © 2026 **IHELPEE TECNOLOGIA LTDA** (<https://ihelpee.com>). Created and authored by [Alex Gotardi](https://github.com/alexgotardi).

Proprietary software, © IHELPEE TECNOLOGIA LTDA. From version 0.17.0, use is governed by the [Prumo Software License](https://github.com/ihelpee/prumo/blob/main/packages/plugin-node-nest/LICENSE.md); earlier versions keep the license they were published with.
