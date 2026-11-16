# AGENTS.md

Fixed rules for all AI agents (Claude, Codex, Copilot, …) working in this repo. This file is the main source; `CLAUDE.md` only points here.

## What this is

The list of plugins in the plugin store of [dvoix](https://github.com/dvoi-dev/dvoix). The app loads `registry.json` from the `main` branch and shows the entries under **Settings → Plugin store**. This repo holds only that list and its check, no plugin code.

## The dvoix repos

dvoix is spread across several repositories. This section is also in the `AGENTS.md` of every repo. Whoever changes it, changes it in all of them.

| Repo | Purpose | What belongs there |
| --- | --- | --- |
| [`dvoi-dev/dvoix`](https://github.com/dvoi-dev/dvoix) | the app | Rust core, Tauri shell, Svelte frontend, core plugins, plugin API, plugin store in the app, developer guide for store plugins (`docs/plugins/`), developer README |
| [`dvoi-dev/dvoix-website`](https://github.com/dvoi-dev/dvoix-website) | product page and docs | product page at `/`, user and plugin developer docs at `/docs`, screenshots for the page, Docker image for the server |
| [`dvoi-dev/dvoix-plugins`](https://github.com/dvoi-dev/dvoix-plugins) | list of the plugin store | only `registry.json` and its check (`scripts/validate.mjs`). No plugin code |
| `dvoi-dev/dvoix-<name>` | one official store plugin each | code, `manifest.json`, releases with `manifest.json`, `main.js`, `styles.css` |

### Where does what go?

| Task | Repo |
| --- | --- |
| Basic note and editor function that is always there (explorer, search, tags, backlinks, tabs, views, …) | `dvoix`, as a core feature under `src/lib/features/` |
| Optional function that ships with the app (tasks, canvas, graph, …) | `dvoix`, as a core plugin under `src/lib/plugins/core/` |
| Extra function not everyone wants (kanban, projects, Git, tracking, …) | its own repo `dvoi-dev/dvoix-<name>` as a store plugin, plus an entry in `dvoix-plugins` |
| Extend the plugin API (`PluginContext`, permissions) | `dvoix`, plus `docs/plugins/README.md` there and the `plugin-api.md` page on the website |
| Explain how to use dvoix | `dvoix-website`, page in `src/content/docs/` |
| Offer or remove a plugin in the store | `dvoix-plugins`, `registry.json` |
| Bug in a store plugin | in the plugin's repo, not in the app |

### What is connected across repo boundaries

- **Format of the plugin list:** defined in `crates/dvoix-core/src/store.rs` (`parse_registry`) of the app, example in `docs/plugins/registry.example.json`, checked by `scripts/validate.mjs` in `dvoix-plugins`. If the format changes, all three places change.
- **Address of the plugin list:** the app reads `https://raw.githubusercontent.com/dvoi-dev/dvoix-plugins/main/registry.json`. Never change the repo name, the branch `main` or the file name, or no installed app finds plugins anymore.
- **Official:** `"channel": "official"` only applies to repos of `dvoi-dev`. The app and the check in `dvoix-plugins` both enforce this.
- **Plugin API:** `src/lib/plugins/store/api.ts` in the app, `docs/plugins/README.md` in the app and `src/content/docs/plugin-api.md` on the website describe the same API and must match.
- **Features of the app:** what the website docs say about the app must match the code in `dvoix`. If the app changes in a way users see, the docs are updated.

### Changes across several repos

- One PR per repo. The PRs link to each other in their descriptions.
- Merge order: first the app (`dvoix`), then `dvoix-plugins` and `dvoix-website`. A doc or plugin list must not promise anything the app does not yet support on `main`.
- For all repos the same applies: code, comments and commit messages in English, everything users see in English too; always the newest stable versions of Node.js, dependencies and GitHub Actions; never push directly to `main`.

## Basic rules

1. **Only the list.** This repo holds `registry.json`, its check and the docs for it. Plugin code, releases and issues about a plugin belong in the plugin's repo.
2. **The app decides the format.** The source of truth is `parse_registry` in `crates/dvoix-core/src/store.rs` in the app repo. `scripts/validate.mjs` checks the same rules, only stricter: whatever the app silently skips makes the CI fail here. If one changes, the other is updated in the same change.
3. **The address stays fixed.** The repo name, the `main` branch and the file name `registry.json` never change. The app does not read `"version"` today: an incompatible format only goes live once an app version can read it, and older apps must not break on it.
4. **Official means `dvoi-dev`.** `"channel": "official"` only for repos of `dvoi-dev`. Everything else is `community`.
5. **Never change an `id`.** It is the `id` from the plugin's `manifest.json` and the folder name in `<vault>/.dvoix/plugins/<id>/`. Renaming it means, for users: plugin gone, settings gone.
6. **Always the newest versions.** Node.js (the current version, not an older LTS) in `.nvmrc` and every GitHub Action on its newest stable version. Older versions only with a reason in the PR.

## Layout

```
registry.json                  the plugin list the app loads
scripts/validate.mjs           checks registry.json like the app does (no dependencies)
.github/workflows/validate.yml runs the check on every PR and on main
.nvmrc                         Node version for CI and local use
README.md                      guide for plugin authors: how to add a plugin
```

## Entries in `registry.json`

```json
{
  "id": "word-count",
  "name": "Word Count",
  "author": "your-name",
  "description": "One sentence about what the plugin does.",
  "repo": "your-name/dvoix-word-count",
  "channel": "community"
}
```

- All six fields are required and must not be empty.
- `id`: lowercase, `a-z0-9-`, at most 64 characters, unique in the list, equal to the `id` in `manifest.json`.
- `name` and `description` in English; the description is one sentence.
- `repo`: `owner/name` on GitHub. The repo must have a release whose tag is the version from `manifest.json` without a `v`, with `manifest.json`, `main.js` and optionally `styles.css` as assets.
- New entries go at the end of the list. Do not add versions: the app finds new releases on its own.
- Remove a plugin only with a reason in the PR (broken, abandoned, harmful). Copies users have already installed are not affected.

## Reviewing an entry

Before merging a new entry:

- The repo exists, is public and has a matching release with `manifest.json` and `main.js`.
- The `id` in the entry and in `manifest.json` match.
- The permissions in `manifest.json` (`vault:read`, `vault:write`, `editor`) fit what the plugin does according to its description.
- For community plugins, skim the release's `main.js`: no network access without a clear reason, no obfuscated code. Permissions are not a sandbox; this review is the only protection before a plugin is listed.

## Commands

```sh
node scripts/validate.mjs   # checks registry.json, just like the CI
```

The check must be green before every commit. There are no npm dependencies and no `npm install`. If the check cannot run locally, say so explicitly instead of claiming success.

## Conventions

- **Language:** everything is in English: code, identifiers, comments, commit messages and everything users or plugin authors see (README, `name`, `description`, the messages of `validate.mjs`).
- **Comments** explain the why, not the what. Every file starts with a short comment about what it is for.
- **No dependencies.** `validate.mjs` runs on plain Node.js. Keep it that way so the CI works without `npm ci`.
- **`registry.json`** indented with two spaces, one object per entry, keys in the order `id`, `name`, `author`, `description`, `repo`, `channel`.

## Workflow

- Never push directly to `main`.
- Commit messages in English, in the imperative, with a one-line summary (e.g. "Add kanban plugin to the registry").
- PR description: which plugin, link to the repo and the release, how it was checked.
- Update the README and these rules when the format, the check or the process changes.
