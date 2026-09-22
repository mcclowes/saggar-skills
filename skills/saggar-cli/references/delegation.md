# Delegating and talking to other terminals

## Delegating to a new agent terminal

Prefer your own subagent mechanism for work contained inside your current task:
it has direct context transfer, structured results, and a lifecycle you own.
Use `saggar agent` when the work should become a separate, user-owned process —
for example, it needs a different provider, should survive your context, or
deserves a terminal the user can inspect, redirect, or take over.

```sh
saggar agent codex "investigate the relay test failure; report findings only"
saggar agent claude.opus55 "review the authentication change; do not edit files"
```

The new terminal inherits this terminal's project and working directory. It
does not steal focus. Provider names are `claude`, `codex`, `antigravity`, `pi`,
`opencode`, and `copilot`; append a model alias such as `claude.opus55` or
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

When the user asks you to queue sessions, they mean this flag. Pass `--queue` on
every launch, the first included: the sessions then run one at a time, each
waiting for the ones before it. Without it every agent starts at once and they
run side by side, however you describe them afterwards. Launch several without
`--queue` only when the user asked for parallel work and the write scopes don't
overlap.

This is delegation, not fire-and-forget process spawning. Before calling it:

- Give the child a bounded task, expected output, and whether it may edit.
- Avoid overlapping write scopes. Two agents editing the same files are still
  two processes racing, even though saggar shows both.
- Do not recursively spawn more agent terminals unless the user asked for that
  shape of work.
- Use `saggar list` to find the child and `saggar read <id>` for a bounded look
  at its output. There is no hidden result channel back to you.

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

A bare `saggar message` has no target: Saggar resolves only the linked counterpart and refuses to
type unless it is idle. Observers cannot message. `handoff` is lead-only, moves both sessions
into the finishing phase, and tells Codex it may make the final changes, run checks, and commit.

## Messaging a terminal

`--to` sends the same attributed line to any terminal, linked or not — the one verb that puts
words somewhere else:

```sh
saggar message --to 7c310000 the regression is in the retry backoff, not the parser
saggar message --to "API review" ready for you when you are
```

`--to` has to come first, because everything after the target is the message; a `--to` further
along is words you meant to send. The line arrives as `[<your terminal's name>] <message>`, so the
reader can tell an agent from its user.

What is running there is your problem, not Saggar's. An agent reads the line as a prompt and
queues it if it is mid-turn. A shell runs it as a command. A foreground process gets it on stdin.
Three things still refuse: a terminal that has gone, your own terminal (`attention` is how you
speak about yourself), and a terminal showing a permission prompt — answering those is `saggar
approve`'s job, and its guards exist on purpose.

Every `--to` asks for execution consent on the Mac, and the prompt shows the target and the exact
text. A pairing link is a standing agreement; naming a terminal is not.

Habits worth keeping:

- **Read before you speak.** `saggar read <id>` costs nothing and often answers the question you
  were about to ask.
- **Message the terminal, not the user.** A message lands in another agent's context; it does not
  raise a flag. If a person needs to act, that is `attention`.
- **Say what to look at.** The same rule `attention` follows. One line, the useful one.
