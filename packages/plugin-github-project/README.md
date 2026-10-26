# @ihelpee/prumo-plugin-github-project

Experimental, read-only GitHub Projects v2 and Issues client used by Prumo's pipeline plugin.
It returns GitHub data in plain typed objects and does not decide whether an issue mutation is
allowed. The pipeline owns that policy.

## What it provides

- `GitHubProjectClient` for GraphQL requests and issue details.
- `parseIssueRef` for `owner/repo#123` references and GitHub issue URLs.
- Issue data including title, body, state, assignees, comments, sub-issues and a selected project
  status.
- Optional project number filtering when an issue belongs to more than one project.

This package does not create, edit, close, assign or move issues. It has no retry or pagination
loop: the current issue-details query reads bounded pages (up to 100 comments, 50 sub-issues and
10 project items).

## Installation

Install it with Prumo and the pipeline plugin at matching versions:

```bash
pnpm add @ihelpee/prumo @ihelpee/prumo-plugin-github-project @ihelpee/prumo-plugin-pipeline
```

## Use the client

Provide a token with access to the target issue and project. The client reads `GH_TOKEN` or
`GITHUB_TOKEN` when no token is passed explicitly.

```ts
import {
  GitHubProjectClient,
  parseIssueRef,
} from '@ihelpee/prumo-plugin-github-project';

const client = new GitHubProjectClient({ token: process.env.GH_TOKEN });
const issue = await client.getIssueDetails(parseIssueRef('owner/repo#123'), 13);

console.log(issue.projectStatus, issue.assignees, issue.subIssues);
```

The `apiUrl` option can point the client at a compatible GitHub API endpoint. Set `locale` to
`en` or `pt-BR` to localize client-generated diagnostics; English is the default.

## Use with the pipeline plugin

Register both plugins in `prumo.config.ts`:

```ts
import { defineConfig } from '@ihelpee/prumo';
import githubProject from '@ihelpee/prumo-plugin-github-project';
import pipeline from '@ihelpee/prumo-plugin-pipeline';

export default defineConfig({ plugins: [githubProject, pipeline] });
```

The pipeline uses this client to read issue and project state before evaluating its own
preflight policy. See the [pipeline README](https://github.com/ihelpee/prumo/blob/main/packages/plugin-pipeline/README.md)
for its commands and configuration.

## Errors and access

Requests fail with an explicit diagnostic when the token is missing, GitHub returns an HTTP or
GraphQL error, or the issue cannot be found. Use a token with the least privilege that can read
the repository issue and the requested project. The client does not log or persist the token.

## Compatibility

Use the same release version as `@ihelpee/prumo` and its companion plugins. This package is
experimental; its API may change between minor releases.
