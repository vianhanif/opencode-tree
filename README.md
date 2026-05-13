# opencode-tree

**Temporary isolated git worktrees for opencode + tmux sessions with auto-cleanup.**

Stop switching branches, stashing changes, and losing context. `opencode-tree` creates a clean git worktree, drops you into a tmux session with opencode ready, and removes everything when you're done.

## How it works

```
opencode-tree .  feature/my-feat
                        │
     ┌──────────────────┴──────────────────┐
     │                                      │
  git worktree add                     tmux session
  (isolated checkout)           ┌──────────┴──────────┐
                               │                      │
                          Window 1: code          Window 2: shell
                          (opencode)              (terminal)
     │                                      │
     └──────────────────┬──────────────────┘
                        │
            User exits → cleanup runs
         git worktree remove (+ branch cleanup)
```

## Quick start

```bash
# From inside any git repo — creates temp branch, launches opencode + tmux
opencode-tree .

# Same, but with a specific branch
opencode-tree .  feature/my-feature

# Point to a repo elsewhere
opencode-tree ~/projects/my-app
opencode-tree ~/projects/my-app  feature/x

# Reattach to a running session
opencode-tree --resume .

# Clean stale worktrees (e.g. after a crash)
opencode-tree --prune

# Without tmux (opencode in current terminal)
opencode-tree --no-tmux .
```

## What happens

1. **Branch** — if you specify one, it's used (created from the repo's default branch if needed). If omitted, a temp branch `openworktree/<ts>-<random>` is created from the default branch.

2. **Worktree** — `git worktree add` creates an isolated checkout in `~/.local/share/opencode-tree/<repo>/<branch>/`.

3. **tmux** — a session is created with two windows:
   - `code` — opencode TUI
   - `shell` — a terminal

4. **Cleanup** — when the session ends (all windows closed, or `tmux kill-session`), the script:
   - Detects uncommitted changes and prompts you
   - Deletes temp branches (asks about unpushed commits)
   - Removes the worktree
   - Runs `git worktree prune`

## Options

| Flag | Description |
|------|-------------|
| `-f, --force` | Skip all prompts during cleanup |
| `--no-tmux` | Run opencode in current terminal (no tmux session) |
| `-r, --resume` | Reattach to an existing session |
| `--prune` | Scan and remove stale worktrees, then exit |
| `-b, --base <branch>` | Base branch for temp branches (auto-detected by default) |
| `--version` | Show version |
| `-h, --help` | Show help |

## Environment

| Variable | Description |
|----------|-------------|
| `OPENCODE_TREE_DIRS` | Colon-separated directories to search for repos (e.g. `~/code:~/work`) |

When you give a name instead of a path, each directory in `OPENCODE_TREE_DIRS` is checked for a matching subdirectory.

## Requirements

- Python 3.6+
- git 2.5+ (first version with worktree support)
- tmux 3.0+ (optional — use `--no-tmux` without it)
- opencode (obviously)

## Install

```bash
# Clone
git clone https://github.com/vianhanif/opencode-tree.git
cd opencode-tree

# Add to PATH (or symlink somewhere on your PATH)
ln -s "$PWD/opencode-tree" ~/.local/bin/
```

## Uninstall

```bash
# Remove symlink / script
rm ~/.local/bin/opencode-tree

# Remove all worktree data (only if you want to wipe everything)
rm -rf ~/.local/share/opencode-tree
```

## License

MIT
