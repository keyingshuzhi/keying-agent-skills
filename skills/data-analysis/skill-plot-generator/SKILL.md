---
name: plot-generator
description: A skill for generating statistical, domain-specific, 3D, and map visualizations using the MCP Research-Draw tool. Provides guidance on listing available plots, understanding required parameters, and rendering plots with appropriate data.
---

# Plot Generator Skill

## Purpose
This skill equips the agent to create a wide variety of plots (boxplots, scatter, volcano, forest, MA, enrichment, GSEA, embedding/Upset, Kaplan‑Meier, ROC, calibration, Bland‑Altman, decision curve, stress‑strain, XRD/Raman spectra, phase diagrams, hysteresis, spectral signatures, time‑series indices, confusion matrices, class distributions, candlestick charts, rolling statistics, efficient frontiers, ACF/PACF, Likert, raincloud, IRT ICC, factor loadings, interaction effects, 3D scatter/surface/wireframe/contour/line/isosurface/slice/quiver/waterfall/embedding/mesh, world/China maps, choropleths, raster, point maps) via the MCP Research‑Draw service. It does **not** contain executable code; instead it tells the agent how to call the provided MCP tools (`mcp__research-draw__list_plots`, `mcp__research-draw__get_plot_schema`, `mcp__research-draw__render_plot`) to produce visualizations.

## When to Use
- The user asks to “make a plot”, “visualize”, “draw a chart”, or requests a specific plot type (e.g., “show me a volcano plot of differential expression”).
- The user wants to know what kinds of plots are available.
- The user needs to render a plot with given data columns and wishes to confirm required fields.
- The user seeks help choosing an appropriate visualization for their data.

## When Not to Use
- The user wants to perform statistical calculations or model fitting that is not directly tied to a plot.
- The user requests low‑level image processing (e.g., filtering, convolution) beyond what the MCP Research‑Draw tool provides.
- The user asks to modify the underlying MCP service or install additional plotting libraries; this skill only utilizes the existing MCP endpoint.

## Working Modes & Permissions
- **Inspect (default, read‑only)**: List available plots, query a plot’s schema, or check demo data locations. No writes or side effects.
- **Plan**: Prepare a rendering plan that specifies the plot type, mode (`demo`, `custom`, `file`), and required configuration. This step does not execute the plot; it only validates inputs and asks for user approval if external data or writes are involved.
- **Apply**: Execute the plot rendering using `mcp__research-draw__render_plot`. This mode may produce an image file (PNG) and/or base64‑encoded output. It requires explicit user authorization because it may write files to the workspace and perform network calls (if demo data must be fetched).
- **Verify**: After rendering, confirm that the output image exists, check its basic properties (size, format), and ensure no unexpected side effects occurred.

## Standard Workflow
1. **Establish Context**  
   - Determine the workspace path where any output image should be saved.  
   - Identify the data source: either a built‑in demo dataset, a user‑provided CSV/TSV, or in‑memory values passed via `config`.

2. **Inspect**  
   - Call `mcp__research-draw__list_plots` to retrieve the full catalog of available plots.  
   - Optionally, call `mcp__research-draw__get_plot_schema` for a specific plot to learn its required and optional fields, defaults, demo data location, and any extra dependencies (`dependency_extra`).

3. **Plan**  
   - Choose a plot type that matches the user’s intent and data structure.  
   - Decide the rendering mode:
     - `demo`: use the plot’s built‑in demo data (if any).  
     - `custom`: supply data via the `config` argument (must include all required fields).  
     - `file`: point to an external data file using the `file` parameter in `config`.  
   - Fill in optional parameters (title, axis labels, color scheme, thresholds, etc.) as desired.  
   - Summarize the plan: plot name, mode, data source, key configuration, expected output path.  
   - Request explicit user approval to proceed with `apply`.

4. **Apply**  
    - Invoke `mcp__research-draw__render_plot` with the selected `mode`, `plot` (dispatch key), and `config`.  
    - The tool returns a dictionary containing `success`, `message`, optionally `image` (base64 PNG) and `path` to a saved file (located in the MCP service's output directory).  
    - If `success` is true, move the file from the returned `path` to a desired location within the workspace (e.g., `<workspace>/<filename>.png`).  
    - If `success` is false, report the error and suggest alternatives (e.g., check required fields, try demo mode).

5. **Verify**  
   - Confirm that the returned `path` points to a valid PNG file (use `read_image` or check file size).  
   - Optionally, display a brief description of what the plot shows.  
   - Report any warnings (e.g., missing optional fields, use of defaults).  
   - If the plot was generated in `demo` mode, note that the data is illustrative and may not reflect the user’s actual dataset.

## Safety & Constraints
- **Read‑only by default**: Inspect and plan modes never write files or modify the environment.
- **Write only with permission**: Apply mode may create image files inside the workspace; the agent must not write outside the workspace root.
- **Network usage**: The MCP tool may fetch demo data from bundled files; it does not perform arbitrary network calls. No external download is attempted unless the user supplies a `file` URL that points outside the workspace (which should be avoided).
- **Data privacy**: Do not send sensitive or confidential data to the MCP service unless the user explicitly authorizes it and the data is included in the `config` for a `custom` render.
- **No automatic installation**: The skill never attempts to install Python packages, system tools, or modify the MCP service.

## Fact, Inference, and Result Labeling
- `Verified`: The plot was successfully rendered and the output file exists; evidence includes the tool’s `success:true` and a readable image.
- `Likely`: The schema indicates the plot should work with the given fields, but rendering has not yet been attempted.
- `Needs Verification`: The user has provided a plan but not yet executed the render step.
- `Unknown`: Required information (such as data columns) is missing; the agent should ask for clarification.

## Default Output Format
When asked to summarize a plotting task, the agent may use:

```
Plot Type: <dispatch_key>
Mode: <demo|custom|file>
Required Fields: <list>
Provided Config: <key‑value pairs>
Output Path: <workspace>/<filename>.png (if saved)
Status: SUCCESS / FAILED
Message: <tool message>
```

## References
- `references/guide.md` – concise cheat‑sheet of supported plots and typical use‑cases.
- `references/list_plots.md` – raw JSON output of `mcp__research-draw__list_plots` for reference.
- `references/schema_example.md` – example of a plot schema returned by `mcp__research-draw__get_plot_schema`.

## References Directory
- `guide.md` – high‑level guidance.
- `list_plots.md` – cached list of plots (optional).
- `schema_example.md` – example schema.

## Scripts (optional)
- `list_available.py` – thin wrapper to call `mcp__research-draw__list_plots` and pretty‑print.
- `check_schema.py` – wrapper for `mcp__research-draw__get_plot_schema`.
- `render_plot.py` – wrapper for `mcp__research-draw__render_plot` with basic validation.

## Tests
- Unit tests for the helper scripts (if implemented) can reside in `tests/`.

---
*This skill does not contain executable code; it is a set of instructions for the agent to leverage the existing MCP Research‑Draw tool.*
