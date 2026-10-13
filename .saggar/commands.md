# Commands

This repo is the public mirror of the saggar-cli skill. The canonical copy lives in
saggar-desktop, checked out beside this repo; changes land there first, then sync here.

- Validate marketplace: `claude plugin validate .` — marketplace metadata and plugin entries #quick #safe #ci
- Validate plugin: `claude plugin validate .claude-plugin/plugin.json` — plugin manifest #quick #safe #ci
- Skill drift: `npm --prefix ../saggar-desktop run skill:drift` — diff this mirror against the canonical skill #quick #safe
- Sync from canonical: `rsync -a --delete ../saggar-desktop/Sources/saggar/Skills/saggar-cli/ skills/saggar-cli/` — replaces the tracked mirror, including removing files absent from the canonical copy; then review and commit #quick #writes

## Layout

- Claude: `claude` #primary #writes #interactive
- Git: `lazygit` #companion #icon:arrow.triangle.branch #writes #interactive
