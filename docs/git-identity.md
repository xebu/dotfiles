# Git identity: personal default, per-client overrides

## Goal

- Personal identity is the default everywhere.
- Client repos automatically commit as the right client identity.
- No client names, emails or hostnames ever appear in this public repo.

## How it fits together

```
~/.gitconfig                      tracked (chezmoi): personal name/email,
  [include] path = ~/.gitconfig-local   then includes the local file
~/.gitconfig-local                untracked, per machine: one includeIf per client
  [includeIf "gitdir:~/code/work/acme/"]
    path = ~/code/work/acme/.gitconfig-local
~/code/work/acme/.gitconfig-local untracked: the client's [user] email/name
```

Git reads `~/.gitconfig`, which includes `~/.gitconfig-local`. For any repo
under `~/code/work/acme/`, the `includeIf` matches and the client file
overrides `user.email` (and optionally `user.name`). Everywhere else, the
personal identity applies.

The `includeIf` lives in `~/.gitconfig-local`, not the tracked
`~/.gitconfig`, precisely so the client folder name never reaches GitHub.

## Adding a client

```bash
mkdir -p ~/code/work/acme && cd ~/code/work/acme
setup-work-identity          # prompts for email (and optional name)
cd some-repo && git config user.email   # verify
```

Re-running it for an existing client rewrites that client's identity file
and leaves the `includeIf` alone.

## SSH keys per identity

`setup-git-identity` creates a key and an SSH host alias per identity, e.g.
`github-acmeuser`, so a client GitHub account uses its own key:

```bash
git clone github-acmeuser:acme/repo.git
git remote set-url origin github-acmeuser:acme/repo.git
```

Non-primary keys are named `id_ed25519_<username>_<device_id>`, using the
stable `device_id` from chezmoi rather than the hostname.

## Gotchas

- `gitdir:` patterns need the trailing `/` to match everything below.
- `includeIf "gitdir:..."` only applies inside a git repo. Running
  `git config user.email` in `~/code/work/acme/` itself (not a repo) shows
  the personal email; that's expected.
- Verify which file set a value with `git config --show-origin user.email`.
