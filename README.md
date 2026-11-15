# dvoix-plugins

The list of plugins in the plugin store of [dvoix](https://github.com/dvoi-dev/dvoix). The app reads [`registry.json`](registry.json) from the `main` branch and shows the entries under **Settings → Plugin store**.

This repo holds only the list. Each plugin's code lives in its own repository, and the app downloads it from that repository's GitHub releases.

## Official and community

- **Official plugins** are maintained by dvoix itself. They live in repositories of `dvoi-dev` and can always be installed in the app.
- **Community plugins** are written by others. Users first have to enable them in the app with the "Allow community plugins" switch and see a warning before they do.

`"channel": "official"` only applies to repositories of `dvoi-dev`. The app downgrades everything else to community, and the check in the CI here rejects it.

## Adding a plugin

1. Write the plugin. The guide with an example is in the app repo under [`docs/plugins/`](https://github.com/dvoi-dev/dvoix/tree/main/docs/plugins).
2. Create a release in the plugin repository. The tag is the version from `manifest.json` without a `v` (e.g. `1.0.0`), with `manifest.json`, `main.js` and optionally `styles.css` attached as assets.
3. Open a pull request here that adds an entry to `registry.json`:

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

| Field | Meaning |
| --- | --- |
| `id` | exactly the `id` from `manifest.json`: lowercase, `a-z0-9-`, never change it |
| `name` | display name in the store |
| `author` | who maintains the plugin |
| `description` | one sentence, in English |
| `repo` | GitHub repository `owner/name` with the releases |
| `channel` | `official` (only `dvoi-dev`) or `community` |

Updates do not need a new pull request: the app finds new releases on its own.

## Checking

```sh
node scripts/validate.mjs
```

The same check runs in the CI on every pull request.
