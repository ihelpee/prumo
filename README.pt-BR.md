<p align="left"><img src="docs/assets/logo.svg" alt="Prumo" height="48" /></p>

# Prumo

Um gate de CI determinístico para bases de código escritas com agentes de IA. Ele encontra
testes que só repetem o que o código já faz e aplica regras estruturais que testes unitários não
enxergam.

[![npm](https://img.shields.io/npm/v/@ihelpee/prumo?label=npm)](https://www.npmjs.com/package/@ihelpee/prumo)
[![License](https://img.shields.io/badge/License-proprietary-lightgrey.svg)](LICENSE)

[English](README.md) · A documentação detalhada é em inglês.

## O problema

Quando o mesmo agente escreve o código e os testes, os testes herdam as suposições do código.
Se o código está errado, o teste está errado do mesmo jeito, e a suíte continua verde.

Este teste tira a expectativa da implementação. Ele é um **espelho**:

```ts
// test/orders.service.spec.ts
import { OrdersService } from '../src/orders/orders.service.js';

it('totals the items', async () => {
  const order = await new OrdersService().create({ items: [{ quantity: 2, unitPriceCents: 1500 }] });
  expect(order.total).toBe(3000);   // o teste concorda com o que o código calcular
});
```

O gate acusa (saída com `--lang pt-BR` e o pacote `@ihelpee/prumo-i18n-pt-br`):

```text
FAIL  oracle-isolation  testFiles=2 mirrors=1 clean=1
      x oracle-no-src (1) teste importa implementação: é espelho, não oráculo.
          · test/orders.service.spec.ts -> ../src/orders/orders.service.js
```

Um **oráculo** tira a expectativa de algo que não veio do código: a intenção escrita, um
contrato congelado ou infraestrutura real. Ele consegue falhar quando o código está errado:

```ts
// oracle/orders.contract.spec.ts
import type { OrderContract } from '../.prumo/contract/orders.contract.js';
// regra 1 da intenção: todo valor monetário é inteiro em centavos
```

O **Oracle Independence Index (OII)** é a fração de arquivos de teste que são oráculos. O check
`budget` reprova o build quando a suíte cresce sem o OII crescer.

## Início rápido

```bash
npm install -D @ihelpee/prumo
npx prumo setup          # skills de agente, config MCP, estrutura .prumo/
npx prumo gate           # exit 1 em falha; --json para CI
```

Node ≥ 20.11. Sem etapa de build: o `prumo.config.ts` é carregado direto, em projetos ESM ou

### Instalar com um agente de IA

Peça ao seu agente:

> Instale o Prumo neste projeto: https://github.com/ihelpee/prumo

O agente segue o [guia de instalação](docs/agent-install.pt-BR.md): ele pede confirmação antes de
escrever configuração de agente e hooks, e nunca aprova regras por você.
CommonJS.

## O que é verificado

| Check | O que pega | Bloqueia |
|---|---|---|
| `oracle-isolation` | testes que importam `src/` (caminhos relativos, aliases do tsconfig, imports dinâmicos literais) | sim |
| `invariants` | regras de framework: rotas de mutação sem guard, DTOs sem validação, imports entre features, … | em `error` |
| `contract-frozen` | contrato alterado depois do freeze (ledger de hash); contrato alterado no mesmo commit que a implementação (avisa) | sim |
| `intent-schema` | intent sem `id`/`owner`/`status`, ids duplicados, ids de regra `R-<domínio>-NNN` malformados | sim |
| `trace` | regras de intent sem contrato congelado ou sem oráculo (avisa); contratos que citam regras não declaradas | sim |
| `provenance` / OII | quais testes são independentes (isolamento + histórico git, + mutação se ligada) | — |
| `budget` | OII abaixo do piso, ou mais testes sem mais OII | sim |
| `context-integrity` | caminhos quebrados em `CLAUDE.md`, `AGENTS.md`, skills e regras | sim |
| `typecheck` | o seu `typecheckCommand` | sim |

O gate não usa LLM e não envia telemetria.

**Limites:**
- O Prumo não escreve intenção nem oráculos por você. `prumo contract draft` transforma as regras da intenção num rascunho de contrato; uma pessoa faz o freeze.
- Proveniência git é sinal de higiene de commits, não prova.
- Teste de mutação (a evidência mais forte) vem desligado por padrão.
- As barreiras de leitura das skills só valem com hook na IDE.
- Todo plugin de framework é `experimental`: as regras de Nest e Next foram conferidas em quatro repositórios públicos ([validação externa](docs/verification.md#external-validation)).

## Configuração

```ts
// prumo.config.ts
import { defineConfig } from '@ihelpee/prumo';
import nest from '@ihelpee/prumo-plugin-node-nest';

export default defineConfig({
  plugins: [nest],
  testFilePatterns: ['**/*.spec.ts', 'oracle/**/*.ts'],
  oii: { profile: 'standard' },            // fast | standard | strict
  budget: { enabled: true, minOii: 0.5, minOiiDelta: 0 },
  locale: 'pt-BR',                         // requer @ihelpee/prumo-i18n-pt-br
});
```

Para começar numa base de código existente: `npx prumo gate --preset starter`.

Estrutura:

```text
.prumo/intent/      o que o sistema deve fazer (humano, não executável)
.prumo/contract/    fronteiras congeladas: DTOs, interfaces, schemas
.prumo/invariants/  suas próprias regras (todo arquivo .ts é carregado)
.prumo/waivers/     dívida explícita, cada entrada com data de expiração e aprovador
oracle/             testes caixa-preta; nunca importam src/
src/                implementação
```

## CI

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0        # obrigatório: o OII lê o histórico git
- uses: actions/setup-node@v4
  with:
    node-version: 22
- run: npm ci
- run: npx prumo gate --json > prumo-report.json
```

## Plugins

Instale a mesma versão do core; todos os pacotes são lançados juntos.

**Plugins de framework** — regras para o gate:

| Pacote | Adiciona |
|---|---|
| [`@ihelpee/prumo-plugin-node-nest`](packages/plugin-node-nest/README.md) | NestJS: guards (decorators, APP_GUARD, middleware de auth), validação de DTO, exports de módulo, fronteiras de feature |
| [`@ihelpee/prumo-plugin-node-next`](packages/plugin-node-next/README.md) | Next.js App Router: auth em Server Actions, vazamento de fronteira client, validação de entrada, segredos `NEXT_PUBLIC` |
| [`@ihelpee/prumo-plugin-node-jest`](packages/plugin-node-jest/README.md) | Jest: análise de mock tautológico, auditoria de testes redundantes |
| [`@ihelpee/prumo-i18n-pt-br`](packages/i18n-pt-br/README.md) | saída em português do Brasil |

**Plugin de motor TypeScript** — modelo de código e planejamento de mutações usados pelo core:

| Pacote | Adiciona |
|---|---|
| [`@ihelpee/prumo-plugin-typescript`](packages/plugin-typescript/README.md) | provedor de AST TypeScript e planejador de mutações; não adiciona regras de framework |

**Plugins de fluxo de trabalho (experimentais)** — fluxo do agente em volta do gate, fora do
veredito dele. As interfaces podem mudar entre versões minor:

| Pacote | Adiciona |
|---|---|
| [`@ihelpee/prumo-plugin-speckit`](packages/plugin-speckit/README.md) | entrevistador de especificação e fluxo de planejamento Prompt Zero |
| [`@ihelpee/prumo-plugin-pipeline`](packages/plugin-pipeline/README.md) | timeline de fases e preflight para mutações em issues do GitHub |
| [`@ihelpee/prumo-plugin-github-project`](packages/plugin-github-project/README.md) | cliente de GitHub Projects/Issues (usado pelo pipeline) |

## Comandos

`gate` · `report` · `findings --promote <id>` · `status` · `start` · `allowed` · `setup` ·
`harness sync` · `interview` · `contract draft|freeze|extract|ratify` · `trace` · `badge` ·
`doctor` · `loopcheck` · `mcp`. Veja `npx prumo --help` e a
[referência](docs/reference.md#commands).

## Documentação (em inglês)

| Doc | Para |
|---|---|
| [Concepts](docs/concepts.md) | por que espelhos falham, OII, pirâmide de testes, modos de uso, limites |
| [Reference](docs/reference.md) | config, checks, regras, waivers, comandos, CI, troubleshooting, upgrade |
| [Verification](docs/verification.md) | cada afirmação com o comando que a reproduz |
| [Harness](docs/harness.md) | aplicar as barreiras de leitura das skills na IDE |
| [Architecture](docs/architecture.md) | pacotes, estrutura do core, manifesto de plugin |

## Contribuindo

Veja [CONTRIBUTING.md](SUPPORT.md). Este repositório roda o próprio gate: oráculos são
commitados separados do código que verificam, e `pnpm selftest` precisa passar.

## Licença

Software proprietário: [Prumo Software License](LICENSE), a partir da versão 0.17.0. Versões anteriores
mantêm a licença com que foram publicadas (MIT ou Apache-2.0). O nome e a marca Prumo não estão cobertos pela licença do
código; veja [TRADEMARK](docs/TRADEMARK.md). Mantido por
[Alex Gotardi](https://github.com/alexgotardi) · IHELPEE TECNOLOGIA LTDA.
