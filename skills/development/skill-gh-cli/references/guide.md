# GitHub CLI (`gh`) Practical Guide

## 1. Install and authenticate

macOS (Homebrew):
```bash
brew install gh
gh --version
gh auth login
gh auth status
```

Windows (WinGet):
```powershell
winget install --id GitHub.cli
# Open a new terminal after installation
gh --version
gh auth login
```

Linux: use the package instructions for your distribution from the official manual. `gh auth login` supports a browser-based login flow. In automation, use an environment-provided token and avoid echoing it.

Official docs: https://cli.github.com/manual/

## 2. Repository operations

```bash
# Inspect a repository
gh repo view OWNER/REPO --json nameWithOwner,description,visibility,url
# List your repositories
gh repo list --limit 50
# Clone
gh repo clone OWNER/REPO
# Create a private repository (remote write)
gh repo create REPO --private --description "Project description" --confirm
# View repository in browser
gh repo view OWNER/REPO --web
```

`gh repo create` flags can change between CLI versions; run `gh repo create --help` before scripting. Avoid accidentally making a repository public.

## 3. Issues

```bash
gh issue list -R OWNER/REPO --state open --limit 50
gh issue view 123 -R OWNER/REPO
gh issue create -R OWNER/REPO --title "Bug: example" --body "Reproduction steps..."
gh issue comment 123 -R OWNER/REPO --body "Update..."
```

Issue creation and commenting are external writes: verify repo and content before executing.

## 4. Pull requests

```bash
gh pr list -R OWNER/REPO --state open
gh pr view 42 -R OWNER/REPO --json title,state,baseRefName,headRefName,url
gh pr checks 42 -R OWNER/REPO
gh pr diff 42 -R OWNER/REPO
gh pr create --base main --head feature/example --title "Add feature" --body "Summary and testing"
```

Merging changes remote history; obtain explicit authorization and inspect CI/review status first.

## 5. Releases and assets

```bash
gh release list -R OWNER/REPO
gh release view v0.1.0 -R OWNER/REPO
gh release create v0.1.0 ./dist/app.zip -R OWNER/REPO --title "v0.1.0" --notes "Release notes" --draft
gh release upload v0.1.0 ./dist/checksums.txt -R OWNER/REPO --clobber
```

Publishing, replacing assets, or deleting releases changes external state. For installers, verify architecture, checksums, licensing and signing status. Use a draft first when appropriate.

## 6. GitHub Actions

```bash
gh workflow list -R OWNER/REPO
gh run list -R OWNER/REPO --limit 20
gh run view RUN_ID -R OWNER/REPO
gh run view RUN_ID -R OWNER/REPO --log-failed
gh run watch RUN_ID -R OWNER/REPO
# Explicitly triggers a workflow (remote write)
gh workflow run build.yml -R OWNER/REPO --ref main
```

Never paste raw logs containing tokens or credentials. Workflow dispatch may incur billing or affect deployments.

## 7. API and JSON automation

```bash
gh api repos/OWNER/REPO --jq '{full_name,private,default_branch}'
gh repo view OWNER/REPO --json nameWithOwner,visibility,url --jq '.url'
gh api --paginate repos/OWNER/REPO/issues --jq '.[].number'
```

`gh api` can issue modifying requests via `--method` and `--field`; review these as carefully as explicit `gh` write commands. Prefer documented `gh` subcommands for common tasks.

## 8. Troubleshooting

| Symptom | Check |
| --- | --- |
| `gh: command not found` | Confirm installation and restart terminal / check PATH. |
| Not logged in | Run `gh auth status`, then `gh auth login`. |
| `404 Not Found` | Check OWNER/REPO, access permissions, repository visibility and asset/tag existence. |
| `403` | Check scopes, organization policy, rate limits, and token permissions. |
| Wrong repository | Always pass `-R OWNER/REPO` when working outside the repository. |
| Release asset upload fails | Confirm tag/release exists, file exists, filename matches, and user has write permissions. |
| CI failure | Inspect `gh run view RUN_ID --log-failed` with secret redaction. |

## 9. Example workflow: publish a Windows desktop release

1. Verify repository and working tree (`gh repo view`, `git status`).
2. Build Windows installer using project-specific build steps.
3. Generate SHA-256 checksums; verify local artifacts.
4. Review release notes, license notices, target architecture and version.
5. Create a **draft** release with `gh release create TAG FILE --draft`.
6. Inspect with `gh release view TAG`.
7. Publish only after explicit user approval.

## 10. Security and privacy

Use least-privilege credentials. Do not persist tokens in repository files or shell history. Do not log secret contents. Private repository data stays private unless user authorizes disclosure. For destructive commands, obtain explicit target and confirmation.
