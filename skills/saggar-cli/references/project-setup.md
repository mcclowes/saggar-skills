# Project and provider setup

## Project configuration

`saggar config` prints the current folder's `.saggar/project.json`. It works
without a running Saggar app. Pass one or more options to update the manifest:

```sh
saggar config --name "Example site"
saggar config --clear-name
saggar config --docs handbook
saggar config --clear-docs
saggar config command --name "Example site" --monitor --browser --pin -- npm run start
```

The name is shared with everyone who checks out the project; clearing it falls
back to the folder's name. The documentation folder must be a relative path
inside the project. Run
`saggar init` first when
the folder doesn't have a manifest.

The `command` form writes `.saggar/commands.md`. Use one of `--primary`,
`--companion`, `--monitor`, `--background`, or `--quick` to choose where it
opens. Add `--browser` or `--simulator` when running the command should reveal
that live surface on a remote client. The surface is independent of terminal
placement, so `--monitor --simulator` is valid. `--pin` also makes it the one-click command in this Mac's terminal
headers, so the project must already have been opened in Saggar. Repeating the
same name replaces its command, role, and live surface instead of adding a duplicate.

## Provider hooks

`saggar hooks` reports the presence handshake for every provider saggar can
hook, without a running app:

```sh
saggar hooks
saggar hooks --json
```

Each row is a provider, one of `installed`, `not-installed`, or `no-cli`, and
the config file an install would edit. Anything still standing between a
written config and a hook that runs prints under the table, named by its
provider. Saggar cannot verify Codex hook trust or effective managed policy;
check `/hooks` in Codex for the current definitions.

Reading is always safe. `saggar hooks install <provider>` and `saggar hooks
remove <provider>` are not: they edit the user's own provider configuration,
outside any project. Run them when the user asks you to set hooks up, and not
otherwise — Settings ▸ Providers is where they would do it themselves, and an
install they did not ask for is a change to their machine they did not make.

```sh
saggar hooks install claude
saggar hooks remove codex
```

The install refuses when the provider's own CLI isn't on this Mac, and names
the command that installs it. Claude's install can collide with a marketplace
named saggar registered from somewhere else; it stops and says so rather than
replacing it, and `--replace-marketplace` is the user's decision to make.

## Durable custom resume

Use a custom binding only when a durable terminal tool has its own checkpoint
but no native Saggar provider integration:

```sh
saggar resume set --kind tmux --checkpoint build:1 --cwd /repos/api \
  --provenance "tmux integration" -- tmux attach -t build
saggar resume show
saggar resume clear
```

Repeat `--env KEY=VALUE` only for non-sensitive values the resume command
needs. Saggar rejects token, password, secret, API key, and private-key names.
The binding stays untrusted until the user approves its exact command, working
directory, and environment in Settings ▸ Sessions. Do not ask the user to
approve it. An untrusted binding restores as an idle shell for manual use, and
a native provider session ID always takes precedence.
