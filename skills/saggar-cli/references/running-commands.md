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

## Scheduled jobs

`saggar job add` asks the user to schedule a command in the calling terminal's
project. A terminal without a project creates a user-level job instead, which
runs from the home directory. The request waits for a card on the Mac with no countdown and no
reusable consent; approval applies only to the request shown.
The call blocks until the user answers, up to two minutes, and prints the next
run on success. Quote the cron expression as one word, and put `--name` first:

```sh
saggar job add --name "Nightly deps" "0 3 * * *" npm outdated
saggar job add @hourly git fetch --all
saggar job add "@once 2026-10-14 09:00" npm run release:check
```

`@once YYYY-MM-DD HH:MM` runs a single time, in the Mac's local time, and then
deletes the job; a time that has already passed is refused. The job opens a
background terminal in the project root, or home for a user-level job, on each run. To pause,
edit, or delete it, send the user to Jobs; the CLI can only ask to add one.

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

## Preserve pending work during cleanup

Use `saggar inventory --json` and inspect immediately before closing. Idle is not done.
Queued launches, prompts, reminders, unsent drafts, and dependent work keep a session
open, including with `close --force`. Use `saggar cancel <id>` only when the user
explicitly wants to cancel that work. Never infer cancellation from a cleanup request.

## Recovering closed sessions

`close` returns a recovery ID. `history` lists the last 30 days of durable archives,
including queued tasks. Recovery recreates the session and its pending work paused,
preserving stable identities, settings, and ordering. It does not start a process.
Review the recovered task, then use Resume or Resume queue in the Mac app.

```sh
saggar history --project /repos/api --search "original task" --since 2026-10-28 --json
saggar restore <recovery-id>
saggar restore --batch <batch-id>
saggar restore --batch last
```

For a cleanup operation, preview the mechanical protections, select only sessions whose
requests and results you have reviewed, and apply the chosen IDs as one undoable batch:

```sh
saggar cleanup --preview
saggar inspect <id> --json
saggar cleanup --apply <id> <other-id>
```

Eligibility is a mechanical guard, not proof that the work is complete. Apply rechecks
current queue membership and session lifetime after consent; any unsafe target refuses
the batch. Never substitute cancellation or force for a refusal. Undo through the returned
batch ID, or Sessions → Recently closed sessions → Undo this cleanup. Missing original
blockers are shown for review and never cause recovered work to execute automatically.
