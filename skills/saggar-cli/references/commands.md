# Command catalog

| Command | What it does |
| --- | --- |
| `saggar init` | Declare the current folder as a project in `.saggar/project.json` |
| `saggar config [options]` | Read or update the current project's shared name and docs folder, and this Mac's icon and editor |
| `saggar config command [options] -- <command…>` | Add or update a project command, its view, and its pin |
| `saggar hooks [--json]` | Report each provider's presence hooks, the file they live in, and anything still blocking them |
| `saggar hooks install\|remove <provider>` | Install or remove saggar's hooks for one provider — only when the user asks |
| `saggar attention "why"` | Flag this terminal as needing the user, with the reason |
| `saggar attention --note "why"` | Same, as a look-when-free note that makes no claim |
| `saggar monitor <command…>` | Dock a monitor in this project running the command |
| `saggar monitor --discreet <command…>` | Same, docked at half weight until the user points at it |
| `saggar quick <command…>` | Run the command in a one-shot window beside the user's work |
| `saggar monitor\|quick --cwd <path> <command…>` | Same, starting in that directory |
| `saggar job add [--name <name>] "<cron>\|@once YYYY-MM-DD HH:MM" <command…>` | Ask the user to schedule a command in this terminal's project, or from home if it has no project, repeating or once; nothing is saved until they approve it on the Mac |
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
| `saggar inventory [--json]` | Compact session metadata with pending work and mechanical close eligibility |
| `saggar history [--project path] [--search text] [--since date] [--json]` | Search durable closed-session archives |
| `saggar restore <id>` / `restore --batch <id>` | Recover a session or whole cleanup paused; never starts its tasks |
| `saggar cleanup --preview` / `cleanup --apply <ids…>` | Preview protections, then close chosen completed sessions with one batch undo |
| `saggar cancel <id>` | Explicitly cancel pending work only when requested; preserves a recovery archive |
| `saggar close [--force] <id\|name>` | Close only sessions without pending work; `--force` overrides running processes |
| `saggar list [--json]` | List the project menu with a stable short id for each terminal |
| `saggar focused [--json]` | Read metadata for the terminal in the key window's main pane, without needing `SAGGAR_SESSION` |
| `saggar status [--json]` | Alias for `saggar list` |
| `saggar schema [command…]` | Print the clispec v0.2 command schema, optionally narrowed to one command subtree |
| `saggar capabilities` | Compatibility alias for `saggar schema` |
| `saggar read [id\|name] [--lines N\|--screen\|--conversation [--turns N]] [--json]` | Read bounded output, or an agent's recent requests and replies, from one terminal |
| `saggar inspect [id\|name] [--json]` | Inspect current work and resume readiness without returning a transcript |
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
