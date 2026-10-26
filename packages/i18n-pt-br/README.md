# @ihelpee/prumo-i18n-pt-br 🇧🇷

<p align="center">
  <a href="https://www.npmjs.com/package/@ihelpee/prumo-i18n-pt-br"><img src="https://img.shields.io/npm/v/@ihelpee/prumo-i18n-pt-br?logo=npm&color=cb3837&label=npm" alt="npm version" /></a>
  <a href="https://github.com/ihelpee/prumo/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-proprietary-lightgrey" alt="Proprietary license" /></a>
  <a href="https://www.npmjs.com/package/@ihelpee/prumo"><img src="https://img.shields.io/badge/telemetria-zero-brightgreen?logo=ghost&logoColor=white" alt="Zero telemetry" /></a>
  <a href="https://nodejs.org/"><img src="https://img.shields.io/badge/node-%3E%3D20.11-339933?logo=node.js&logoColor=white" alt="Node >= 20.11" /></a>
</p>

> Brazilian Portuguese (pt-BR) language pack for [Prumo](https://www.npmjs.com/package/@ihelpee/prumo).

The official Prumo internationalization package for Brazilian Portuguese. It translates diagnostics, gate reports, integrity verdicts, and recommendations for developers and AI agents.

## Installation

Peer extension — requires `@ihelpee/prumo` at the **same semver** as this pack:

```bash
pnpm add -D @ihelpee/prumo @ihelpee/prumo-i18n-pt-br
# or
npm install --save-dev @ihelpee/prumo @ihelpee/prumo-i18n-pt-br
```

## Usage

In your `prumo.config.ts` file:

```typescript
import { defineConfig } from '@ihelpee/prumo';

export default defineConfig({
  locale: 'pt-BR',
});
```

Or directly from the command line:

```bash
prumo gate --lang pt-BR
```

## Author & Maintainer

**Alex Gotardi**  
- 📧 Email: [alexgotardi@gmail.com](mailto:alexgotardi@gmail.com)  
- 🐙 GitHub: [@alexgotardi](https://github.com/alexgotardi)  
- 💼 LinkedIn: [linkedin.com/in/alexgotardi](https://www.linkedin.com/in/alexgotardi/)  

---

## License & Brand

Copyright © 2026 **IHELPEE TECNOLOGIA LTDA** (<https://ihelpee.com>). All rights reserved.
Created and authored by [Alex Gotardi](https://github.com/alexgotardi).

Proprietary software, © IHELPEE TECNOLOGIA LTDA. From version 0.17.0, use is governed by the [Prumo Software License](https://github.com/ihelpee/prumo/blob/main/packages/i18n-pt-br/LICENSE.md); earlier versions keep the license they were published with.

The **"Prumo"** name and logo are trademarks of IHELPEE TECNOLOGIA LTDA and are **not** licensed under the software license.
