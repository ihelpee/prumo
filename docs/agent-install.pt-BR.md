# Instalar o Prumo com um agente

Este guia é para um agente de IA (Claude Code, Cursor, Codex, Gemini CLI ou outro) que recebeu
o pedido de instalar o Prumo num projeto. Siga os passos na ordem. Pare quando um passo pedir
confirmação.

## Antes de começar

- Confirme que o diretório é a raiz do projeto: deve existir `package.json` ou o projeto deve ser
  um repositório git. Se não houver nenhum dos dois, pare e pergunte à pessoa.
- Confirme Node.js 20.11 ou superior (`node --version`). Se for mais antigo, pare e informe a
  versão encontrada.
- Descubra o gerenciador de pacotes pelo lockfile: `pnpm-lock.yaml` usa pnpm, `yarn.lock` usa yarn,
  os demais usam npm.

## 1. Instalar o pacote

Use a versão 0.18, para que o projeto receba exatamente o que foi testado.

```bash
npm install -D @ihelpee/prumo@^0.18 @ihelpee/prumo-plugin-typescript@^0.18
```

Com pnpm: `pnpm add -D @ihelpee/prumo@^0.18 @ihelpee/prumo-plugin-typescript@^0.18`. Com yarn: `yarn add -D @ihelpee/prumo@^0.18 @ihelpee/prumo-plugin-typescript@^0.18`.

## 2. Mostrar o plano e pedir confirmação

Os próximos dois passos escrevem arquivos de configuração de agente e hooks no projeto. Antes de
rodar, diga à pessoa quais agentes você vai configurar e o que será escrito, e espere um "sim".

Agentes possíveis: `agents` (o padrão universal), `cursor`, `claude`, `codex`, `gemini`.
Escolha os que a pessoa usa. Se ela não souber, use `agents`.

## 3. Configurar o projeto

```bash
npx prumo setup --target <agentes> -y
npx prumo sync --target <agentes>
```

- `setup` cria a pasta `.prumo/`, as skills, o arquivo de configuração do MCP e o bloco de
  instruções no `AGENTS.md` (ou no arquivo do agente escolhido).
- `sync` instala os hooks do agente.
- Os dois aceitam `--dry-run`: use antes se a pessoa quiser ver o resultado antes de qualquer escrita.

Não use `--prune` nem `contract freeze`. Esses comandos apagam testes ou registram aprovações, e
são decisões humanas.

## 4. Verificar

```bash
npx prumo doctor
npx prumo gate
npx prumo status --json
```

- `doctor` mostra o ambiente. Se apontar um problema, mostre à pessoa antes de corrigir.
- `gate` mostra o veredito. Uma reprovação na primeira execução é esperada num projeto sem
  intents; não tente corrigir o código por causa dela.
- `status --json` diz em que estágio o projeto está e qual é o próximo passo. Siga o que ele
  indicar, e pare se a ação exigir decisão humana.

## 5. Próximos passos com a pessoa

Depois da instalação, o trabalho de definir regras é com a pessoa. Explique, em uma ou duas
frases, o que falta:

- **Entrevista:** `npx prumo interview --json` faz as perguntas uma por vez. Você registra as
  respostas como rascunho de intent, mostra à pessoa e espera a aprovação dela.
- **Aprovação:** só a pessoa aprova uma intent. Nunca marque uma regra como aprovada sem um "sim"
  explícito dela, e nunca rode `contract freeze` ou `findings --promote` por conta própria.
- **Dia a dia:** `npx prumo gate` antes de dar uma tarefa por pronta, e `npx prumo status --json`
  quando não souber o próximo passo.

## Se algo falhar

- Erro de instalação do pacote: mostre a mensagem completa à pessoa. Não troque a versão sem
  perguntar.
- `setup` ou `sync` falhar: não rode de novo com outras opções. Informe a mensagem e pare.
- `gate` com erro de ambiente (Node, caminho, arquivo de configuração): use `doctor` para
  identificar a causa.
