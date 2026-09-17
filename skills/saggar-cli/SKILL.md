---
name: saggar-cli
description: Drive saggar, the macOS terminal manager, from inside one of its terminals using the `saggar` command — list and read terminals, raise attention, start monitors and one-shots, delegate durable work to a new agent terminal, focus a terminal, and clear your own permission prompt. Use when you are blocked and want the user's eye, when a process should run where they can see it, when work needs an independent user-owned agent session, or when you want to inspect which terminals are busy before speaking up.
---

# Driving saggar from a terminal

saggar is a macOS terminal manager built around one question: *which terminal
needs me right now?* The `saggar` command lets the agent sitting in a terminal
answer that directly instead of leaving saggar to infer it from screen chrome.
You know which line of a four-minute test run matters; no heuristic will.

## Check the channel exists first

Every command except `saggar init`, `saggar config`, `saggar settings`, `saggar hooks`, `saggar turns`, `saggar schema`, `saggar help`, and `saggar <path>` needs three things:
saggar running, the shim installed (the user does that once from Settings ▸ CLI), and a
`SAGGAR_SESSION` in the environment naming the terminal you are standing in.

```sh
[ -n "$SAGGAR_SESSION" ] && command -v saggar >/dev/null
```

If that fails, you are not in a saggar terminal. Carry on with the task and say
nothing about it — an absent saggar is not an error worth reporting to the user.
The same goes for exit code 3 at runtime, which means there was nobody to ask.

## Respect the agent's permission boundary

Even read-only queries use a request file under `~/.saggar/control`, so they
can fail in Codex's workspace sandbox. With execution consent, `quick`, `monitor`,
and `agent` launch separate processes with the Mac user's permissions; they do
not inherit the calling agent's sandbox or approval policy. Execution consent
is separate from workspace navigation and must be granted in the Mac UI.
Do not request broad access to the spool just to read status. If the channel
is blocked, use the agent's normal approval flow for the exact authorized
command or continue without Saggar.

Never use Saggar to route around a denied action, sandbox restriction,
Auto-review refusal, or pending user decision. Authorization to inspect a
terminal does not authorize launching commands or approving prompts.

## The verbs

| Command | What it does |
| --- | --- |
| `saggar init` | Declare the current folder as a project in `.saggar/project.json` |
| `saggar config [options]` | Read or update the current project's docs folder |
| `saggar config command [options] -- <command…>` | Add or update a project command, its view, and its pin |
| `saggar hooks [--json]` | Report each provider's presence hooks, the file they live in, and anything still blocking them |
| `saggar hooks install\|remove <provider>` | Install or remove saggar's hooks for one provider — only when the user asks |
| `saggar attention "why"` | Flag this terminal as needing the user, with the reason |
| `saggar attention --note "why"` | Same, as a look-when-free note that makes no claim |
| `saggar monitor <command…>` | Dock a monitor in this project running the command |
| `saggar monitor --discreet <command…>` | Same, docked at half weight until the user points at it |
| `saggar quick <command…>` | Run the command in a one-shot window beside the user's work |
| `saggar agent <provider[.model]> <task…>` | Run an independent agent in a new terminal beside this one |
| `saggar agent <provider> [options] -- <task…>` | Reuse or configure a terminal, including provider flags |
| `saggar agent claude --team -- <task…>` | Experimental: run Claude with native teammates shown as Saggar terminals |
| `saggar agent <provider> --queue -- <task…>` | Queue an agent until the project's active terminals, this one included, settle |
| `saggar agent codex --observe -- <task…>` | Link a Codex observer, told not to edit, to this session |
| `saggar agent codex --pair -- <task…>` | Link a Codex pairing partner to this session |
| `saggar message <text…>` | Message the linked pairing counterpart when it is idle |
| `saggar handoff` | Hand worktree ownership from the lead to the pairing partner |
| `saggar add <path>` | Register a project without focusing it or opening a terminal |
| `saggar close [--force] <id\|name>` | Close an idle terminal; `--force` closes a busy one |
| `saggar list [--json]` | List the project menu with a stable short id for each terminal |
| `saggar status [--json]` | Alias for `saggar list` |
| `saggar schema [command…]` | Print the clispec v0.2 command schema, optionally narrowed to one command subtree |
| `saggar capabilities` | Compatibility alias for `saggar schema` |
| `saggar read [id\|name] [--lines N\|--screen] [--json]` | Read bounded plain-text output from one terminal |
| `saggar session [id\|name] [--json]` | Print one terminal's name, presentation, and turbo setting |
| `saggar session [id\|name] set <name\|presentation\|turbo> <value>` | Change one of those settings; another terminal's name or presentation asks on the Mac first, and arming turbo needs the Settings ▸ CLI switch and never works on the calling terminal |
| `saggar settings [--json]` / `settings get\|set\|reset <key>` | Read or change app preferences; guarded keys still need the Mac app |
| `saggar explain [id\|name] [--json]` | Say why a terminal wears its status |
| `saggar wait [id\|name] [--until <status>…\|--match <text>] [--timeout N] [--json]` | Block until a terminal settles, reaches a status, or prints a line |
| `saggar focus [id\|name]` | Bring a terminal to the main pane (no target means this one) |
| `saggar approve` | Answer the permission prompt waiting in this terminal |
| `saggar resume set [options] -- <command…>` | Attach an untrusted custom resume binding to this terminal |
| `saggar resume show` | Inspect this terminal's custom resume binding and trust state |
| `saggar resume clear` | Remove this terminal's custom resume binding |
| `saggar turns [--json]` | List the agent turns recorded for this terminal, newest first |
| `saggar <path>` | Add or focus a repo on the project menu; the only verb that can launch saggar |
| `saggar help` | Print the verbs and flags |

`monitor` and `quick` take the rest of the line, so `saggar monitor npm run dev`
works without quoting. Quote when spacing matters. `attention` joins its words
the same way, and `--note` and `--discreet` may sit anywhere among them.

A call takes up to one status beat (~2s) to land. Workspace navigation (`focus`,
project opens, `close`, and teammate focus or close) then
follow **Settings ▸ CLI ▸ CLI workspace changes**: ask first by default,
allow after 10 visible, idle seconds, or allow immediately. The CLI waits for the
result; rejection, cancellation, or expiry returns exit code 1 without running
that action. Pending requests expire after two minutes. Don't retry a pending
request or change its permission setting.

Opening a project through the CLI shows its project page without launching a
shell or startup commands. Request command execution separately when needed.

Command launches, teammate input (`message`), `handoff`, and setting a resume command
require separate execution consent. The Mac offers **Allow once** or **Allow for this session**.
A session grant ends when the terminal closes or restarts, or Saggar quits;
the user can revoke it in **Settings ▸ CLI ▸ Command execution**. Navigation's
countdown and immediate settings never grant execution permission. A grant
does not authorize bypassing an agent's own restrictions. The file transport
still uses a self-reported session ID; these controls do not authenticate the
calling process or enforce sandbox isolation.

Explain a workspace change with an optional reason before the command:

```sh
saggar --reason "The test failure needs your input" focus "API tests"
```

Saggar attributes the reason to the requesting terminal and shows a completion
card even when permission is automatic. `Go back` restores navigation, not
commands that have already run. Reads, `saggar add`, `attention`, `approve`, `session`
without `set`, `session set` on your own terminal, and `resume show` or `resume clear` don't
prompt. `session set` on another terminal's name or presentation asks like `focus`.

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
directory, and environment in Settings ▸ Terminals. Do not ask the user to
approve it. An untrusted binding restores as an idle shell for manual use, and
a native provider session ID always takes precedence.

## Project configuration

`saggar config` prints the current folder's `.saggar/project.json`. It works
without a running Saggar app. Pass one or more options to update the manifest:

```sh
saggar config --docs handbook
saggar config --clear-docs
saggar config command --name "Example site" --monitor --browser --pin -- npm run start
```

The documentation folder must be a relative path inside the project. Run
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
shell `cd` does not retarget them. To work in another project, run `monitor` or
`quick` from one of that project's Saggar terminals. For an agent terminal,
pass an absolute `--cwd` instead:

An agent can start in a configured new terminal, or reuse an existing one:

```sh
saggar agent codex --title "API review" --cwd /repos/api -- fix the tests
saggar agent codex --session-id 7c310000 --cwd /repos/api -- review the diff
saggar agent claude --session-id 7c310000 -- what did that test actually assert?
saggar agent claude --flags '--permission-mode plan --add-dir ../shared' -- fix the tests
saggar close 7c310000
```

Use the short IDs from `saggar list`. `--session-id` does one of two things,
depending on what the terminal is doing:

- **An idle shell** takes the launch command line, as it always has.
- **A terminal already running the agent you named** takes the task as a
  prompt, the way the user would type it. Its context is the point — a session
  that has been reading this codebase for an hour is worth addressing rather
  than restarting. The agent queues the prompt if it's mid-turn.

Anything else running still refuses: a different provider, an agent saggar only
recognises, or an ordinary command. So do `--cwd` and `--flags` against a
running agent, which keeps the directory and flags it launched with, and a
terminal waiting on a permission prompt — answering prompts is `saggar
approve`'s job and its guards are there on purpose. `close` still refuses any
running child; stop its work rather than silently killing it.
`saggar close --force <id>` closes it anyway and ends whatever is running. Use
it only when the user asked for that terminal to go; it still asks on the Mac
like any close, and the audit log records it as a forced close.

`--flags` accepts one trusted shell fragment and inserts it after the selected
provider and model arguments, before the safely quoted task. Use it only for
provider options the user asked for; shell operators in the fragment are live.

## Observer and pairing sessions

Start an observer when another agent should review a Claude session without changing its work:

```sh
saggar agent codex --observe -- summarize progress and flag concrete concerns
```

The observer receives the lead session ID and reads bounded terminal output through `saggar read`.
Its prompt tells it not to edit files, run mutating commands, or commit. That is an instruction,
not a sandbox: the observer runs with the same Codex permissions as any other agent terminal.

Pairing adds an attributed message channel and a deliberate worktree handoff:

```sh
saggar agent codex --pair -- backseat this implementation
saggar message "Check the migration's rollback path."
saggar handoff
```

Messages have no target argument: Saggar resolves only the linked counterpart and refuses to
type unless it is idle. Observers cannot message. `handoff` is lead-only, moves both sessions
into the finishing phase, and tells Codex it may make the final changes, run checks, and commit.

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

It is off unless the user turned it on, so expect a refusal to be the normal
answer. The exit-1 message on stderr says why. The refusals worth knowing:

- **It is off.** "approving from the terminal is off in Settings ▸ CLI"
  is a settled answer, not a transient one, and it's the default. Do not retry,
  do not ask the user to change the setting, and do not look for a way around
  it — flag the terminal and stop:
  `saggar attention "blocked on a permission prompt"`.
- **A destructive command is on screen.** `rm -rf`, a force push, a raw disk
  write and their kin are held back whatever the setting says, the same line
  automatic approval stops on. The message names the shape and ends "this one is
  for the user". That prompt is the user's; leave it and say why.
- **"no prompt waiting".** Nothing on screen is asking.
- **"no answerable menu on screen".** The needs-you came from a bell, or from a
  wizard with no numbered choices, so there was nothing to type.

Habits:

- **Approve your own stop, not as a substitute for asking.** If the prompt is
  the CLI checking something you should have checked — a migration, a
  force-push, a file outside the repo — the honest move is `attention`, not
  `approve`.
- **Once, then look.** If the same prompt is still there on the next turn, your
  answer didn't land. Calling again won't help (a re-answer of the same prompt
  is refused by fingerprint); raise a flag instead.

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

## Delegating to a new agent terminal

Prefer your own subagent mechanism for work contained inside your current task:
it has direct context transfer, structured results, and a lifecycle you own.
Use `saggar agent` when the work should become a separate, user-owned process —
for example, it needs a different provider, should survive your context, or
deserves a terminal the user can inspect, redirect, or take over.

```sh
saggar agent codex "investigate the relay test failure; report findings only"
saggar agent claude.opus5 "review the authentication change; do not edit files"
```

The new terminal inherits this terminal's project and working directory. It
does not steal focus. Provider names are `claude`, `codex`, `antigravity`, `pi`,
`opencode`, and `copilot`; append a model alias such as `claude.opus5` or
`codex.gpt-6-astra` when the model matters. The task is shell-quoted by saggar, so pass the task as ordinary
words rather than building a provider command yourself.

To hand off work that should start only once the project's current work settles,
add `--queue`. The terminal appears now, and its agent launches when every
working or needs-you terminal in the project, yours included, has gone idle or
finished. If nothing is active, it starts straight away. The reply says which
happened:

```sh
saggar agent codex --queue -- "review the diff this session leaves behind"
```

This is delegation, not fire-and-forget process spawning. Before calling it:

- Give the child a bounded task, expected output, and whether it may edit.
- Avoid overlapping write scopes. Two agents editing the same files are still
  two processes racing, even though saggar shows both.
- Do not recursively spawn more agent terminals unless the user asked for that
  shape of work.
- Use `saggar list` to find the child and `saggar read <id>` for a bounded look
  at its output. There is no hidden result channel back to you.

## List and read terminals before interrupting

`saggar list` is what makes the channel worth having rather than merely
usable. It prints one line per terminal — status, project, name, agent — so you
can see whether the user is already dealing with something louder:

```sh
saggar list            # short id, status, project, name, and agent
saggar list --json     # the same menu as a snapshot object, for parsing
```

`saggar status` is an alias for compatibility. Use `list` in new calls.

The JSON is one snapshot object; its `sessions` array carries each terminal's
`id`, `projectName`, `projectPath`, `name`, `status` (a `kind` plus any
`exitCode`), `agent`, `promptWaiting`, `lastActivityAt`, git state, and a short
`transcriptTail` of its screen. If two other terminals already read needs-you,
a third flag is noise; finish, leave a `--note`, and let the user come to you.

Use the short id from `list` when another terminal's output would answer the
question. A name is accepted when it resolves uniquely; an ambiguous name is
refused with candidates instead of guessing.

```sh
saggar read                 # the calling terminal's last 80 lines
saggar read 7c310000        # another terminal by short id
saggar read 7c310000 --lines 200
saggar read 7c310000 --screen
saggar read 7c310000 --json
```

`read` returns plain text, defaults to an 80-line tail, and accepts at most 500
lines. `--screen` returns only the rows currently painted. Terminal output may
contain credentials or other private material, so ask for the smallest useful
window and do not repeat its contents unless the task requires it.

## A `saggar session <id>` line is someone pointing at a terminal

The user can drag a terminal from saggar's project menu into a chat, which
leaves a line like `saggar session 7c310000` in what they send you. It is a
reference, not an instruction: they are naming *that* terminal in the sentence
around it. Read the sentence for what they want, and use the short id with the
read-only verbs when you need the terminal itself:

```sh
saggar session 7c310000     # its name, presentation, and turbo setting
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
the `because` line when you do. The rule words are stable: `exit-code`,
`agent-call`, `hook-claim`, `prompt-cue`, `bell`, `watch-cue`, `notification`,
`command-running`, `background-work`, `turn-ended`, `turn-running`, `recent-output`, and `quiet`.

## What the CLI deliberately cannot do

Knowing the shape of the boundary saves you probing it:

- **No verb lowers a flag.** There is no ack, no dismiss, no way to mark another
  terminal's claim answered. An agent that could silence its own needs-you could
  report success it had not earned. `approve` is not an exception to this: it
  *answers* the prompt, and your next turn is the proof it landed.
- **A command speaks for the terminal it runs in.** `monitor`, `quick`,
  `attention`, and `approve` always act on your own session. `focus`, `close`,
  `session`, and the read-only `read`, `explain`, and `wait` may identify another
  terminal by id or unique name.
- **Session management stays narrow.** `saggar agent` may create one terminal
  for delegated work. Claude's explicit `--team` mode may create provider-owned
  child terminals; it is experimental, because it stands in for tmux with a shim
  and Claude Code has never promised the tmux calls it makes, so a Claude Code
  update can break it. `close` is limited to idle terminals unless the user
  asked for `--force`; there is no general `new` or `send` verb.
  `agent --session-id` is the one way to put words into another terminal, and
  it is narrow on purpose: an agent of the provider you named, no permission
  prompt on screen, and the words are its task.
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
