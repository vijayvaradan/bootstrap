# bootstrap

One-command setup for a new macOS or Ubuntu machine, using
[vijayvaradan/home](https://github.com/vijayvaradan/home) (private dotfiles).

```bash
export TOOLCHAINS=ALL       # or e.g. GO,JVM, or NONE - see below; required, no default
export MACHINE_OWNER=BOTH   # or PERSONAL - see below; required, no default
curl -fsSL https://raw.githubusercontent.com/vijayvaradan/bootstrap/main/bootstrap | sh
```

This is the only file hosted publicly, since it has to run before any GitHub
auth exists. It contains no secrets, just orchestration:

1. Checks `TOOLCHAINS` and `MACHINE_OWNER` are both set to valid values and
   aborts immediately (no prompt) if not - see below.
2. Installs `gh` (GitHub CLI) if missing.
3. Runs `gh auth login` if not already authenticated (a browser-based
   device-code flow).
4. Clones `vijayvaradan/home` via `gh`'s authenticated HTTPS transport, which
   works before any SSH key exists.
5. Hands off to that repo's own scripts: `bin/setup-github-ssh` (reads
   `MACHINE_OWNER` to decide whether to set up the work SSH identity),
   `bin/link-dotfiles` (symlinks everything into `$HOME`), then
   `setup-dev-tools-macos.sh` or `setup-dev-tools-ubuntu.sh` (reads
   `TOOLCHAINS`) - all three read their variable from the environment this
   script inherited it into.

Everything past step 4 lives in the private repo and is documented there
(`ONBOARDING.md`). This is the only copy of this script; it isn't tracked
in `vijayvaradan/home` at all, since nothing there depends on it being
present locally, it's purely the entrypoint that gets you to a cloned repo.

## Choosing toolchains

`TOOLCHAINS` is required: a case-insensitive, comma-separated list of the
toolchains to install. Valid tokens are `NODE GO JVM CPP FLUTTER`, plus the
sentinels `NONE` (install none of them) and `ALL` (install all of them).
Unset, empty, or containing an unrecognized token, and the script aborts
before doing anything else - no interactive prompt, by design. Rust, like
Python/uv, installs unconditionally - it isn't a `TOOLCHAINS` token.

`SKIP_NEOVIM=1` is separate and still opt-out (see the private repo's
`ONBOARDING.md`): it controls only whether Neovim itself installs, unrelated
to which language toolchains are requested via `TOOLCHAINS`.

## Choosing SSH identities

`MACHINE_OWNER` is required: `PERSONAL` (personal SSH identity only) or `BOTH`
(personal + work), case-insensitive. There's no `WORK`-only value - the
personal key is what `vijayvaradan/home`'s own remote needs, so every machine
gets it regardless.

`export` both variables first, don't prefix the `curl` command with them - a
prefix only applies to `curl`, never to `sh` on the other side of the pipe,
so it silently never reaches the script:

```bash
export TOOLCHAINS=GO,JVM
export MACHINE_OWNER=PERSONAL
curl -fsSL https://raw.githubusercontent.com/vijayvaradan/bootstrap/main/bootstrap | sh
```

or prefix `sh` itself, on the receiving end of the pipe:

```bash
curl -fsSL https://raw.githubusercontent.com/vijayvaradan/bootstrap/main/bootstrap | TOOLCHAINS=GO,JVM MACHINE_OWNER=PERSONAL sh
```
