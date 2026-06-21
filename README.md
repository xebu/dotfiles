# dotfiles

Managed with [chezmoi](https://www.chezmoi.io/).

## Bootstrap (clean Mac)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
eval "$(/opt/homebrew/bin/brew shellenv)"
brew install chezmoi
chezmoi init --apply xebu
exec zsh
```

You'll be prompted for:
- **Git name / email** — your personal identity, used as the default in `~/.gitconfig`
- **`device_id`** — a short, stable label for this machine (e.g. `mba`, `m5max`, `homelab-node1`). Used to select the right SSH key per machine; pick something you won't need to change later.
- **`device_name`** — the machine's hostname (Norse naming, e.g. `odin`, `thor`, `sif`). Set automatically on personal devices via `run_onchange_before_00-set-hostname`.
- **`personal_device`** — `true` for your own machines, `false` for client/managed machines. Gates personal-only actions (hostname rename, personal app installs) so a client machine is never renamed or loaded with personal apps.

## After bootstrap (manual steps, not automated)

These are deliberately **not** part of the automated flow, since they involve secrets or per-client identity that should never be committed to this (public) repo:

1. **SSH keys** — generate or restore from a secure backup (Bitwarden):
```bash
   bin/setup-git-identity
```
   Run once per identity (personal GitHub, any work/client GitHub orgs).

2. **Per-client git identity** (if doing client work) — create an untracked override. See `docs/git-identity.md` for the full pattern and rationale.

## Repo structure

- `dot_*` / `dot_config/` — chezmoi-managed dotfiles, rendered to `~/.dotfile` or `~/.config/dotfile`
- `bin/` — interactive setup scripts, copied to `~/bin` and added to `$PATH`
- `run_once_*` / `run_onchange_*` — bootstrap scripts (package installs, macOS defaults, hostname), run automatically by chezmoi when their content changes

## Script run order

Bootstrap scripts run in phases — `before` → file application → `after` — alphabetically within each phase:

1. `run_onchange_before_00-set-hostname` — sets the machine hostname (personal devices only)
2. `run_once_before_01-rosetta` — installs Rosetta 2
3. *dotfiles applied*
4. `run_onchange_after_01-packages` — core Homebrew packages + casks
5. `run_onchange_after_02-macos-defaults` — macOS system preferences
6. `run_once_after_03-personal-casks` — personal apps (personal devices only)
