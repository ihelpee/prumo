# @ihelpee/prumo-plugin-node-jest

<p align="center">
  <a href="https://www.npmjs.com/package/@ihelpee/prumo-plugin-node-jest"><img src="https://img.shields.io/npm/v/@ihelpee/prumo-plugin-node-jest?logo=npm&color=cb3837&label=npm" alt="npm version" /></a>
  <a href="https://github.com/ihelpee/prumo/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-proprietary-lightgrey" alt="Proprietary license" /></a>
  <a href="https://www.npmjs.com/package/@ihelpee/prumo"><img src="https://img.shields.io/badge/telemetry-zero-brightgreen?logo=ghost&logoColor=white" alt="Zero telemetry" /></a>
  <a href="https://nodejs.org/"><img src="https://img.shields.io/badge/node-%3E%3D20.11-339933?logo=node.js&logoColor=white" alt="Node >= 20.11" /></a>
</p>

Jest & test suite audit plugin for [Prumo](https://www.npmjs.com/package/@ihelpee/prumo). Inspects test ASTs for framework redundancies, mock tautologies, and performs surgical pruning of empty test suites.

**Renamed:** `@ihelpee/prumo-plugin-jest` is discontinued; use **`@ihelpee/prumo-plugin-node-jest`** ([guide](https://github.com/ihelpee/prumo/blob/main/docs/reference.md#upgrading)).

## Features

- **Redundant Guard Checks:** Identifies test cases asserting 401/403 HTTP status codes already proven by static invariants.
- **Redundant DTO Schema Checks:** Identifies tests checking 400 Bad Request on missing fields already guarded by `class-validator` / schema validation.
- **Tautological Mocks:** Detects circular mock verification (`jest.spyOn(...).toHaveBeenCalled()`) that only tests the mock setup.
- **DI Boot Checks:** Identifies boilerplate `toBeDefined()` checks duplicated across unit tests.
- **Surgical AST Prune:** Prunes redundant test blocks cleanly and automatically deletes empty test suites and orphan test files.

## Installation

Peer extension — install the gate and the plugin at the **same `@ihelpee/*` version** (lockstep releases):

```bash
pnpm add -D @ihelpee/prumo @ihelpee/prumo-plugin-node-jest
```

## Usage

In your `prumo.config.ts`:

```typescript
import { defineConfig } from '@ihelpee/prumo';
import jest from '@ihelpee/prumo-plugin-node-jest';

export default defineConfig({
  plugins: [
    jest(),
  ],
});
```

Or run via CLI:

```bash
npx prumo plugin node-jest
npx prumo node-jest test-audit --prune all
# The original core command remains available too:
npx prumo test-audit --prune all
```

## Author & Maintainer

**Alex Gotardi**  
- 📧 Email: [alexgotardi@gmail.com](mailto:alexgotardi@gmail.com)  
- 🐙 GitHub: [@alexgotardi](https://github.com/alexgotardi)  
- 💼 LinkedIn: [linkedin.com/in/alexgotardi](https://www.linkedin.com/in/alexgotardi/)  

## Corporate Ownership & License

Copyright © 2026 **IHELPEE TECNOLOGIA LTDA** (<https://ihelpee.com>). Created and authored by [Alex Gotardi](https://github.com/alexgotardi).

Proprietary software, © IHELPEE TECNOLOGIA LTDA. From version 0.17.0, use is governed by the [Prumo Software License](https://github.com/ihelpee/prumo/blob/main/LICENSE); earlier versions keep the license they were published with.
