# Analysis Report: Plot Generator Skill

## 1. Project Overview
The Plot Generator skill is a **meta‑skill** that provides instructions for the agent to utilize the existing MCP Research‑Draw tool to produce visualizations. It does not contain executable code; instead it defines how the agent should discover, plan, and render plots.

## 2. Technology Stack
- **Type**: DSH Skill (Markdown + extension.json)
- **Language**: Markdown (instructional)
- **Runtime**: Relies on the MCP Research‑Draw service exposed via the `mcp__research-draw_*` tool namespace.
- **Version Management**: Directory snapshot (v0.1.0)
- **Dependencies**: None beyond the MCP tool availability.

## 3. Project Structure
```
skill-plot-generator/
├── SKILL.md                 # Skill contract and workflow
├── extension.json           # Machine‑readable metadata
├── references/
│   ├── guide.md             # Cheat‑sheet of plot groups and usage
│   ├── list_plots.md        # (Optional) cached list of plots
│   └── schema_example.md    # (Optional) example schema output
├── scripts/
│   ├── list_available.py    # Helper to list plots
│   ├── check_schema.py      # Helper to get schema
│   └── render_plot.py       # Helper to render with validation
├── tests/
│   └── test_helpers.py      # Placeholder for unit tests
├── README.md                # Usage documentation
└── LICENSE                  # MIT license
```

## 4. Program Entrypoint
Not applicable – the skill is invoked by the agent via the skill system; there is no standalone executable.

## 5. System Architecture
- **Host (DSH)** → provides the MCP Research‑Draw tool.
- **Skill (SKILL.md)** → defines how the agent should call the MCP tool.
- **Agent** → follows the skill’s workflow to inspect, plan, apply, and verify plots.
- No client‑side UI; all output is returned as tool results (image/base64 or file path).

## 6. Core Modules
- **Inspect Module**: Use `mcp__research-draw__list_plots` and `mcp__research-draw__get_plot_schema`.
- **Plan Module**: Validate required fields, choose mode, assemble config.
- **Apply Module**: Execute `mcp__research-draw__render_plot`.
- **Verify Module**: Check output file existence and basic properties.

## 7. Dependencies
- **Runtime Dependence**: Availability of the `mcp__research-draw_*` tool set.
- **Internal Dependencies**: None.
- **External Services**: MCP Research‑Draw service (bundled with DSH).

## 8. Data & State
- The skill itself stores no state.
- Plot generation may produce image files inside the workspace (under user‑approved paths).
- No persistent configuration or environment modification.

## 9. Build & Run
- No build step required.
- "Run" = agent following the skill’s instructions to call MCP tools.

## 10. Testing
- No automated test suite currently; the skill relies on manual verification.
- Helper scripts in `scripts/` could be unit‑tested in future.

## 11. Deployment & Release
- Distributed as a skill directory; version tracked via `extension.json`.
- Release process: update version in `extension.json`, add a git tag, and optionally create a GitHub release.

## 12. Key Findings
**Finding 1**: The skill correctly references the MCP tool namespace and outlines a clear, safe workflow.  
**Evidence**: The SKILL.md details inspect/plan/apply/verify steps and maps them to specific MCP calls.  
**Confidence**: High

**Finding 2**: All required files for a DSH Skill are present (SKILL.md, extension.json, references, README, LICENSE).  
**Evidence**: Directory listing shows each file.  
**Confidence**: High

**Finding 3**: The skill does not attempt to install or modify external systems, adhering to the default read‑only stance unless explicit apply authorization is given.  
**Evidence**: The "Safety & Constraints" section forbids unauthorized writes and global changes.  
**Confidence**: High

## 13. Technical Risks
**Risk 1**: If the MCP Research‑Draw tool is unavailable or changes its interface, the skill’s instructions may become outdated.  
**Evidence**: Dependence on external tool; no version pinning.  
**Confidence**: Medium  
**Mitigation**: Periodically verify tool availability via a simple inspection call.

**Risk 2**: Users may supply malformed data paths or incorrect column names, leading to rendering failures.  
**Evidence**: The skill relies on user‑provided config; errors bubble up from the MCP tool.  
**Confidence**: Medium  
**Mitigation**: The plan step includes validation against the schema and encourages demo mode first.

## 14. Unknowns / Items to Confirm
- The exact format expected for inline data arrays in `config` when not using a file (the MCP documentation may specify JSON‑list vs CSV string).
- Whether the MCP tool supports streaming or returning raw pixel data beyond base64 PNG.
- Any limits on image size or number of concurrent render requests.

## 15. Suggested Further Investigation
- Test each plot group with a representative dataset to confirm the schema matches actual rendering behavior.
- Automate a nightly check that lists the plots and ensures no breaking changes in the MCP tool API.
- Consider adding a small wrapper script that normalizes common inputs (e.g., converting a pandas DataFrame to the expected format) for advanced users.

---
*Report generated based on the current state of the skill directory.*