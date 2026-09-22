# Running commands where the user can see them

## `monitor` versus `quick`

Both spawn a terminal in the calling terminal's project and working directory.
The difference is what happens when the command ends.

**`monitor`** docks the terminal in saggar's monitor dock, where it stays
visible. Use it for anything long-lived the user should be able to watch out of
the corner of their eye:

```sh
saggar monitor npm run dev
saggar monitor npm run test:watch
saggar monitor tail -f log/development.log
```

`--discreet` docks the card at half weight, back to full the moment the user
points at it. Reach for it when the command belongs on screen but doesn't
deserve to compete with the pane they're typing in — a log tail you started for
your own benefit, a watcher you want them to be able to check rather than one
you want them to read. It costs nothing when you're wrong: a discreet card that
starts erroring, exits badly, or lands on a prompt draws at full weight anyway.

```sh
saggar monitor --discreet tail -f log/development.log
```

**`quick`** runs the command as `<command> && exit` in a one-shot window: a
clean finish folds the window away after a beat, a failure leaves the live
prompt sitting in its output where the user can investigate. Use it for errands
that either work or want looking at:

```sh
saggar quick git pull
saggar quick npm install
```

Getting this backwards is the common mistake. A dev server in `quick` never
tidies itself away, and a `git pull` docked as a monitor leaves a dead terminal
in the dock forever.

Neither is a substitute for running a command yourself. Use these when the
output belongs on the user's screen rather than in your context — you do not
get the output back.

## Automation lifecycle

Use `add` when automation needs the Project menu populated without moving the
user's focus:

```sh
saggar add /repos/api
```

The command is idempotent. Relative paths start from the calling Saggar
terminal's directory, not from a later `cd` in its shell.

`monitor`, `quick`, and `agent` resolve their default project and working
directory from the calling Saggar terminal identified by `SAGGAR_SESSION`.
They do not infer either value from the CLI process's current directory, so a
shell `cd` does not retarget them. Pass `--cwd` to start somewhere else; a
relative path starts from the calling terminal's directory. The new terminal
still belongs to the caller's project, so to work in another project, run the
command from one of that project's Saggar terminals:

```sh
saggar monitor --cwd packages/web npm run dev
saggar quick --cwd /repos/api git pull
```

An agent can start in a configured new terminal, or reuse an existing idle one:

```sh
saggar agent codex --title "API review" --cwd /repos/api -- fix the tests
saggar agent codex --session-id 7c310000 --cwd /repos/api -- review the diff
saggar agent claude --flags '--permission-mode plan --add-dir ../shared' -- fix the tests
saggar close 7c310000
```

Use the short IDs from `saggar list`. Reuse and close refuse a terminal with a
running child process; stop its work first rather than silently killing it.
`saggar close --force <id>` closes it anyway and ends whatever is running. Use
it only when the user asked for that terminal to go; it still asks on the Mac
like any close, and the audit log records it as a forced close.
To put words into a terminal that is already busy, reach for `saggar message`
rather than trying to reuse it.

`--flags` accepts one trusted shell fragment and inserts it after the selected
provider and model arguments, before the safely quoted task. Use it only for
provider options the user asked for; shell operators in the fragment are live.
