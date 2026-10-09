---
name: github-cli
description: Use GitHub CLI (gh) to authenticate, manage repositories, issues, pull requests, releases, and GitHub Actions safely from a terminal.
---

# GitHub CLI Skill

## Purpose
Use the official `gh` CLI for GitHub platform operations; use `git` for local version control. Prefer `gh` commands over hand-written REST calls for supported operations.

## When to use
- User asks to create, inspect, clone, or configure a GitHub repository.
- User asks to manage issues, pull requests, releases, or Actions workflows.
- User asks to check GitHub authentication or use `gh api`.

## Prerequisites
1. Check `gh --version` and `git --version`.
2. Check `gh auth status` without exposing tokens.
3. Determine the target repository explicitly with `gh repo view --json nameWithOwner` or `-R OWNER/REPO`.
4. For installation, consult `references/guide.md`.

## Operating procedure
1. **Identify intent**: read-only, create, update, publish, or destructive.
2. **Resolve scope**: account, repository, branch, issue/PR number, workflow or release tag.
3. **Inspect before changing**: retrieve existing state using `gh ... view/list`.
4. **Plan commands**: prefer noninteractive, explicit `-R OWNER/REPO` and structured `--json` output.
5. **Get approval** before external writes that publish, send, delete, merge, change access, trigger workflows, or modify remote resources, unless the user has already explicitly authorized that exact action.
6. **Execute and verify** with `gh ... view/list` and report identifiers or URLs.
7. **Never display credentials**, tokens, secret values, or private repository content without authorization.

## Command patterns
```bash
gh auth status
gh repo view OWNER/REPO --json nameWithOwner,visibility,url
gh repo list OWNER --limit 30 --json name,visibility,url
gh issue list -R OWNER/REPO --state open --limit 30
gh pr list -R OWNER/REPO --state open --limit 30
gh run list -R OWNER/REPO --limit 10
gh release list -R OWNER/REPO --limit 10
```

## Guardrails
- Never run `gh auth token` to print tokens into chat or logs.
- Never assume the current directory points to the intended repository.
- Treat `gh repo delete`, `gh release delete`, `gh secret set`, `gh pr merge`, `gh workflow run`, `gh repo edit --visibility`, and release publication as high-impact operations.
- For CI failures, inspect `gh run view ... --log-failed`; redact secrets and private data.
- Distinguish local commands (`git add/commit`) from remote effects (`git push`, `gh` writes).
- Do not fabricate command output or claim successful execution without verification.

## Detailed reference
See [references/guide.md](references/guide.md) for setup, recipes, troubleshooting, and advanced usage.
