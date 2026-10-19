# claude-plugins

Mike Ascah's Claude Code plugins, in one marketplace. Each plugin lives in its own repository; this repo only lists them.

| Plugin | What it does |
| --- | --- |
| [mascah-skills](https://github.com/mascah/skills) | Engineering skills: discover, shape, specify and implement with proportionate verification. |
| [visual-recap](https://github.com/mascah/visual-recap) | Live view of where the agent spends its time, plus a running summary timeline of the session. |

## Install

At the prompt of a Claude Code terminal session, add the marketplace once:

```
/plugin marketplace add mascah/claude-plugins
```

Then install any plugin from it:

```
/plugin install visual-recap@mascah
/plugin install mascah-skills@mascah
```

From a shell, the same commands are `claude plugin marketplace add mascah/claude-plugins` and `claude plugin install <plugin>@mascah`.

In the desktop app: click + next to the prompt box, then Plugins, Add marketplace with `mascah/claude-plugins`, then Add plugin.

## Adding a plugin

Add an entry to `.claude-plugin/marketplace.json` pointing at the plugin's repository:

```json
{ "name": "<plugin>", "source": { "source": "github", "repo": "mascah/<repo>" } }
```
