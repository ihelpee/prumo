# Install Prumo with an agent

This guide is for an AI agent (Claude Code, Cursor, Codex, Gemini CLI or another) that was asked
to install Prumo in a project. Follow the steps in order. Stop when a step asks for confirmation.

## Before you start

- Confirm that the directory is the project root: it must contain `package.json`, or the project
  must be a git repository. If neither exists, stop and ask the person.
- Confirm Node.js 20.11 or newer (`node --version`). If it is older, stop and report the version
  you found.
- Find the package manager from the lockfile: `pnpm-lock.yaml` means pnpm, `yarn.lock` means yarn,
  anything else means npm.

## 1. Install the package

Use version 0.18, so the project gets exactly what was tested.

```bash
npm install -D @ihelpee/prumo@^0.18 @ihelpee/prumo-plugin-typescript@^0.18
```

With pnpm: `pnpm add -D @ihelpee/prumo@^0.18 @ihelpee/prumo-plugin-typescript@^0.18`. With yarn: `yarn add -D @ihelpee/prumo@^0.18 @ihelpee/prumo-plugin-typescript@^0.18`.

## 2. Show the plan and ask for confirmation

The next two steps write agent configuration files and hooks into the project. Before running
them, tell the person which agents you will configure and what will be written, and wait for a
"yes".

Possible agents: `agents` (the universal default), `cursor`, `claude`, `codex`, `gemini`.
Choose the ones the person uses. If they do not know, use `agents`.

## 3. Configure the project

```bash
npx prumo setup --target <agents> -y
npx prumo sync --target <agents>
```

- `setup` creates the `.prumo/` folder, the skills, the MCP configuration file and the
  instruction block in `AGENTS.md` (or in the file of the chosen agent).
- `sync` installs the agent hooks.
- Both accept `--dry-run`: use it first if the person wants to see the result before anything is written.

Do not use `--prune` or `contract freeze`. The first deletes tests and the second records an
approval; both are human decisions.

## 4. Verify

```bash
npx prumo doctor
npx prumo gate
npx prumo status --json
```

- `doctor` shows the environment. If it reports a problem, show it to the person before fixing
  anything.
- `gate` shows the verdict. A failure on the first run is expected in a project without intents;
  do not try to change the code because of it.
- `status --json` says which stage the project is in and what the next step is. Follow it, and
  stop if the next action needs a human decision.

## 5. Next steps with the person

After installation, writing the rules is the person's job. Explain in one or two sentences what
is missing:

- **Interview:** `npx prumo interview --json` asks the questions one at a time. You record the
  answers as a draft intent, show it to the person and wait for their approval.
- **Approval:** only the person approves an intent. Never mark a rule as approved without their
  explicit "yes", and never run `contract freeze` or `findings --promote` on your own.
- **Day to day:** run `npx prumo gate` before calling a task done, and `npx prumo status --json`
  when you do not know the next step.

## If something fails

- The package install fails: show the full error message to the person. Do not change the version
  without asking.
- `setup` or `sync` fails: do not run them again with other options. Report the message and stop.
- `gate` fails with an environment error (Node, paths, configuration file): use `doctor` to find
  the cause.
