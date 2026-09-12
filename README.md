# bootstrap

One-command setup for a new macOS or Ubuntu machine, using
[vijayvaradan/home](https://github.com/vijayvaradan/home) (private dotfiles).

```bash
curl -fsSL https://raw.githubusercontent.com/vijayvaradan/bootstrap/main/bootstrap | sh
```

This is the only file hosted publicly, since it has to run before any GitHub
auth exists. It contains no secrets, just orchestration:

1. Installs `gh` (GitHub CLI) if missing.
2. Runs `gh auth login` if not already authenticated (a browser-based
   device-code flow).
3. Clones `vijayvaradan/home` via `gh`'s authenticated HTTPS transport, which
   works before any SSH key exists.
4. Hands off to that repo's own scripts: `bin/setup-github-ssh` (personal/work
   SSH identity split), `bin/link-dotfiles` (symlinks everything into `$HOME`),
   then `setup-dev-tools-macos.sh` or `setup-dev-tools-ubuntu.sh`.

Everything past step 3 lives in the private repo and is documented there
(`ONBOARDING.md`). This is the only copy of this script; it isn't tracked
in `vijayvaradan/home` at all, since nothing there depends on it being
present locally, it's purely the entrypoint that gets you to a cloned repo.
