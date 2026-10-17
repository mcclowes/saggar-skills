# Commands

This repo is the public mirror of the saggar-cli skill. The canonical copy lives in
saggar-desktop, checked out beside this repo; changes land there first, then sync here.

- Validate marketplace: `claude plugin validate --strict .claude-plugin/marketplace.json` — reject manifest errors and warnings with Claude Code 2.1.302 #quick #safe #ci
- Validate plugin: `claude plugin validate --strict .claude-plugin/plugin.json` — reject manifest errors and warnings with Claude Code 2.1.302 #quick #safe #ci
- Skill drift: `npm --prefix ../saggar-desktop run skill:drift` — compare the mirror and canonical skill; also checks the installed personal copy #quick #safe
- Sync from canonical: `rsync -a --delete ../saggar-desktop/Sources/saggar/Skills/saggar-cli/ skills/saggar-cli/` — replaces the mirror, including deletions; then review and commit #quick #destructive

## Layout

- Claude: `claude` #primary #destructive #interactive
- Git: `lazygit` #companion #destructive #interactive #icon:arrow.triangle.branch
