# dotfiles

macOS dotfiles and bootstrap, managed with [chezmoi](https://www.chezmoi.io/).

## Bootstrap (clean Mac)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
eval "$(/opt/homebrew/bin/brew shellenv)"
brew install chezmoi
chezmoi init --apply xebu
exec zsh
```

You'll be prompted once per machine (answers are cached, untracked, in
`~/.config/chezmoi/chezmoi.toml`):

- **Git name / email**: your personal identity, the default in `~/.gitconfig`.
- **`device_id`**: a short, stable label for this machine (e.g. `mba`, `m5max`). Used in SSH key filenames; pick something you won't need to change.
- **`device_name`**: the hostname (Norse naming, e.g. `odin`, `thor`, `sif`). Set automatically on personal devices.
- **`personal_device`**: `true` for your own machines, `false` for client/managed ones. Gates the hostname rename and the `personal` package group, so a client machine is never renamed or loaded with personal apps.

## Day to day

```bash
chezmoi update -v        # pull + apply (on any machine)
chezmoi diff             # what would change
chezmoi edit --apply ~/.zshrc
chezmoi cd               # into the repo
```

## Updating an existing Mac to the `home/` layout (one-time)

The source moved into `home/` (see `.chezmoiroot`). On a Mac that was set
up before that change:

```bash
chezmoi update -v   # 1. pulls the new layout; applies nothing (guarded by the root .chezmoiignore)
chezmoi diff        # 2. review
chezmoi apply -v    # 3. applies with the new layout
```

Step 1 may print `warning: config file template has changed, run chezmoi
init`. That's expected for this one run (it's still reading the old layout);
ignore it. Your cached answers are unchanged and nothing needs re-prompting.

Step 3 re-runs these once, because chezmoi tracks `run_onchange_` scripts by
name and they moved: hostname (no-op if already set), packages
(`brew bundle`), mise, macOS defaults (Dock/Finder restart). After that,
updates behave normally.

Afterwards, delete `~/README.md` if it's the copy of this file that the old
layout installed; chezmoi no longer manages it. Once every Mac is migrated,
the root `.chezmoiignore` can be deleted.

## Packages

All Homebrew packages are declared in
[`home/.chezmoidata/packages.yaml`](home/.chezmoidata/packages.yaml), in
groups: `core`, `cloud`, `dev`, `productivity` (every Mac) and `personal`
(only when `personal_device = true`).

They are rendered to `~/.Brewfile` and installed with
`brew bundle --no-upgrade` (installs what's missing; doesn't upgrade).
Editing the YAML re-runs the install on the next `chezmoi apply`.

```bash
brew bundle check --global --verbose --no-upgrade   # declared but not installed
brew bundle cleanup --global           # installed but not declared (dry run)
brew bundle upgrade --global           # deliberate upgrade
```

If an app was installed by hand rather than via Homebrew, `brew bundle`
reports it as failed. Adopt it with `brew install --cask --adopt <name>`.

Languages (Go, Node, Python, Ruby) are managed by mise:
[`home/dot_config/mise/config.toml`](home/dot_config/mise/config.toml).

Tools you'd want to keep up to date are deliberately not in the bootstrap
(it would only ever install them once). Install them by hand; re-running the
same command updates them.

**Personal Macs only.** bootdev must not be installed on client machines;
`home/.chezmoiremove` deletes `~/go/bin/bootdev` on any Mac with
`personal_device = false`.

```bash
go install github.com/bootdotdev/bootdev@latest   # lands in ~/go/bin, on PATH
```

## After bootstrap (manual steps, not automated)

These are deliberately **not** automated, since they involve secrets or
per-client identity that should never be committed to this (public) repo:

1. **SSH keys**: generate, or restore from Bitwarden, then register:
   ```bash
   setup-git-identity
   ```
   Run once per identity (personal GitHub, any client orgs).
2. **Per-client git identity** (client work only): from inside
   `~/code/work/<client>/`, run `setup-work-identity`. See
   [docs/git-identity.md](docs/git-identity.md).

## Repo structure

```
.chezmoiroot          # "home": chezmoi's source lives in home/
.chezmoiignore        # transition guard for the home/ move (see above)
README.md, docs/      # documentation; never copied into $HOME
home/
  .chezmoi.toml.tmpl  # per-machine prompts
  .chezmoidata/       # packages.yaml
  .chezmoitemplates/  # Brewfile (shared by dot_Brewfile and 10-packages)
  .chezmoiscripts/    # bootstrap scripts (no ~/ directory created)
  bin/                # setup scripts -> ~/bin (on PATH)
  dot_*, dot_config/  # dotfiles -> ~/.*, ~/.config/*
```

## Script run order

`before` scripts, then files are applied, then `after` scripts;
alphabetical within each phase:

| Script | Runs | Scope |
|---|---|---|
| `run_onchange_before_00-set-hostname` | when `device_name` changes | personal Macs |
| `run_once_before_01-rosetta` | once | Apple Silicon |
| *dotfiles applied (incl. `~/.Brewfile`)* | | |
| `run_onchange_after_10-packages` | when the rendered Brewfile changes | all |
| `run_onchange_after_20-mise` | when the mise config changes | all |
| `run_onchange_after_30-macos-defaults` | when the script changes | all Macs |
