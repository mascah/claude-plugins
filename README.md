# claude-plugins

Mike Ascah's Claude Code plugins, in one marketplace. Larger plugins live in their own repositories; small ones live under `plugins/` here.

| Plugin | What it does |
| --- | --- |
| [mascah-skills](https://github.com/mascah/skills) | Engineering skills: discover, shape, specify and implement with proportionate verification. |
| [visual-recap](https://github.com/mascah/visual-recap) | Live view of where the agent spends its time, plus a running summary timeline of the session. |
| [codex-review](plugins/codex-review) | Iterative adversarial review of a spec, plan, or code diff via the Codex CLI. Needs the `codex` CLI; reads model settings from `~/.codex/review.config.toml` if present. |

## Install

At the prompt of a Claude Code terminal session, add the marketplace once:

```
/plugin marketplace add mascah/claude-plugins
```

Then install any plugin from it:

```
/plugin install visual-recap@mascah
/plugin install mascah-skills@mascah
/plugin install codex-review@mascah
```

From a shell, the same commands are `claude plugin marketplace add mascah/claude-plugins` and `claude plugin install <plugin>@mascah`.

In the desktop app: click + next to the prompt box, then Plugins, Add marketplace with `mascah/claude-plugins`, then Add plugin.

## Adding a plugin

Add an entry to `.claude-plugin/marketplace.json` pointing at the plugin's repository:

```json
{ "name": "<plugin>", "source": { "source": "github", "repo": "mascah/<repo>" } }
```

Or, for a small plugin kept in this repo, put it under `plugins/<plugin>/` and point at it:

```json
{ "name": "<plugin>", "source": "./plugins/<plugin>" }
```
