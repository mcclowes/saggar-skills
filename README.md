# saggar skills

Claude skills for [saggar](https://saggar.marginalutility.dev), the macOS terminal manager built around one question: *which terminal needs me right now?*

One skill so far. **saggar-cli** teaches an agent sitting inside a saggar terminal to speak for itself through the `saggar` command: flag its terminal as needing you and say why, dock a dev server as a monitor, run a one-shot beside your work, clear its own permission prompt, and read the project menu before deciding whether to interrupt at all.

## Install

As a Claude Code plugin, so updates arrive with the marketplace:

```
/plugin marketplace add mcclowes/saggar-skills
/plugin install saggar@saggar-skills
```

Or just copy the file: [`skills/saggar-cli/SKILL.md`](skills/saggar-cli/SKILL.md) into `~/.claude/skills/saggar-cli/SKILL.md`. Any agent runner that reads `SKILL.md` files can use it the same way.

You'll also want saggar itself:

```
brew install --cask mcclowes/saggar/saggar
```

plus the `saggar` command, installed once from the app's settings. The skill checks for both before doing anything and stays silent when they're absent, so it's safe to install globally — an agent outside a saggar terminal just carries on.

## Where this comes from

The canonical copy lives in the saggar repo, next to the CLI it documents, so behavior changes and skill text move together; this repo is the public mirror. If the skill claims something the app doesn't do, open an issue here — the fix lands there first, then syncs.
