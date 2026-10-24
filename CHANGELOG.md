# @ihelpee/prumo

## 0.16.0

### Minor Changes

- Automatic version from conventional commits:
  
  - feat(harness): lockfile-exec MCP entry, and Claude Code reads .mcp.json (d192de1)
  - feat(mcp): agent tools for every anti-invention step (R-mcp-001) (9974b00)
  - feat(harness): Claude Code hooks turn missing artifacts into an interview (c161d9a)
  - feat(gaps): prumo gaps lists what is missing per module, with questions (cb52f2d)

## 0.15.0

### Minor Changes

- ad528ed: Intent to contract, end to end.
  
  - `intent-schema` check: every intent declares `id`, `owner` and `status` in frontmatter; intent ids and `R-<domain>-NNN` rule ids are unique and well formed.
  - `prumo contract draft` writes a deterministic contract draft per intent; `prumo contract freeze <name> --by <person>` freezes it into a committed hash ledger, and `contract-frozen` fails on edits without a new freeze.
  - `prumo contract extract` and `prumo contract ratify` for legacy code: extractors produce observed contracts that never count until ratified and frozen. Nest DTO, Pydantic and Laravel FormRequest parsers ship in their plugins.
  - `trace` check, `prumo trace` and a matrix in `prumo report`: intent rule ↔ frozen contract ↔ oracle, with optional `budget.minIntentCoverage`.
  - `prumo start` detects the project stage and prints the next step; `--init-intent` writes a draft intent. `prumo allowed` says whether a path may be written now, and the Cursor hook enforces it in strict mode.
  - Extension points in config: `checks`, `reporters`, `oiiCriteria`, `extractors`, and a declarative `pipeline` (`disabled`, `levels`).
  - Waivers now cover `provenance-gap`; every plugin declares a `maturity` (all `experimental` until measured on an independent corpus).

## 0.14.0

### Minor Changes

- Automatic version from conventional commits:
  
  - feat: English-only code with plugin i18n, oracle-isolation fixes, and core hardening (00ab6b0)
  - fix(plugins): set publishConfig access public for new scoped plugins (ac26877)

## 0.13.0

### Minor Changes

- d450dc7: Prumo v0.13: core leve e agnóstico (manifest v2, GateContext extensível, interview engine sem catálogo default, harness sync dinâmico) e extração de regras de domínio para plugins npm `@ihelpee/prumo-plugin-speckit`, `@ihelpee/prumo-plugin-github-project` e `@ihelpee/prumo-plugin-pipeline`.

## 0.13.0

### Minor Changes

- **Core agnóstico e motor de plugins manifest v2**:
  - Extração de todas as regras de domínio, skills (`spec-interviewer`), fluxos Speckit e catálogos de entrevista do core para plugins npm.
  - `GateContext` extensível (`root`, `config`, `locale`, `ledger`, `waivers`, `filesystem`, `env`, `extensions.<plugin>`), sem dependência obrigatória de convenções de layout como `sourceFiles`.
  - Manifesto de plugins v2 (`name`, `prumo`, `contributes.rules`, `contributes.cliCommands`, `contributes.statusProviders`, `contributes.harness`, `contracts`, `setup.scaffolds`, `gate.phases`).
  - Sincronização dinâmica de harness (`contributes.harness.skills`) e CLI extensível para comandos de plugins (`contributes.cliCommands`).
  - Motor de interview agnóstico sem catálogo default (`DEFAULT_INTERVIEW_CATALOG = []`), extensível via plugins.

## 0.12.0

### Minor Changes

- Versão automática a partir de commits convencionais:
  
  - feat(plugins): manifest discovery, auto-Prumo per package, and local gate (50d129b)
  - feat(harness): sync CLI, status drift, and gate harness-policy (ac7638b)
  - feat(setup): site-foreman onboarding and intent guard hook (dc7cfc4)

## 0.11.0

### Minor Changes

- Versão automática a partir de commits convencionais:
  
  - feat(invariants): load every .ts under layout.invariants (56d9028)
  - feat(setup): skill `site-foreman`, regra Cursor `prumo-foreman`, hook `preToolUse` intent-guard, intent `agent_onboarding.md`

## 0.10.1

### Patch Changes

- fix(ci): lockstep semver, peer deps on plugins, cleaner planted fixture gates in Docker oracles

## 0.10.0

### Minor Changes

- cccac51: FASE 2 S0 — governança e rastreabilidade (abre a escrita com segurança):
  
  - Migrador de ledger versionado: encadeamento por versão (`LEDGER_MIGRATORS`,
    `migrateLedger`); `schema` desconhecido continua erro com diagnóstico
    (fail-closed da FASE 0 preservado, nunca reset).
  - `writeLedger` atômico e idempotente: write-temp + rename + fsync (nunca
    truncamento in-place); reescrever o mesmo estado não produz delta.
  - Golden dos contratos públicos: `tools/list` do MCP (13 tools completas) e
    `--json` dos comandos determinísticos congelados em `oracle/golden/`
    (baseline de compatibilidade antes de qualquer mudança).
  - Tags de aresta no parser de intents: `prumo:origin` / `prumo:approval` /
    `prumo:case` / `prumo:evidence` — estritamente opcionais; intents
    existentes parseiam byte-idênticos (golden CA-P5).
  - Regras da camada via `rule()` (C12): `evidence-anchored-claims`,
    `case-anchored-intents`, `provenance-required` — cada uma com burla
    plantada no fixture `fabric-burla` (padrão C03).
  - Oráculos novos: `check-ledger-migration.ts` (CA-P4 + invariante de escrita
    sob SIGKILL), `check-intent-tags.ts` (CA-P5 + anti-burla),
    `check-contracts-golden.ts` (CP 1/2 congelados).
- f260a27: FATIA B — projeções por perfil com próxima ação derivada + léxico anti-jargão
  (`prumo status --as` pleno):
  
  - `--as stakeholder` evolui de **filtro de apresentação** para **projeção
    determinística por perfil** sobre o MESMO snapshot N1: legenda de negócio por
    tabela fechada do catálogo `Messages` (tipos de pendência, capacidades, ação e
    sinais), `comando: null` para o leigo (sem autoridade de linha de comando),
    `decisoes`/`prioridade`/`ranAt`/`proveniencia` preservados (Art. IV: ninguém é
    protegido da verdade). Nunca gravada (`no-persisted-projection` vigia);
    veredito/OII/gate idênticos ao engineer (L5: adaptação é apresentação).
  - Próxima ação por papel derivada de UM nível por prioridade fixa
    (`deriveAcaoNivel`, 00-PROPOSTA §5.3): a ação do leigo muda quando o estado
    muda (derivação, não supressão — fixture espelho no oráculo).
  - Léxico fechado de jargão versionado (`oracle/jargon-lexicon.txt`) + lei
    `layperson-projection-no-jargon` (error): regiões `prumo:jargon-scan:start|end`
    dos catálogos i18n não podem conter termo do léxico (verificação lexical, não
    subjetiva). Burla plantada em `fixtures/fabric-burla` (pega); controle
    negativo em `fixtures/fabric-claims-verde` (legenda limpa não é pega).
  - i18n dia 1: bloco `status.stakeholder` em `en` + `pt-BR` (paridade
    `check-i18n`). Intent novo `.prumo/intent/projecoes_perfil.md` (C36).
  - Oráculo `oracle/check-perfil-projection.ts` (CA-B1/B2/B3 + anti-burla +
    não-vacuidade) registrado no agregador.
  
  Edição justificada e declarada de golden (regra da casa): `tools-list.json`
  regenerado apenas pela descrição do parâmetro `as` de `prumo_status` (a
  semântica evoluiu de filtro para projeção — a descrição acompanha o contrato).
  O `--json` default (engineer) de todos os comandos congelados permanece
  byte-idêntico (CA-B4); `intents-parse.json` intocado.
- 4b2ab2c: FATIA C — interviewer contínuo com memória de decisões + oracle
  `check-interview` (fecha L1-d):
  
  - **Catálogo de perguntas desacoplado** (`skills/interview-catalog.ts`): as
    perguntas hardcoded de `analyzeCodebaseForInterview` viram fonte de dados
    (catálogo default serializável com gatilhos mecânicos `finding` /
    `finding-or-src` / `baseline`). O motor lê catálogo + findings; sem catálogo
    custom o comportamento é **byte-idêntico** ao histórico (CA-C3 — golden
    congelado da base em `oracle/golden/interview/`). Assinatura e
    `InterviewQuestion[]` preservados.
  - **Memória de decisões** (`skills/interview-memory.ts`): antes de perguntar, a
    camada consulta intent (C19), invariante em `.prumo/invariants/` e waiver
    (C18) que cubram a capacidade — decisão registrada nunca reaparece (CA-C2).
    Persistência = artefatos existentes; nenhum store paralelo. Re-sinalização
    mecânica: só o `--delta` tocando a evidência da capacidade traz a pergunta de
    volta. Omissão é declarada ao operador, nunca silenciosa.
  - **Atalho do expert** (CA-C1): com `specSource` (C11) + capacidade coberta por
    intent COM aresta `prumo:approval`, a spec é tratada como fonte — spec
    completa aprovada ⇒ `perguntas: []`.
  - **Seleção por papel em jogo** (`--role engineer|stakeholder|qa|po`): filtra o
    catálogo pelo papel (mesmo vocabulário fechado de `FabricAudience`); só
    apresentação, nunca muda o que é registrado.
  - **CLI `interview --interactive` (opt-in)**: o humano escolhe de verdade
    (corrige o default silencioso no `submit`); o default atual
    (`defaultOptionIndex`) permanece **byte-idêntico** para CI/`--batch` (CA-C3).
  - i18n dia 1: bloco `interview` em `en` + `pt-BR` (paridade `check-i18n`).
    Intent novo `.prumo/intent/interview_continuo.md` (C36).
  - Oráculo `oracle/check-interview.ts` (CA-C1/C2/C3: golden analyze/submit →
    artefatos, `tools/list` byte-idêntico, memória, delta, papel, não-vacuidade)
    registrado no agregador.
  
  Nada de superfície pública mudou: tool MCP `prumo_interview` (nome, actions,
  `inputSchema`), CLI default, `--batch` e `setup --interview` intactos — o
  golden `tools-list.json` não foi regenerado (T3 o prova byte-idêntico).
- 06d247f: FATIA D — skill de orquestração do fluxo ponta-a-ponta (lacunas 1/6/11):
  
  - **Skill nova `flow-orchestrator`** (`skills/definitions.ts`): nona skill em
    `SKILLS`, as 8 existentes intocadas. Conduz o fluxo sobre o MESMO snapshot N1
    sem virar um segundo orquestrador com estado.
  - **Camada de condução** (`core/orientation.ts`): funções PURAS sobre o
    snapshot N1 (zero LLM, zero escrita — mesma disciplina de `status.ts`).
    Compõe proveniência + projeções + interviewer e conduz as 3 lacunas do fluxo
    (00-PROPOSTA §4):
    - `avaliarEstagio` — estágio por sinais reais de descoberta (config/intents/
      fontes/report), reusa `deriveEtapa`/`deriveAcaoNivel`.
    - `conduzirPromptZero` (etapa 1) — localiza antes de perguntar: estado vago ⇒
      convite ÚNICO (setup/interview) e `perguntasTecnicas: []` SEMPRE (a camada
      convida ao interviewer nativo, nunca emite pergunta técnica — CA-D1).
    - `derivarPlano` (etapa 6) — plano derivado sob demanda, avanço curto (o
      primeiro passo espelha o nível da próxima ação derivada), sem planejador
      persistente (CA-D3).
    - `pacoteEvidencia` (etapa 11) — só apresentação; o veredito é ECO do report
      do gate (com `ranAt`) ou `null`, nunca recalculado nem atenuado (CA-D2).
  - **CLI `prumo orient`** + **tool MCP `prumo_orient`**: cada um com intent
    obrigatório (`.prumo/intent/orquestracao_fluxo.md`) e oráculo
    (`oracle/check-orientation.ts`). A lei `orientation-tool-has-oracle` foi
    estendida para vigiar a superfície de condução (intent + oráculo obrigatórios)
    e `no-persisted-projection`/`no-llm-in-assertion` agora cobrem
    `orientation.ts` (reconhecido como núcleo de superfície).
  - i18n dia 1: bloco `orient` em `en` + `pt-BR` (paridade `check-i18n`).
  - Oráculo `oracle/check-orientation.ts` (CA-D1..D4; cenários A–H materializados
    como estados de fixture; byte-determinismo em sessão fria) registrado no
    agregador (`run-oracles.mjs`, scopes `all`/`core`).
  
  Superfície pública ADITIVA: nenhuma das 13 tools existentes nem nenhum comando
  atual mudou de comportamento. O golden `oracle/golden/tools-list.json` foi
  REGENERADO via `scripts/gen-golden-contracts.ts` para incluir a tool nova
  `prumo_orient` (extensão dirigida pela spec do contrato público — precedente da
  FATIA A, que adicionou `prumo_status`). A contagem de skills no
  `oracle/check-skills.ts` acompanhou a skill nova (8 → 9).
  
  Nada é auto-executado: a condução nomeia o próximo comando pela porta
  `tools/call` existente mas nunca o executa sem confirmação humana (N3 pleno é
  adiamento deliberado). Nenhum planejador persistente, nenhuma sincronização com
  Kanban/Gitea, nenhuma fusão das 8 skills.
- 30eaf34: FATIA A — lei `unmarked-model-claims` (CA-A2, âncora mecânica do Art. IV):
  
  - Regra nova via `rule()` em `invariants/fabric-laws.ts`: inferência de modelo
    em superfície afirmativa de intent (título + descrição) sem marcação
    `prumo:origin source="model"|"hypothesis"` e sem `prumo:evidence ranAt=` com
    respaldo mecânico no ledger é finding `error`. Vocabulário fechado PT+EN
    (hedging epistêmico, opinião em primeira pessoa, atribuição ao modelo);
    tokens de enum entre backticks são referências a símbolos, não conjecturas.
  - Isenções: marcação explícita de inferência/premissa; âncora em evidência
    executada COM respaldo (fail-safe CA-E3: ledger ausente/ilegível = sem
    respaldo, conjectura continua finding).
  - Burla plantada em `fixtures/fabric-burla` (padrão C03) e controle negativo
    em `fixtures/fabric-claims-verde` (marcada `model`, âncora `executed`
    respaldada, prosa limpa — nenhum é denunciado).
  - Oráculo `oracle/check-unmarked-claims.ts` (U1 burla pega, U2 controle verde
    com respaldo, U3 fail-safe sem ledger, U4 vacuidade no repo) registrado no
    agregador. Intent da lei em `.prumo/intent/proveniencia.md` §4.

### Patch Changes

- fix(ci): plugin oracles on macOS tar and multi-arch Go images

## 0.9.0

### Minor Changes

- 9a8c164: feat(fabric): fatia vertical — `prumo status` (CLI) + `prumo_status` (MCP), 100% somente-leitura
  
  Fase 1 do Agentic Fabric. Uma função de derivação determinística (N2) sobre o
  snapshot somente-leitura do estado commitado (N1), expondo os cinco blocos
  `{etapa, decisoes, pendencias, proxima_acao, capacidades_relevantes}` e nomeando
  capacidades existentes (N3: setup/interview/gate) sem reimplementar nenhuma.
  
  - `prumo status` / `prumo_status`: aditivos; as 12 tools MCP e os 16 comandos CLI
    permanecem intactos (nome + inputSchema byte-idênticos).
  - `proxima_acao` é sempre `marcacao: "recomendacao"`; veredito só em `pendencias`
    com `ranAt` citável (recomendação ≠ verificação, P8).
  - Nada é persistido; saída byte-determinística (sessão fria byte-idêntica).
  - i18n en + pt-BR no dia 1.
  - Leis da camada via `rule()` (C12): `no-llm-in-assertion`,
    `no-persisted-projection`, `orientation-tool-has-oracle`, com fixture anti-burla.
  - Oráculo `oracle/check-fabric-status.ts` (CA-V1..V7, V9) integrado ao `pnpm oracle`.
  
  Não entrega (fases futuras): projeções por papel, escolha humana no interview,
  tags de aresta, migrador de ledger, N3 com auto-execução, sync Kanban.
- OII profiles (fast/standard/strict), provenance-gap diagnostics, fabric `status --as`, safer findings promote (route + DTO templates), Cursor harness installed by `prumo setup`, docs path migration.
- d353882: fix(ledger): schema desconhecido ou JSON corrompido agora é erro fail-closed, nunca reset silencioso. `readLedger` lançava `EMPTY_LEDGER` quando o ledger tinha `schema` ≠ 1 ou JSON inválido; o próximo `gate` gravava por cima e destruía o histórico de promoções (a evidência commitável do projeto). Agora `readLedger` lança `LedgerIntegrityError` com diagnóstico acionável (versão encontrada, suportadas, caminho, recuperação via `git checkout`) e NÃO toca no arquivo; ledger ausente (ENOENT, projeto novo) mantém o comportamento atual. `prumo doctor` passa a reportar FAIL (antes warn com texto enganoso) para schema desconhecido e JSON corrompido. Justificativa declarada: o novo contrato de erro substitui destruição de dados. Oráculo novo `oracle/check-ledger.ts` cobre schema 99, JSON corrompido, `report` e `findings --promote` com verificação byte-idêntica do arquivo no disco.
- 8047e92: feat(mutation): motor de mutação cirúrgica em delta de PR — OII critério 3 (spec `docs/especificacao-mutacao-cirurgica.md`).
  
  - `core/mutation/`: parser de delta git (`--unified=0 --no-renames`, merge-base auto-detect com fallback `HEAD~1`, nunca `main` hardcoded), operadores AST em memória (cond-inversion, bool-negation, guard-removal, boundary-shift, negation, arith-flip, block-removal), plano priorizado com cap 30 global / 10 por arquivo, e motor com deadline de 5 s (fail-open → `inconclusive`, nunca FAIL por tempo) e timeout de 2 s por desaprumo induzido.
  - Working tree intocável (D4): mutação aplicada em shadow-copy temporária com `node_modules` em symlink; seleção de testes por grafo de imports com fallback feature-para-oráculo.
  - Check `mutation` no gate: skip honesto em todos os casos de borda (sem git, sem testCommand, PR só de testes, sem base); `mutation-complacent-suite` (warn; FAIL direto no preset strict — D8); herança D1 via ledger (`mutationScore` + `killedByFile` persistidos — teste fora do delta herda, o OII global nunca degrada por arquivo não tocado).
  - A fórmula do OII NÃO muda (D2): o veto vive na produção do set `mutationPassing`; `mutationScore` entra como métrica reportada no `BudgetVerdict`.
  - Config `mutation: { enabled, base, timeoutMs, maxMutants, maxPerFile, operators, perMutantTimeoutMs }`; CLI `--mutation` / `--no-mutation` / `--mutation-base <ref>`; preset `strict` liga o motor, `starter` mantém desligado.
  - Texto do report: `passedMutation=desligado (v2)` → `desligado` (CA-8 — única mudança permitida com o motor off; comportamento byte-idêntico).
  - i18n: chaves `mutation.*` nos dois catálogos (en/pt-BR).
  - Oráculo `oracle/check-mutation.ts` (differential testing via subprocess, CA-1..CA-6 e CA-8) integrado em `pnpm oracle`.
- f1c1c09: feat(cli): `prumo badge` — badge SVG shields.io do OII a partir do último report. Lê `.prumo/last-report.json` (nunca roda o gate); tipos `oii|gate|invariants`, estilos `flat|flat-square|for-the-badge`, saída em `.prumo/badge.svg` ou stdout (`--json`/`--output -`). Badge vermelho reporta estado e sai 0; flags inválidas saem 2.

### Patch Changes

- a27f761: refactor(i18n): textos do comando `badge` migram para o contrato `Messages` como chaves `badge.*` (`badge.passing`, `badge.failing`, `badge.noReport`, `badge.written`, `badge.invalidType`, `badge.invalidStyle`). `badgeSpecFromReport` passa a receber o catálogo injetado via `getMessages(locale)` (gerador permanece puro); `oracle/check-i18n.ts` ganha verificação estrutural de paridade de chaves entre `CANONICAL_EN` e o pack pt-BR.
- ba87ae7: `prumo doctor` agora valida o binário `php` (presença e versão `>= 8.1`, R1.4/R4.2 da spec php-laravel): ausente ou abaixo do piso reporta warn com o comando corretivo e respeita `PRUMO_PHP_BIN`. O gate nunca crasha sem `php` (skip honesto via plugin), mas o doctor nomeia o runtime ausente antes do primeiro run frustrado.
- cc279c1: feat(plugin-go-gin): pacote oficial `@ihelpee/prumo-plugin-go-gin` (Fases 0–3 da spec `docs/especificacao-prumo-go-gin.md`).
  
  - Extractor Go isolado (`go-extractor/`, módulo `ihelpee.dev/prumo-go-extractor`, **stdlib-only** — R1.1): `go/parser`+`go/ast` via subprocesso emitindo o JSON canônico v1 da spec §1.4 (imports, engineRoots, routes, groupDefs, useCalls, bindings, structs, varTypes, errorHandling). Roda do módulo isolado com `GOTOOLCHAIN=local`+`GOPROXY=off`: nunca resolve deps do alvo pela rede, nunca `go build` do alvo (R1.2). A lei fica 100% no core Node (§4.2 — anti-bifurcação).
  - 11 invariantes: `mutation-routes-guarded` (R3.1–R3.6, herança transitiva Group.Use/Engine.Use obrigatória), `error-handling-enforced` (R3.7, error) + `error-handling-unchecked` (R3.8, warn heurístico com whitelist `knownErrReturning`), `payload-validation-bound` (R3.9–R3.11, body) + `payload-validation-query` (R3.12, warn), `oracle-no-src`/`src-no-oracle` Go (R2.1–R2.4, R2.7 — resolução por module path, alias/dot/blank cobertos), `oracle-whitebox-test` (R2.5, nativa de Go: `package foo` vs `package foo_test`), `oracle-module-couples-app` (R2.8, fronteira do compilador), `no-cross-feature-import` (R3.13), `monetary-is-integer` (R3.14), `go-toolchain-missing` (R1.4, warn nunca-silencioso) e `go-parse-error` (R1.3).
  - `files` do package.json inclui `go-extractor/` (o bug clássico de empacotamento apontado na spec Python não se repete).
  - Core (patch): o builder de patterns do gate aceita `.go` em `layout.src` (R1.6), espelhando o precedente do `.tsx` do node-next.
  - Fixture `fixtures/go-app` (drift plantado: rotas desprotegidas, erros engolidos, binding sem tag, oracle importando `internal/`, white-box, go.mod acoplado) + oráculo `oracle/check-go.ts` (golden file do BDD da spec) integrados em `pnpm oracle`.
- 4452661: feat(plugin-node-next): pacote oficial `@ihelpee/prumo-plugin-node-next` (Fases 0–3 da spec `docs/especificacao-prumo-node-next.md`).
  
  - Extractor `NextModel` (ts-morph, version 1): directive prologue canônico (CA-6 / TS #64033 — `import type` encerra o prologue), mapeamento de Route Handlers, Server Actions (arquivo e inline) e Server Components; `envReads`, `importsServerOnly`, `authCallLine`/`mutationLine`/`inputValidated` por função; `isProxy` cobre `middleware.ts` e `proxy.ts` (Next 16).
  - 5 invariantes `error`: `server-action-mutation-guarded` (CA-1), `client-boundary-leak-prevented` (CA-2, com trilha), `server-action-input-validated` (CA-3), `middleware-not-sole-auth` (CA-4), `next-public-env-not-secret` (CA-5). Guard delegado a DAL `server-only` não é falso positivo.
  - CLI: `prumo plugin node-next` no catálogo.
  - Core (patch): o builder de patterns do gate agora inclui `.tsx` e aceita entradas de arquivo (`middleware.ts`/`proxy.ts`), espelhando o `walk()` default do core — sem isso o App Router nunca era lido.
  - Fixtures `next-app`/`next-proxy` + oráculo `oracle/check-next.ts` (golden file da spec §9, CA-1..CA-8) integrados em `pnpm oracle`.
- 15e4c66: refinamento(plugin-python-fastapi): não-bloqueantes da homologação do PR #53 (parecer `ihelpee/prumo#53#issuecomment-1566`).
  
  - **R3.6 (downgrade Pydantic v1)** — o flag `pydantic_v1` do extractor agora é consumido pela lei: arquivos com `import pydantic.v1` têm a ausência de constraint no body DTO reportada pela regra irmã `dto-has-validation-pydantic-v1` (warn), não mais como `error` — v1 está deprecado e a equipe pode estar em migração. Partição keyed no arquivo do model (onde vivem `@validator`/`Field(regex=...)` v1). Fixture `app/legacy/` no golden file trava o downgrade e os anti-falsos-positivos (`@validator` v1 e `Field(regex=...)` v1 reconhecidos como constraint).
  - **Spec §4.3 (doctor)** — `prumo doctor` agora valida presença e versão de `python3` no host do gate (piso `>= 3.9`, o `engines.python` do plugin): warn com diagnóstico concreto (regras Python viram no-op com motivo em `python-parse-error`) e comando corretivo. Warn, nunca fail: projeto sem Python não é punido.
  - **R2.5 (fixture conftest de oracle)** — `tests/oracle/conftest.py` importando unidade de produção ganha fixture dedicado no golden file `oracle/check-python.ts` (3º achado de `oracle-no-src`), com lado negativo travado: `tests/conftest.py` fora de oracle é infraestrutura e jamais pode aparecer como alvo.
- 461979f: feat(plugin-python-fastapi): pacote oficial `@ihelpee/prumo-plugin-python-fastapi` (Fases 0–2 da spec `docs/especificacao-prumo-python.md`).
  
  - Extractor AST Python stdlib (`python/ast_extractor.py`, JSON canônico v1 congelado §1.4): imports (absolute/plain/relative resolvido), dynamic imports anti-burla (`importlib.import_module`/`__import__`), rotas FastAPI (decorator, `Depends` por parâmetro, `dependencies=` no decorator e no `APIRouter`), models Pydantic (constraints v2: `Field(...)`, `Annotated`, `con*`, `EmailStr`, `@field_validator`/`@model_validator`), `include_router` e detecção `pydantic.v1`. `SyntaxError` vira `{path, error:"syntax"}` com exit 0 (R1.3); zero dependência pip; o subprocesso nunca importa/executa o código analisado (R1.2).
  - `runPythonExtractor` (`src/ast.ts`): spawn `python3` com timeout 15s (R1.5), `python3` ausente → modelo vazio com motivo explícito e regras no-op (R1.4), walk `.py` determinístico do core (R1.6), lotes de argv.
  - 8 invariantes: `mutation-routes-guarded` (R3.1–R3.4, com resolução cross-file de `include_router` — a camada que mais gera falso-negativo), `dto-has-validation` Pydantic (R3.5–R3.7), `route-has-response-model` (warn, R3.8), `no-cross-feature-import` (warn, R3.9), `monetary-is-integer` (warn, R3.10), `oracle-no-src` (A LEI traduzida: espelho estático + burla importlib; entrypoint `app.main` permitido — oráculo black-box por ASGI), `src-no-oracle` (R2.7) e `python-parse-error` (nunca silencioso).
  - Distribuição (spec §4.2): plugin npm com o `.py` embutido em `files` (lei única no core Node; NUNCA pacote PyPI); `engines.python: ">=3.9"`; registro no catálogo `prumo plugin`.
  - Fixture `fixtures/python-app/` (drift plantado: rota desprotegida, DTO sem constraint, espelho com importlib, produção importando fixture, SyntaxError) + oráculo `oracle/check-python.ts` (golden file das R's da spec) integrados em `pnpm oracle`.
- d9d8393: fix(setup): `.prumo/.gitignore` scaffoldado passa a escrever a exceção `!badge.svg`, garantindo que `.prumo/badge.svg` (badge commitável do `prumo badge`) nunca seja ignorado por engano (spec prumo badge §6.2). A forma funcional é relativa ao diretório do próprio `.gitignore`; a literal `!.prumo/badge.svg` do spec seria padrão morto num `.gitignore` aninhado.

## 0.8.2

### Patch Changes

- 45fa6e3: - **Skill Autogeneration Banner**: Injeta aviso explícito no topo de cada `SKILL.md` gerado (`AUTOGENERATED BY @ihelpee/prumo — DO NOT EDIT DIRECTLY`), orientando a declarar regras de projeto e runbooks em `.prumo/constitution.md` ou `AGENTS.md`.
  - **Constituição Estruturada & Portabilidade CI/CD**: O scaffold de `.prumo/constitution.md` agora inclui seções guiadas para governança e decisões upstream, com instrução estrita anti-caminhos relativos quebráveis (`../`) e espaço demarcado para novas leis do projeto.
  - **Spec-Interviewer Separation of Concerns**: Atualizado o workflow do `spec-interviewer` para distinguir explicitamente regras mecânicas de AST (guards, decorators, imports) de regras de produto/negócio (payloads, limites, UX que pertencem a intent e oráculos).
- e3cbde8: - **Explicit `specSource` Configuration**: Removido qualquer parsing heurístico/regex. O usuário declara explicitamente onde residem os arquivos `.md` ou URL do Git do Spec-Driven Development via `prumo setup --upstream <source>` ou configuração `specSource` em `prumo.config.ts`.
  - **Setup Interview Hook**: Caso nenhuma especificação upstream seja informada, o `prumo setup` pergunta interativamente se o usuário deseja realizar uma sessão de entrevista (`prumo interview`) imediatamente para criar as leis e invariantes do projeto.
  - **Interview Acknowledgment**: O comando `prumo interview` reconhece a fonte configurada em `prumo.config.ts` automaticamente sem exigir repetição pelo humano.

## 0.8.0

### Minor Changes

- 75d4c0f: - **Universal Agent Standard (`.agents/skills`)**: Normalizado o diretório padrão de instalação de skills para `.agents/skills` (padrão universal da indústria reconhecido por Cursor, Claude Code, Gemini CLI, etc.).
  - **Scaffold `.prumo/.gitignore`**: Geração automática de `.prumo/.gitignore` ignorando artefatos de runtime do gate (`last-report.json`, `ledger.json`, `*.log`), mantendo os contratos, intenções e invariantes versionados no repositório consumidor.
  - **Setup Interativo Estilo GitNexus**: Seleção interativa dos destinos de skills (`.agents/skills`, Cursor, Claude, Gemini, Todos), registro automático do MCP server e apresentação clara de resumo com status.
  - **Ponytail nas Skills**: Descrições e fluxos técnicos 100% concisos, sem desperdício de tokens de contexto nem narrativa histórica.
  - **Padronização de Documentação**: READMEs, manuais e documentação de contribuição atualizados com o escopo canônico `@ihelpee/prumo` e seus plugins modulares.

### Patch Changes

- 9b64dcd: - **Root `.mcp.json` Generation**: Automatic generation/update of project-level `.mcp.json` (open MCP standard supported by Claude Code, Zed, etc.) alongside detected editor configs (`.cursor/mcp.json`, `.claude/mcp.json`).

## 0.7.0

### Minor Changes

- 3a59ab9: feat(plugin-jest): introduce dedicated @ihelpee/prumo-plugin-jest package and prumo plugin CLI command

### Patch Changes

- 622f9b7: feat(test-audit): completely remove empty test suites and orphan test files on prune
- c72813c: feat(setup): honor existing prumo.config.ts defaults on re-execution
  
  - Auto-discover existing `prumo.config.ts` and use current paths (`intent`, `contract`, `invariants`) and `locale` as default prompt choices
  - Allow developer to press ENTER to preserve existing settings and edit only desired parameters
  - Perform surgical in-place updates to existing `prumo.config.ts` without erasing other custom fields (such as `testFilePatterns`, `ignore`, or custom invariants)

## 0.6.0

### Minor Changes

- 8270038: refactor(monorepo): consolidate architecture into @ihelpee/prumo, isolated plugin-nest, and pluggable i18n language packs
  
  - Unify core engine, CLI executable (`prumo`), context assurance, invariants, and skills into `@ihelpee/prumo` with built-in canonical English
  - Isolate NestJS AST checks into pluggable `@ihelpee/prumo-plugin-nest`
  - Extract Brazilian Portuguese dictionary into standalone `@ihelpee/prumo-i18n-pt-br` language pack
  - Enable `prumo setup` to auto-install language packs on demand when non-English locale is selected
  - Graceful degradation: falls back to canonical English if language pack is uninstalled without throwing errors
- feat(context) & perf(gate): layout.context scoping, ignore exceptions, git caching and live streaming gate
  
  - Add `layout.context` and `ignore` support in PrumoConfig to avoid linting unconfigured contexts
  - Resolve relative external paths (`../`) gracefully as external references in context assurance
  - Implement early tree pruning and scoped walks in `walk()`, `checkContractFrozen`, `test-redundancy`, and `intent`
  - Add in-memory git commit caching and deduplicate `computeOii` evaluations
  - Stream gate checks live line-by-line in CLI with canonical `bin.ts` entrypoint
