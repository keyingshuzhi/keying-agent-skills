# GitHub CLI Skill — Analysis Report

## Objective
Provide a reusable agent Skill for common GitHub CLI operations with explicit scope resolution, safe mutation boundaries, and verifiable results.

## Scope
- In scope: authentication checks, repositories, issues, PRs, releases, Actions, read-only API queries, troubleshooting.
- Out of scope: replacing `git`, managing GitHub billing, guaranteeing API permissions, handling secrets directly, arbitrary deployment scripts.

## Design decisions
1. `SKILL.md` is the entry point and contains activation cues and a concise execution policy.
2. `references/guide.md` holds commands and recipes to keep the main Skill lightweight.
3. `extension.json` is **a project-local metadata example**, not an official universal Skill extension specification. Consumers should adapt it to their host application.
4. Read-before-write and post-write verification are required to reduce mistakes.
5. Repo scope is explicit (`-R OWNER/REPO`) to prevent accidental operations on the wrong project.

## Risks and mitigations
| Risk | Mitigation |
| --- | --- |
| Unauthorized publication or deletion | Require explicit approval for high-impact remote writes. |
| Credentials leakage | Never echo tokens or raw secrets. |
| Wrong target repository | Resolve repo before action and pass `-R`. |
| Unreliable commands due to CLI changes | Consult `gh <command> --help` and official manual. |
| CI logs contain sensitive data | Redact logs before sharing. |
| Licensing confusion | Skill MIT license covers only this Skill, not third-party `gh` or user repositories. |

## Acceptance checks
- [ ] `SKILL.md` frontmatter parses.
- [ ] `references/guide.md` exists and is linked.
- [ ] `extension.json` is valid JSON.
- [ ] LICENSE and README are present.
- [ ] Commands distinguish read-only from remote writes.
- [ ] Examples never contain real tokens.

## Limitations
Command examples are templates and have not been executed against the user's GitHub account. GitHub CLI behavior may vary by installed version, account permissions, and repository policy.
