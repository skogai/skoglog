---
id: doc-3
title: handover-20260427
type: other
created_date: '2026-04-27 18:10'
---
# skoglog handover — 2026-04-27

## What we did this session

### Problem
- `backlog mcp start` and `bun run mcp` both failed to connect as MCP servers
- Root cause: system had Bun 1.3.11 but skoglog requires **Bun 1.2.23** (1.3.x has a websocket CPU regression, see `DEVELOPMENT.md`)

### What was done
1. Cloned `git@github.com:skogai/skoglog.git --branch=clean` to `~/.local/src/skoglog`
2. Pinned Bun to 1.2.23 via mise (`mise use bun@1.2.23` in that dir)
3. `bun install && bun run build` — produced `dist/backlog` at v1.44.0
4. Symlinked: `ln -sf ~/.local/src/skoglog/dist/backlog ~/.local/bin/backlog`
5. MCP server `backlog mcp start` worked with the binary in PATH

### State at end of session
- `~/.local/src/skoglog` was **deleted** (reverted to clean state)
- `~/.local/bin/backlog` symlink is **dangling** — needs to be recreated after next build
- The `clean` branch is the starting point for customization

### What we discovered about the customization goal
skogix made a proof-of-concept change in `src/constants/index.ts` to rename all
backlog naming conventions to skogai conventions:

| constant | original | skogai target |
|---|---|---|
| `BACKLOG` dir | `backlog` | `skoglog` |
| `HIDDEN_BACKLOG` dir | `.backlog` | `.skogai/tasks` |
| `CONFIG` file | `config.yml` | `skogai.yml` |
| `ROOT_CONFIG` file | `backlog.config.yml` | `skogai.backlog.yml` |
| `DEFAULT_STATUSES` | `["To Do", "In Progress", "Done"]` | `["todo", "doing", "done"]` |
| `FALLBACK_STATUS` | `"To Do"` | `"todo"` |

This is the right approach — fork the binary, own the naming.

---

## Next session plan

### Step 1 — Re-clone and build
```bash
git clone git@github.com:skogai/skoglog.git --branch=clean ~/.local/src/skoglog
cd ~/.local/src/skoglog
mise use bun@1.2.23
bun install
bun run build
ln -sf ~/.local/src/skoglog/dist/backlog ~/.local/bin/backlog
```

### Step 2 — Wire up original backlog to skogai project
- The skogai project (`~/skogai/`) currently has:
  - `backlog.config.yml` → symlink to `.skogai/config/backlog.yml`
  - `.skogai/` → symlink to `/home/skogix/.skogai/` (which is a git submodule)
  - `/home/skogix/.skogai/backlog.config.yml` exists at project root level too
- Tasks live in `.skogai/tasks/` (i.e. `/home/skogix/.skogai/tasks/`)
- Verify `backlog task list` works from `~/skogai/` against the unmodified binary first

### Step 3 — Customize constants
Edit `src/constants/index.ts` with the naming changes above, rebuild, verify
the MCP server still connects and tasks are found correctly.

### Step 4 — Config file rename
Once the binary looks for `skogai.backlog.yml`, update the symlink in `~/skogai/`:
```bash
ln -sf ./.skogai/config/backlog.yml ~/skogai/skogai.backlog.yml
rm ~/skogai/backlog.config.yml
```

### Watch out for
- The symlink chain: `~/skogai/backlog.config.yml` → `.skogai/config/backlog.yml` → real file in `/home/skogix/.skogai/config/`
- backlog's project root discovery may not follow symlinks for config lookup — test carefully after each change
- `auto_commit: true` in the config means backlog will git-commit to `.skogai/` (the submodule), not the parent repo — intentional but worth being aware of
