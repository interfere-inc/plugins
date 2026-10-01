# Interfere plugins

Investigate production errors, sessions, and releases from Claude Code, Cursor, or Codex using the Interfere MCP server.

See the [Interfere plugin](plugins/interfere/README.md) for installation, authentication, and usage.

## Repository layout

The installable plugin lives in `plugins/interfere`. It contains the client manifests, MCP configuration, investigation skill, icon, README, and license. It has no package manifest, lockfile, or build step.

The repository root contains the marketplace catalogs and development tools. All three catalogs point to `plugins/interfere`.

## Publishing

For the Claude directory, use repository `interfere-inc/plugins`, branch `main`, and plugin path `plugins/interfere`.

For Cursor, submit the repository URL. Its root `.cursor-plugin/marketplace.json` points to the plugin subdirectory. The listing logo is at `https://raw.githubusercontent.com/interfere-inc/plugins/main/plugins/interfere/assets/icon.png`.

## Development

Run these commands from the repository root.

Install [Bun](https://bun.sh), then run:

```sh
bun install --frozen-lockfile
bun run fmt
bun run lint
```

Use `bun run fmt:check` to check formatting without changing files, or `bun run lint:fix` to apply safe lint fixes. Oxlint enables every rule category, including nursery, with its default built-in plugins.

Installing dependencies sets up Husky. Before each commit, lint-staged runs Oxlint on staged JavaScript and TypeScript files, then Oxfmt on supported staged files. The tasks run sequentially and update the staged changes with their fixes.
