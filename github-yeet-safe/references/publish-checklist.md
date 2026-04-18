# GitHub Publish Checklist

## Repo checks

```bash
git status --short
git branch --show-current
git remote -v
git log -1 --pretty=fuller
ls -la
```

## SSH checks

Prefer SSH first when the machine is already configured for GitHub:

```bash
test -f ~/.ssh/config && sed -n '1,160p' ~/.ssh/config
ssh -T git@github.com
```

If SSH works, keep the repository on an SSH remote when possible.

## Open-source baseline

Before a public publish, check whether the repo should include:

- `README.md`
- `LICENSE`
- `CONTRIBUTING.md`
- `.gitignore`

Add them when the repository is clearly meant to be a shareable project rather than a throwaway internal snapshot.

## Share-safety checks

Search for common leaks before a public push:

```bash
rg -n "token|secret|apikey|api_key|password|/Users/|\\.ssh|config.json|appSecret|userOpenId|userName" .
```

## gh availability

```bash
which gh
gh auth status
```

If `gh` is missing, prefer bootstrapping a temporary binary and continuing.

When browser interaction is needed, prefer a logged-in Chrome CDP session over a fresh browser profile.

## Auth recovery

Preferred sequence:

```bash
gh auth status
gh auth login --web --git-protocol https --skip-ssh-key
gh auth setup-git
gh auth status
```

If TTY or browser flow is awkward, use device flow output that prints the URL and code clearly, then re-check `gh auth status`.

## Public push hygiene

- Prefer a GitHub `noreply` email if the existing commit author reveals a machine-local email.
- Confirm visibility before creating the repo.
- Re-check `origin` before pushing.
- Re-check that public-facing docs do not expose local usernames or absolute home-directory paths.
- If the project is meant to feel complete, add the baseline project files before the first public push.
- If GitHub shows `Confirm access`, treat it as a required user confirmation step and verify the setting after the user completes it.

## Final publish

```bash
gh repo create <owner>/<repo> --public --source <local_repo> --remote origin --push
```
