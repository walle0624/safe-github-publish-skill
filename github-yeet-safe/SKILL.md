---
name: github-yeet-safe
description: Use this skill when the user wants to publish a local project to GitHub, create a new GitHub repository, push code, or recover from missing gh/git auth during publishing. Prefer a reliable end-to-end flow with privacy checks, auth recovery, and git credential setup before pushing.
---

# Safe GitHub Publish

Use this skill when the user wants to publish a local repo to GitHub, especially if the environment may be missing `gh`, missing GitHub login state, or missing Git credential bridging.

This skill exists to avoid repeating the same detours:

- assuming `gh` is installed when it is not
- assuming GitHub login already exists when it does not
- assuming `git push` will work over HTTPS without `gh auth setup-git`
- pushing public code before checking author identity or share-safety

This skill should also help the repo reach a reasonable open-source baseline before publishing, not merely upload files.

## Core Rule

Treat GitHub publishing as a workflow, not a single command.

Before pushing, verify:

1. the local repository is the intended one
2. the content is safe to publish
3. GitHub auth is available
4. Git credential bridging is configured
5. the remote repository exists or can be created
6. the repository has the minimum docs and metadata appropriate for sharing

## Workflow

1. Inspect the local repo state with `git status`, current branch, current remotes, and latest commit.
2. If the repo will become public, check for obvious private data:
   - absolute home-directory paths
   - local usernames
   - tokens, secrets, auth files, account metadata
3. Check whether `gh` exists with `which gh`.
4. If `gh` is missing, first check whether a GitHub connector or browser-backed GitHub session is available:
   - if available, continue the publishing workflow through that path instead of blocking on `gh`
   - if not available, install or bootstrap `gh`
   - prefer an existing system install if available
   - otherwise download a temporary release binary locally and use that
5. Check GitHub login with `gh auth status`.
6. If not logged in, start auth recovery:
   - prefer browser or device-flow login
   - if device flow is used, clearly surface the URL and one-time code to the user
   - wait for the login to complete before continuing
7. After login, always run `gh auth setup-git` before pushing over HTTPS.
8. If the repo is intended to be shareable, verify git author identity is safe:
   - avoid pushing machine-local emails or hostnames when a public-safe identity is better
   - prefer a GitHub `noreply` address when appropriate
9. If the repo is meant to be a polished public project, check for minimum open-source project files:
   - `README.md`
   - `LICENSE`
   - `CONTRIBUTING.md` when contributions are welcome
   - `.gitignore`
10. If missing, add the missing project files before publishing when the user intent clearly implies a shareable/open-source repo.
11. Create the GitHub repo if it does not exist.
12. If command-line push is available and authenticated, set or update `origin` and push.
13. If command-line push is not available but the GitHub connector can create files or commits directly, use the connector path to publish the repository contents.
14. Confirm the final repo URL.

## Open Source Readiness

When the user is publishing something intended for public sharing, do not stop at a bare git push if the repository is obviously missing foundational project files.

Prefer this baseline:

- `README.md`:
  explain what the project is, why it exists, how to install or use it, and any important privacy or compatibility notes
- `LICENSE`:
  add one when the user is publishing a project for reuse; MIT is a reasonable default if the user has not specified otherwise and the repo is their own work
- `CONTRIBUTING.md`:
  add a lightweight guide when outside contributions are likely or explicitly welcome
- `.gitignore`:
  keep local junk and machine-specific files out of the repo

Optional but useful when the repository is meant to look more complete:

- issue templates
- pull request template
- changelog
- screenshots or examples

Use judgment. A tiny internal helper script does not need the same packaging as a polished public project, but a shareable GitHub repo usually benefits from at least the baseline files above.

## gh Recovery Path

Prefer this order:

1. existing `gh` on `PATH`
2. GitHub connector or browser-backed GitHub session that can create repos and upload files
3. temporary locally downloaded `gh` binary
4. only after that, ask the user to install `gh` manually

If you download a temporary `gh`, keep it local to the workspace or a temp folder and continue the workflow instead of stopping after install.

If the GitHub connector path is available, do not insist on `gh` just for the sake of symmetry. Finish the publishing task with the tools that exist.

## Connector Recovery Path

When `gh` is missing or unusable, but GitHub connector tools or a logged-in browser session are available:

1. create the repository through the connector or GitHub web UI
2. add the baseline repo files directly through the connector when needed
3. upload the project contents through connector-backed file creation or commit APIs
4. verify the repository contents from GitHub after upload

Use this path when it is faster and safer than trying to recover local `gh`.

Do not stop at "repo created" if the user asked to upload the project. Repo creation alone is not publication.

## Auth Recovery Path

When `gh auth status` fails:

- do not give up after one attempt
- avoid assuming the user already completed browser auth unless `gh auth status` confirms it
- if interactive TTY prompts are flaky, use a mode that prints a device URL and one-time code clearly
- after the user says they completed auth, re-check with `gh auth status`

Do not move on to repo creation or `git push` until `gh auth status` succeeds.

## Push Safety

For public pushes, review these before publishing:

- commit author name/email
- repo visibility
- tracked files that may expose private material
- documentation that mentions local machine paths or account-specific identifiers
- whether the repository still needs baseline public-project files

If the current commit author exposes a machine-local identity and the repo is meant for public sharing, update the local repo config and amend the commit author before pushing.

## Command Patterns

Common checks:

```bash
git status --short
git branch --show-current
git remote -v
git log -1 --pretty=fuller
which gh
gh auth status
gh auth setup-git
ls -la
```

Common create-and-push flow:

```bash
gh repo create <owner>/<repo> --public --source <local_repo> --remote origin --push
```

If the repo already exists, use normal git remote and push commands after auth is healthy.

If normal git push is blocked but the GitHub connector can write files, continue with the connector instead of failing early.

## Repo Creation Expectations

When the user asks to "publish to GitHub" and the project is clearly intended to be shared, default to creating:

- a sensibly named repository
- the visibility the user asked for, or ask only if the visibility has non-obvious consequences
- a first commit whose author identity is safe for publication
- a repo that is not missing obvious public-facing essentials

Do not treat repo creation as complete if the result is technically online but still missing the files the user likely expects from a real open-source project.

## Guardrails

- Do not print tokens or secrets.
- Do not assume browser auth succeeded until verified.
- Do not push a public repo with obviously private content if it can be cleaned first.
- Do not ask the user to do local investigation that can be done directly with commands.
- Prefer continuing the workflow yourself instead of stopping at intermediate setup milestones.
- Do not treat missing `gh` as a dead end when GitHub connector tools or a logged-in browser path can still complete the publish.

## Reference

Read [references/publish-checklist.md](references/publish-checklist.md) for a compact checklist when needed.
