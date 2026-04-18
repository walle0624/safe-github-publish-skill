# Safe GitHub Publish

A Codex skill for publishing local repositories to GitHub safely, especially in environments where `gh` may be missing, auth may be half-configured, or a public push needs one more privacy and packaging pass before it goes live.

## What It Does

This skill helps Codex:

- inspect the local repository before publishing
- check for obvious secrets or machine-local leakage before a public push
- prefer working SSH when it is already configured for GitHub
- recover GitHub auth without getting stuck on missing `gh`
- reuse a logged-in browser session when GitHub web actions are required
- bridge Git credentials correctly for HTTPS pushes
- create the remote repository and finish the upload
- add a lightweight open-source baseline when the repo is meant to be shared

## Contents

- [github-yeet-safe/SKILL.md](github-yeet-safe/SKILL.md)
- [github-yeet-safe/references/publish-checklist.md](github-yeet-safe/references/publish-checklist.md)

## When To Use It

Use this skill when the user wants to:

- publish a local project to GitHub
- create a new GitHub repository
- push a repo publicly with share-safety checks
- recover from missing `gh`, missing auth, or broken git credential setup

## Notes

- The workflow prefers finishing the publish with the tools that already exist instead of blocking on one missing utility.
- Public publishing is treated as a workflow, not a single command.
- The skill is intentionally strict about auth verification and privacy checks before a public push.
- When SSH already works, the skill prefers SSH remotes so other tools such as OpenClaw can reuse the same machine-level credentials.
