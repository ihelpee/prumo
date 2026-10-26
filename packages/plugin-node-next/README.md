# @ihelpee/prumo-plugin-node-next 🚦

<p align="center">
  <a href="https://www.npmjs.com/package/@ihelpee/prumo-plugin-node-next"><img src="https://img.shields.io/npm/v/@ihelpee/prumo-plugin-node-next?logo=npm&color=cb3837&label=npm" alt="npm version" /></a>
  <a href="https://github.com/ihelpee/prumo/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-proprietary-lightgrey" alt="Proprietary license" /></a>
  <a href="https://www.npmjs.com/package/@ihelpee/prumo"><img src="https://img.shields.io/badge/telemetry-zero-brightgreen?logo=ghost&logoColor=white" alt="Zero telemetry" /></a>
  <a href="https://nodejs.org/"><img src="https://img.shields.io/badge/node-%3E%3D20.11-339933?logo=node.js&logoColor=white" alt="Node >= 20.11" /></a>
</p>

> Official Next.js **App Router** architectural invariant plugin for [Prumo](https://www.npmjs.com/package/@ihelpee/prumo) — the deterministic gatekeeper and business guardrails for AI-assisted engineering.

In the App Router, a **Server Action is a public HTTP endpoint disguised as an internal function**, and `"use client"` marks a **bundle entry point**, not a file. An AI code-generation agent writes the action as if it were a trusted internal call; the compiler turns every `"use server"` `export` into a `POST` reachable by anyone. This plugin polices that boundary with pure static AST — **no Next boot, no `react`, no `next build`**.

## Invariants

| id | sev. | the hidden constructive anti-pattern | the honest boundary (what it does NOT catch) |
|---|---|---|---|
| `server-action-mutation-guarded` | error | A mutating Server Action / Route Handler writes to the database without re-verifying auth **in its own body, before the write** | pure read (`GET`); guard delegated to a `server-only` DAL that does the auth internally |
| `client-boundary-leak-prevented` | error | the client graph (reachable from a `"use client"`) imports an ORM/database client, a secret module, or reads non-`NEXT_PUBLIC_` `process.env` | shared module without ORM/secret; `await import(variableName)` (declared limitation) |
| `server-action-input-validated` | error | A Server Action mutates state without validating input via a schema (Zod/Valibot) **before** the database | a schema validates **shape, not ownership** — IDOR is a separate check, never reported as "safe" |
| `middleware-not-sole-auth` | error | authorization sustained **only** by `middleware.ts`/`proxy.ts`, with no re-verification in the actions/handlers | endpoints that re-verify auth within their own body |
| `next-public-env-not-secret` | error | a `NEXT_PUBLIC_*` with a secret name (`KEY`/`SECRET`/`TOKEN`/`DATABASE_URL`/...) is inlined into the bundle at build time | a genuinely public `NEXT_PUBLIC_*` (e.g.: `NEXT_PUBLIC_SITE_URL`) |

Compatible with **Next 14 / 15 / 16**: the extractor recognizes `middleware.ts` **and** `proxy.ts` (Next 16 renamed middleware→proxy) as the same class, and accepts both `cookies()` and `await cookies()` (Next 15 async).

## Installation

```bash
pnpm add -D @ihelpee/prumo @ihelpee/prumo-plugin-node-next
# or
npm install --save-dev @ihelpee/prumo @ihelpee/prumo-plugin-node-next
```

## Usage

In your Next app's `prumo.config.ts`:

```ts
import { defineConfig } from '@ihelpee/prumo';

export default defineConfig({
  layout: {
    src: ['app', 'lib', 'middleware.ts', 'proxy.ts'],
    oracle: ['test', 'oracle'],
    invariants: ['.prumo/invariants'],
    intent: ['.prumo/intent'],
    waivers: ['.prumo/waivers'],
  },
});
```

In `.prumo/invariants/rules.ts`, apply the preset:

```ts
import { withWaivers } from '@ihelpee/prumo';
import { nextPreset } from '@ihelpee/prumo-plugin-node-next';

// A narrow waiver is visible debt; a broad one is the gate switched off with a pretty name.
export default withWaivers(nextPreset, '.prumo/waivers/legacy.json', [
  'server-action-mutation-guarded',
]);
```

Or register the whole plugin (extractor + rules) via the contract:

```ts
import next from '@ihelpee/prumo-plugin-node-next';
// prumo plugin node-next  → shows the catalog and the config snippet
```

Run the gate:

```bash
npx prumo gate
```

## Example — the finding a mocked unit test never sees

```ts
// app/posts/[id]/page.tsx  (Server Component)
export default async function PostPage({ params }) {
  const { id } = await params;
  async function deletePost(formData: FormData) {
    'use server';                                   // ← becomes a public POST
    const postId = String(formData.get('id'));
    await db.post.delete({ where: { id: postId } }); // ← no auth(), no schema
  }
  return <form action={deletePost}>…</form>;
}
```

The happy-path test ("logged in → clicks → 200") **confirms the agent's belief**. The gate denies it:

```
server-action-mutation-guarded  app/posts/[id]/page.tsx:6 deletePost
server-action-input-validated   app/posts/[id]/page.tsx:6 deletePost
```

## Oracle: the action is an endpoint, not a function

A test that does `import { deletePost } from '@/app/actions'` and calls it as JS **bypasses the public POST** — it is a mirror, not an oracle, and the core's `oracle-isolation` flags it as such (it does not count toward the OII). The canonical oracle is the **negative test**: a `POST` to the action **without a session cookie** must fail. Forged cookies do not count as evidence.

## Author & Maintainer

**Alex Gotardi**  
- 📧 Email: [alexgotardi@gmail.com](mailto:alexgotardi@gmail.com)  
- 🐙 GitHub: [@alexgotardi](https://github.com/alexgotardi)  
- 💼 LinkedIn: [linkedin.com/in/alexgotardi](https://www.linkedin.com/in/alexgotardi/)

---

## License & Brand

Copyright © 2026 **IHELPEE TECNOLOGIA LTDA** (<https://ihelpee.com>). All rights reserved.
Created and authored by [Alex Gotardi](https://github.com/alexgotardi).

Proprietary software, © IHELPEE TECNOLOGIA LTDA. From version 0.17.0, use is governed by the [Prumo Software License](https://github.com/ihelpee/prumo/blob/main/packages/plugin-node-next/LICENSE.md); earlier versions keep the license they were published with.

The **"Prumo"** name and logo are trademarks of IHELPEE TECNOLOGIA LTDA and are **not** licensed under the software license.
