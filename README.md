# bootstrap

One-command setup for a new macOS or Ubuntu machine, using
[vijayvaradan/home](https://github.com/vijayvaradan/home) (private dotfiles).

```bash
export TOOLCHAINS=ALL      # or e.g. RUST,GO, or NONE - see below; required, no default
export MACHINE_OWNER=WMG   # or VIJAY - see below; required, no default
curl -fsSL https://raw.githubusercontent.com/vijayvaradan/bootstrap/main/bootstrap | sh
```

This is the only file hosted publicly, since it has to run before any GitHub
auth exists. It contains no secrets, just orchestration:

1. Checks `TOOLCHAINS` and `MACHINE_OWNER` are both set to valid values and
   aborts immediately (no prompt) if not - see below.
2. Installs `gh` (GitHub CLI) if missing.
3. Runs `gh auth login` if not already authenticated (a browser-based
   device-code flow), storing the token in `~/.config/gh/hosts.yml`
   (`--insecure-storage`) rather than the keyring.
4. Clones `vijayvaradan/home` via `gh`'s authenticated HTTPS transport, which
   works before any SSH key exists. Until the dotfiles are linked, git's global
   config is a throwaway file, so nothing creates a real `~/.gitconfig`.
5. Hands off to that repo's own scripts, in this order: `bin/link-dotfiles`
   (symlinks everything into `$HOME`), `bin/setup-github-ssh` (reads
   `MACHINE_OWNER`, `VIJAY` or `WMG`; sets up the SSH keys and identities),
   then `setup-dev-tools-macos.sh` or `setup-dev-tools-ubuntu.sh` (reads
   `TOOLCHAINS`). If `setup-github-ssh` reports something unresolved (e.g. an
   Azure DevOps key not yet added on the website), the toolchain install still
   runs and the script ends by telling you to re-run `setup-github-ssh`.

Everything past step 4 lives in the private repo and is documented there
(`ONBOARDING.md`). This is the only copy of this script; it isn't tracked
in `vijayvaradan/home` at all, since nothing there depends on it being
present locally, it's purely the entrypoint that gets you to a cloned repo.

## Choosing toolchains

`TOOLCHAINS` is required: a case-insensitive, comma-separated list of the
toolchains to install. Valid tokens are `RUST NODE GO JVM CPP FLUTTER`, plus
the sentinels `NONE` (install none of them) and `ALL` (install all of them).
Unset, empty, or containing an unrecognized token, and the script aborts
before doing anything else - no interactive prompt, by design.

`SKIP_NEOVIM=1` is separate and still opt-out (see the private repo's
`ONBOARDING.md`): it controls only whether Neovim itself installs, unrelated
to which language toolchains are requested via `TOOLCHAINS`.

## Choosing SSH identities

`MACHINE_OWNER` is required: `VIJAY` (personal SSH identity only) or `WMG`
(personal + work), case-insensitive. There's no `WMG`-only value - the
personal key is what `vijayvaradan/home`'s own remote needs, so every machine
gets it regardless.

`export` both variables first, don't prefix the `curl` command with them - a
prefix only applies to `curl`, never to `sh` on the other side of the pipe,
so it silently never reaches the script:

```bash
export TOOLCHAINS=RUST,GO
export MACHINE_OWNER=VIJAY
curl -fsSL https://raw.githubusercontent.com/vijayvaradan/bootstrap/main/bootstrap | sh
```

or prefix `sh` itself, on the receiving end of the pipe:

```bash
curl -fsSL https://raw.githubusercontent.com/vijayvaradan/bootstrap/main/bootstrap | TOOLCHAINS=RUST,GO MACHINE_OWNER=VIJAY sh
```
