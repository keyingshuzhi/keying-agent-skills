---
name: workspace-environment-manager
description: Diagnose and prepare isolated project environments in restricted AI-agent workspaces. Reuse accessible host tools before considering installation; distinguish missing tools from PATH, permission, sandbox and execution failures; keep project dependencies and generated state inside the workspace; use vetted network mirrors only when needed. Trigger for environment setup, uv/pip/npm/brew not found, dependency installation, workspace isolation, network timeouts and mirror fallback.
---

# Workspace Environment Manager

## Purpose
This skill equips the agent to diagnose and prepare isolated project environments inside restricted AI‑agent workspaces. It focuses on:
- Detecting which tools are already available on the host (PATH, permissions, sandbox).
- Distinguishing between missing tools, PATH failures, permission denials, sandbox restrictions, and execution errors.
- Keeping all project dependencies and generated state inside the workspace root.
- Using vetted network mirrors only when necessary (e.g., pip, npm, brew, uv).

## When to Use
- The user asks to set up a project environment, install dependencies, or configure a workspace.
- A tool like `uv`, `pip`, `npm`, or `brew` is reported as "not found" or fails unexpectedly.
- The agent needs to isolate a project's dependencies and generated files inside the workspace.
- Network operations time out and a mirror fallback is needed.

## When Not to Use
- The user wants to modify the host system outside the workspace.
- The task is purely about code review, testing, or architecture (use the corresponding skills).
- The user explicitly forbids network access or mirror usage.

## Working Modes & Permissions
- **Inspect (default, read‑only)**: Check available tools, environment variables, PATH entries, and sandbox restrictions. No writes or side effects.
- **Plan**: Propose an environment setup plan (tools to install, mirrors to use, directories to create). This step does not execute anything; it only validates the approach and asks for user approval if external network calls or system modifications are involved.
- **Apply**: Execute the plan — create directories, install dependencies, configure mirrors. This mode may write files inside the workspace and perform network calls (via vetted mirrors). It requires explicit user authorization.
- **Verify**: After applying, confirm that tools are accessible, dependencies are installed, and generated state is contained within the workspace.

## Standard Workflow
1. **Establish Context**
   - Identify the workspace root where all project files and dependencies must reside.
   - Determine which language ecosystems are involved (Python, Node.js, etc.).
2. **Inspect**
   - Run `which <tool>` or `command -v <tool>` to check availability.
   - Check environment variables (`PATH`, `PYTHONPATH`, `NODE_PATH`, etc.).
   - Inspect sandbox restrictions (e.g., read‑only filesystem, network blocking).
3. **Plan**
   - Based on inspection results, decide whether tools need to be installed or if existing tools can be reused.
   - Choose appropriate mirrors (e.g., `pypi.tuna.tsinghua.edu.cn` for pip, `registry.npmmirror.com` for npm).
   - Outline the exact commands and directories to create.
   - Request user approval before proceeding.
4. **Apply**
   - Create project directories inside the workspace (e.g., `<workspace>/.venv`, `<workspace>/node_modules`).
   - Install dependencies using the selected mirrors.
   - Ensure all generated files stay within the workspace root.
5. **Verify**
   - Re‑check tool availability after installation.
   - Confirm that dependencies are functional (e.g., `python -c "import <pkg>"`).
   - Verify that no files were written outside the workspace.

## Safety & Constraints
- **Read‑only by default**: Inspect and plan modes never modify the filesystem or environment.
- **Write only with permission**: Apply mode may create directories and install packages inside the workspace; the agent must not write outside the workspace root.
- **Network usage**: Only vetted, well‑known mirrors may be used for package downloads. The agent must not attempt arbitrary network calls or access untrusted sources.
- **No automatic installation**: The skill never installs system‑wide packages or modifies the host beyond the workspace. If a tool is unavailable and cannot be installed inside the workspace, the agent should report the limitation and suggest alternatives.
- **Data privacy**: Do not send sensitive or confidential data to external mirrors unless the user explicitly authorizes it.

## Fact, Inference, and Result Labeling
- `Verified`: The tool is accessible and functional; evidence includes successful command execution and import checks.
- `Likely`: The plan indicates the tool should be installable, but installation has not yet been attempted.
- `Needs Verification`: The user has approved a plan but the apply step has not been executed.
- `Unknown`: Required information (such as network access or mirror availability) is missing; the agent should ask for clarification.

## Default Output Format
When asked to summarize an environment task, the agent may use:
```
Tool: <name>
Status: AVAILABLE / NOT_FOUND / PERMISSION_DENIED / SANDBOX_BLOCKED
Action: <what was done or proposed>
Workspace Path: <workspace>/<path> (if applicable)
Message: <details>
```

## References
- `references/guide.md` – detailed mirror configurations and troubleshooting steps.

---
*This skill is a set of instructions for the agent to manage project environments within the workspace.*