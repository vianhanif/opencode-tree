# opencode-tree

**Isolated opencode environments via git worktree + tmux.**

Stop switching branches, stashing changes, and losing context. `opencode-tree` creates a clean git worktree, drops you into a tmux session with opencode ready, and removes everything when you're done.

For non-git directories, it skips git operations entirely and just launches opencode in tmux for that directory.

## How it works

**Git mode:**
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

**Plain mode** (non-git directory):
```
opencode-tree /some/folder
                        │
                        ▼
                   tmux session
           ┌──────────┴──────────┐
           │                      │
      Window 1: code          Window 2: shell
      (opencode)              (terminal)
                        │
                        ▼
                  User exits → done
               (no git cleanup needed)
```

## Quick start

```bash
# From inside any git repo — creates temp branch, launches opencode + tmux
opencode-tree .

# With a specific branch
opencode-tree .  feature/my-feature

# Point to a repo elsewhere
opencode-tree ~/projects/my-app
opencode-tree ~/projects/my-app  feature/x

# Plain directory (non-git) — just launches tmux + opencode
opencode-tree /some/folder

opencode-tree --resume .             # Reattach to session
opencode-tree --remove ot-repo-branch  # Kill session + remove worktree
opencode-tree --prune                # Clean stale worktrees
opencode-tree --no-tmux .            # Without tmux
```

## What happens

### Git mode

1. **Branch** — if you specify one, it's used (created from the repo's default branch if needed). If omitted, a temp branch `openworktree/<ts>-<random>` is created.
2. **Worktree** — `git worktree add` creates an isolated checkout.
3. **tmux** — session with `code` + `shell` windows.
4. **Cleanup** — destroys dirty state, deletes temp branch, removes worktree.

### Plain mode (non-git)

1. No git operations — the directory is used as-is.
2. **tmux** — same session layout, just pointed at your directory.
3. **Cleanup** — prints "Session ended." No git cleanup needed.

## Options

| Flag | Description |
|------|-------------|
| `-f, --force` | Skip all prompts during cleanup (now default behavior for dirty worktrees) |
| `--no-tmux` | Run opencode in current terminal (no tmux session) |
| `-r, --resume` | Reattach to an existing session |
| `-R, --remove` | Kill a tmux session and auto-clean its worktree |
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
- git 2.5+ (optional — plain mode works without it)
- tmux 3.0+ (optional — use `--no-tmux` without it)
- opencode

## Install

```bash
git clone https://github.com/vianhanif/opencode-tree.git
cd opencode-tree
ln -s "$PWD/opencode-tree" ~/.local/bin/
```

## Uninstall

```bash
rm ~/.local/bin/opencode-tree
rm -rf ~/.local/share/opencode-tree
```

## License

MIT
