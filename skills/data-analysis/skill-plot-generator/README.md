# Plot Generator Skill

This skill enables the agent to create a wide variety of plots (statistical, domain-specific, 3D, maps) using the MCP Research‑Draw tool.

## Install
Copy this directory to your agent's Skills location:
```bash
cp -r skills/development/skill-plot-generator /path/to/koying-pilot/skills/
```

## Usage
- Ask the agent to “list available plots” → it will call `mcp__research-draw__list_plots`.
- Ask for details of a specific plot (e.g., “what do I need for a volcano plot?”) → it will call `mcp__research-draw__get_plot_schema`.
- Request a plot to be drawn (e.g., “draw a scatter plot of gene length vs expression”) → the agent will plan, then apply the `mcp__research-draw__render_plot` tool with the appropriate parameters.

## Files
- `SKILL.md` – skill entry point with behavior contract.
- `extension.json` – machine‑readable metadata.
- `references/` – guidance and reference outputs.
- `scripts/` – optional helper scripts.
- `tests/` – placeholder for unit tests.
- `README.md` – this file.
- `LICENSE` – MIT license.

## License
MIT