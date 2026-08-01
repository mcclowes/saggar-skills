---
name: saggar-cli
description: Drive saggar, the macOS terminal manager, from inside one of its terminals using the `saggar` command — flag this terminal as needing the user (`saggar attention`), dock a long-running command as a monitor (`saggar monitor`), run a one-shot beside the user's work (`saggar quick`), bring a terminal forward (`saggar focus`), clear your own permission prompt (`saggar approve`), and read the project menu before interrupting (`saggar status`). Use when you are blocked and want the user's eye, when a dev server, watcher, or log tail should be running where they can see it, or when you want to know which of their terminals are busy before speaking up.
---

# Driving saggar from a terminal

saggar is a macOS terminal manager built around one question: *which terminal
needs me right now?* The `saggar` command lets the agent sitting in a terminal
answer that directly instead of leaving saggar to infer it from screen chrome.
You know which line of a four-minute test run matters; no heuristic will.

## Check the channel exists first

Every verb except `saggar <path>` needs three things: saggar running, the shim
installed (the user does that once from Settings ▸ General), and a
`SAGGAR_SESSION` in the environment naming the terminal you are standing in.

```sh
[ -n "$SAGGAR_SESSION" ] && command -v saggar >/dev/null
```

If that fails, you are not in a saggar terminal. Carry on with the task and say
nothing about it — an absent saggar is not an error worth reporting to the user.
The same goes for exit code 3 at runtime, which means there was nobody to ask.

## The verbs

| Command | What it does |
| --- | --- |
| `saggar attention "why"` | Flag this terminal as needing the user, with the reason |
| `saggar attention --note "why"` | Same, as a look-when-free note that makes no claim |
| `saggar monitor <command…>` | Dock a monitor in this project running the command |
| `saggar quick <command…>` | Run the command in a one-shot window beside the user's work |
| `saggar focus [name]` | Bring a terminal to the main pane (no name means this one) |
| `saggar approve` | Answer the permission prompt waiting in this terminal |
| `saggar status [--json]` | Print the project menu as it stands |
| `saggar <path>` | Add or focus a repo on the project menu; the only verb that can launch saggar |
| `saggar help` | Print the verbs and flags |

`monitor` and `quick` take the rest of the line, so `saggar monitor npm run dev`
works without quoting. Quote when spacing matters. `attention` joins its words
the same way, and `--note` may sit anywhere among them.

A call takes up to one status beat (~2s) to land. That is invisible for
everything here.

## Raising attention

This is the verb worth being deliberate about, because it spends the user's
attention and there is no way to give it back.

Raise it when you are actually stopping:

```sh
saggar attention "migration test fails on a fresh DB — need a decision on the fixture"
saggar attention "three approaches, all reasonable, want a steer before I commit"
```

Use `--note` when the run should stand but the user would want to know:

```sh
saggar attention --note "tests green; the flaky one passed on the second try"
```

Habits that keep the flag meaning something:

- **The default is `blocked`, and that is on purpose** — a dropped or mistyped
  flag should be louder rather than quieter. Reach for `--note` explicitly.
- **Say what to look at, not that something happened.** "the regression is in
  the retry backoff" beats "task finished". The message is one line, capped at
  200 characters.
- **Once per stop, not once per step.** An agent flagging every tool call is a
  broken terminal, and the rate limit (20 commands per 10 seconds per terminal)
  reads it that way.
- **A `blocked` call ages out after two minutes, or the moment the user types in
  that terminal.** You do not need to withdraw it, and you cannot: raising a
  flag is a verb, lowering one is not.

## Approving your own prompt

`saggar approve` types the default choice into the permission prompt waiting in
your terminal — the same answer the user's own tick sends. It is the one verb
that answers rather than asks, and it always acts on your session; it takes no
argument and never touches another terminal.

```sh
saggar approve
```

It can be refused, and the exit-1 message on stderr says why. The three
refusals worth knowing:

- **The user turned it off.** "approving from the terminal is off in Settings
  ▸ AI" is a settled answer, not a transient one. Do not retry, and do not look
  for a way around it — flag the terminal and stop:
  `saggar attention "blocked on a permission prompt"`.
- **A destructive command is on screen.** `rm -rf`, a force push, a raw disk
  write and their kin are held back whatever the setting says, the same line
  turbo stops on. The message names the shape. That prompt is the user's; leave
  it and say why.
- **No answerable menu.** The needs-you came from a bell, or from a wizard with
  no numbered choices, so there was nothing to type.

Habits:

- **Approve your own stop, not as a substitute for asking.** If the prompt is
  the CLI checking something you should have checked — a migration, a
  force-push, a file outside the repo — the honest move is `attention`, not
  `approve`.
- **Once, then look.** If the same prompt is still there on the next turn, your
  answer didn't land. Calling again won't help (a re-answer of the same prompt
  is refused by fingerprint); raise a flag instead.

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

## Read the project menu before interrupting

`saggar status` is what makes the channel worth having rather than merely
usable. It prints one line per terminal — status, project, name, agent — so you
can see whether the user is already dealing with something louder:

```sh
saggar status            # needs-you / working / idle / done / failed, per terminal
saggar status --json     # the same as a snapshot object, for parsing
```

The JSON is one snapshot object; its `sessions` array carries each terminal's
`id`, `projectName`, `projectPath`, `name`, `status` (a `kind` plus any
`exitCode`), `agent`, `promptWaiting`, `lastActivityAt`, git state, and a short
`transcriptTail` of its screen. If two other terminals already read needs-you,
a third flag is noise; finish, leave a `--note`, and let the user come to you.

## What the CLI deliberately cannot do

Knowing the shape of the boundary saves you probing it:

- **No verb lowers a flag.** There is no ack, no dismiss, no way to mark another
  terminal's claim answered. An agent that could silence its own needs-you could
  report success it had not earned. `approve` is not an exception to this: it
  *answers* the prompt, and your next turn is the proof it landed.
- **A command speaks for the terminal it runs in.** `monitor`, `quick`,
  `attention`, and `approve` always act on your own session. Only `focus` takes
  another terminal's name, and only to bring it forward.
- **There is no session management.** No `saggar new`, no `saggar send`, no
  `saggar close`. The shell already opens terminals.
- **Every verb that acts is audited**, denials included
  (`~/.saggar/control-audit.jsonl`). Assume the user can see what you called.

## Exit codes

| Code | Meaning |
| --- | --- |
| 0 | Done |
| 1 | Understood, and saggar refused or could not do it |
| 2 | The arguments did not parse |
| 3 | Nobody to ask — saggar is not running, or this is not a saggar terminal |

Treat 3 as "the feature is not available here" and move on silently. Treat 1 as
worth reading the message on stderr once, not worth retrying.

## A worked shape

The end of a long, failed run, from inside a saggar terminal:

```sh
npm test > /tmp/test.log 2>&1
if [ $? -ne 0 ] && [ -n "$SAGGAR_SESSION" ]; then
  saggar attention "auth middleware test fails on the token refresh path"
fi
```

And when the user has asked for a change they will want to look at:

```sh
saggar monitor npm run dev
saggar attention --note "dev server docked on :3000, the new route is /settings/agents"
```
