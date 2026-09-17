# bootstrap

One-command setup for a new macOS or Ubuntu machine, using
[vijayvaradan/home](https://github.com/vijayvaradan/home) (private dotfiles).

```bash
export TOOLCHAINS=ALL   # or e.g. RUST,GO, or NONE - see below; required, no default
curl -fsSL https://raw.githubusercontent.com/vijayvaradan/bootstrap/main/bootstrap | sh
```

This is the only file hosted publicly, since it has to run before any GitHub
auth exists. It contains no secrets, just orchestration:

1. Checks `TOOLCHAINS` is set to a valid value and aborts immediately (no
   prompt) if not - see below.
2. Installs `gh` (GitHub CLI) if missing.
3. Runs `gh auth login` if not already authenticated (a browser-based
   device-code flow).
4. Clones `vijayvaradan/home` via `gh`'s authenticated HTTPS transport, which
   works before any SSH key exists.
5. Hands off to that repo's own scripts: `bin/setup-github-ssh` (personal/work
   SSH identity split), `bin/link-dotfiles` (symlinks everything into `$HOME`),
   then `setup-dev-tools-macos.sh` or `setup-dev-tools-ubuntu.sh`, both of
   which read `TOOLCHAINS` from the environment this script inherited it into.

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

`export` `TOOLCHAINS` first, don't prefix the `curl` command with it - a
prefix only applies to `curl`, never to `sh` on the other side of the pipe,
so it silently never reaches the script:

```bash
export TOOLCHAINS=RUST,GO
curl -fsSL https://raw.githubusercontent.com/vijayvaradan/bootstrap/main/bootstrap | sh
```

or prefix `sh` itself, on the receiving end of the pipe:

```bash
curl -fsSL https://raw.githubusercontent.com/vijayvaradan/bootstrap/main/bootstrap | TOOLCHAINS=RUST,GO sh
```
