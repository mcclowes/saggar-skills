# Commands

This repo is the public mirror of the saggar-cli skill. The canonical copy lives in
saggar-desktop, checked out beside this repo; changes land there first, then sync here.

- Validate plugin: `claude plugin validate .` — plugin and marketplace manifests, and the skill #quick
- Skill drift: `npm --prefix ../saggar-desktop run skill:drift` — diff this mirror against the canonical skill #quick
- Sync from canonical: `rsync -a --delete ../saggar-desktop/Sources/saggar/Skills/saggar-cli/ skills/saggar-cli/` — then review and commit #quick

## Layout

- Claude: `claude` #primary
- Git: `lazygit` #companion #icon:arrow.triangle.branch
