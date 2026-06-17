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
- `run_once_*` / `run_onchange_*` — bootstrap scripts (package installs, macOS defaults), run automatically by chezmoi when their content changes
