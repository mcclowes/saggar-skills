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
  asked for `--force`; there is no general `new` verb. `message --to` is the one
  way to put words into another terminal: attributed, consented to on the Mac,
  audited, and never into a waiting permission prompt.
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

## The verbs

| Command | What it does |
| --- | --- |
| `saggar init` | Declare the current folder as a project in `.saggar/project.json` |
| `saggar config [options]` | Read or update the current project's shared name and docs folder |
| `saggar config command [options] -- <command…>` | Add or update a project command, its view, and its pin |
| `saggar hooks [--json]` | Report each provider's presence hooks, the file they live in, and anything still blocking them |
| `saggar hooks install\|remove <provider>` | Install or remove saggar's hooks for one provider — only when the user asks |
| `saggar attention "why"` | Flag this terminal as needing the user, with the reason |
| `saggar attention --note "why"` | Same, as a look-when-free note that makes no claim |
| `saggar monitor <command…>` | Dock a monitor in this project running the command |
| `saggar monitor --discreet <command…>` | Same, docked at half weight until the user points at it |
| `saggar quick <command…>` | Run the command in a one-shot window beside the user's work |
| `saggar monitor\|quick --cwd <path> <command…>` | Same, starting in that directory |
| `saggar agent <provider[.model]> <task…>` | Run an independent agent in a new terminal beside this one |
| `saggar agent <provider> [options] -- <task…>` | Reuse or configure a terminal, including provider flags |
| `saggar agent claude --team -- <task…>` | Experimental: run Claude with native teammates shown as Saggar terminals |
| `saggar agent <provider> --queue -- <task…>` | Queue an agent until the project's active terminals, this one included, settle |
| `saggar agent codex --observe -- <task…>` | Link a Codex observer, told not to edit, to this session |
| `saggar agent codex --pair -- <task…>` | Link a Codex pairing partner to this session |
| `saggar message <text…>` | Message the linked pairing counterpart when it is idle |
| `saggar message --to <id\|name> <text…>` | Send an attributed line to any terminal |
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
`--cwd <path>` has to come before the command; later on the line, it's the
command's own flag.

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

## Reference, read when you need it

The rules above are the ones to keep in mind. The detail for each verb lives
beside this file; read the one that matches what you are about to do:

- [references/running-commands.md](references/running-commands.md): Read this before `saggar monitor`, `saggar quick`, `saggar add`, or reusing and closing terminals with `saggar agent` and `saggar close`.
- [references/delegation.md](references/delegation.md): Read this before `saggar agent`, an observer or pairing session, `saggar message`, or `saggar handoff`.
- [references/observing.md](references/observing.md): Read this for `saggar wait`, `saggar explain`, `saggar turns`, or a `saggar session <id>` line in a message.
- [references/project-setup.md](references/project-setup.md): Read this for `saggar config`, `saggar hooks`, or `saggar resume`.

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
