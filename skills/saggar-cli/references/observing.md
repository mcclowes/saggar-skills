# Watching other terminals

## Keep status reads compact

Use plain `saggar list` to find terminal IDs and statuses. Its JSON snapshot can
include full conversations for many terminals. When structured metadata is
needed, project only the fields needed for the task before loading it into
context. For example, with `jq`:

```sh
saggar list --json | jq '{sessions: [.sessions[] | {id, projectPath, name, status}]}'
```

Use `inspect` for a selected terminal's work state, or bounded `read` when its
output or conversation is needed.

## Find the focused terminal

```sh
saggar focused
saggar focused --json
```

This reads the app's focused-terminal mirror without the command spool or a
`SAGGAR_SESSION`. It reports ID, name, agent, status, project, and git metadata;
it never includes terminal output or conversation. It observes focus without
changing it. Exit 3 means Saggar isn't running or its mirror is unavailable;
exit 1 means the key window has no terminal focused. Neither means the calling
terminal is selected.

## A `saggar session <id>` line is someone pointing at a terminal

The user can drag a terminal from saggar's project menu into a chat, which
leaves a line like `saggar session 7c310000` in what they send you. It is a
reference, not an instruction: they are naming *that* terminal in the sentence
around it. Read the sentence for what they want, and use the short id with the
read-only verbs when you need the terminal itself:

```sh
saggar session 7c310000     # its name, presentation, and turbo setting
saggar inspect 7c310000     # latest turn, pending prompt, and resume readiness
saggar read 7c310000        # what it has printed
saggar explain 7c310000     # why it wears the status it does
```

Running the line as given is harmless — it prints three settings — but it is
rarely the whole answer. Nothing about the reference grants you more than the
usual read-only access to that terminal.

## Wait instead of polling

`saggar wait` blocks until another terminal is worth looking at, so a loop of
`list` and `sleep` is never the right shape. Without options it returns as soon
as the terminal is anything but `working`: needs-you, idle, finished, or
failed. A status that already matches returns at once.

```sh
saggar wait 7c310000                        # until it stops working
saggar wait 7c310000 --until needs-you      # until it asks for someone
saggar wait 7c310000 --until finished --until failed --timeout 600
saggar wait 7c310000 --match "listening on" # until a tail line contains the text
```

`--until` takes the same words `list` prints and may repeat; `--match` waits
for a literal substring of one line in the last 80 lines instead, and the two
can't be combined. `--timeout` is in seconds, 120 by default and 3600 at most.
On success the reply is the terminal's `list` line, plus the matched line for
`--match`; `--json` prints the record. A timeout exits 1 and names the status
the terminal is still in. If the terminal closes while you wait, the wait fails
with "session disappeared" rather than settling on whatever replaced it.

Waiting is observation: it never types, answers, or lowers a flag. Delegated
work still has no result channel back to you; `wait` tells you *when* to
`read`, not what was said.

## Ask why a status is what it is

`saggar explain` prints the rule that decided a terminal's status and the
evidence around it: the matched prompt cue or hook event, whether the user
already marked a claim seen, whether turbo is presenting a needs-you as
working, which agent was found and how (the process walk, its hooks, or its
screen chrome), and how long ago output, input, and the bell last happened.

```sh
saggar explain              # this terminal
saggar explain 7c310000     # another one
saggar explain 7c310000 --json
```

Use it before reporting that saggar has a terminal's status wrong, and quote
the `because` line when you do. The rule words are stable: `exit-code`, `clipboard-read`,
`agent-call`, `hook-claim`, `prompt-cue`, `bell`, `watch-cue`, `notification`,
`command-running`, `background-work`, `turn-ended`, `turn-running`, `idle-prompt`,
`recent-output`, and `quiet`.

## Inspect work and recovery

`saggar inspect` combines a live status with the latest recorded turn and the
resume path Saggar can see:

```sh
saggar inspect 7c310000
saggar inspect 7c310000 --json
```

The request excerpt is one line, capped at 120 characters. The result reports
turn state and edited-file count, prompt and queued-launch state, and whether
the provider identity or an approved custom binding supports a resume. It
never returns a provider session ID, custom command, environment values, or a
full transcript. A missing ledger means no turn is recorded; it doesn't prove
the session is done.

## Turns

Saggar's Claude Code and Codex hooks record a turn ledger under
`~/.saggar/turns/v1/`: each prompt opens a turn, supported file-editing tool
events name files, and the stop closes it with a snapshot of the worktree. The
hooks call `saggar turn <event>` for you; there is nothing to invoke during a task.

<!-- saggar:if fettle -->
Fettle, the sibling review app, reads that ledger to show the user what each
turn changed.
<!-- saggar:end fettle -->

`saggar turns` shows your own session's turns, newest first, with a short id,
state, time, edited-file count, and the first line of the prompt. `--json`
prints the full records. It works without a running Saggar app and needs only
`SAGGAR_SESSION`.

```sh
saggar turns
saggar turns --json
```
