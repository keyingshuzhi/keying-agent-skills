# Plot Generator Reference Guide

## Supported Plot Groups
The MCP Research‑Draw tool organizes plots into four groups:

1. **stats_2d** – 2‑D statistical plots (box, violin, scatter, volcano, forest)
2. **domain_2d** – domain‑specific 2‑D plots (MA, enrichment dot, GSEA, embedding/Upset, KM, ROC, calibration, Bland‑Altman, DCA, stress‑strain, XRD/Raman, phase, hysteresis, spectral signature, time‑series index, confusion matrix, class distribution, candlestick, cumulative return, rolling stats, frontier, ACF/PACF, Likert, raincloud, IRT ICC, factor loadings, interaction)
3. **plots_3d** – 3‑D plots (scatter3d, surface3d, wireframe3d, contour3d, line3d, isosurface3d, slice3d, quiver3d, waterfall3d, embedding3d, mesh3d)
4. **maps** – geographic visualizations (world, world_admin1, china, choropleth_world, choropleth_china, raster, points)

## Common Parameters
Many plots share the following optional arguments in the `config` object:
- `title`: Plot title.
- `xlabel`, `ylabel`: Axis labels (for 2‑D plots).
- `zlabel`: Z‑axis label (for 3‑D plots).
- `color`, `palette`: Color scheme.
- `width`, `height`: Image dimensions in pixels (defaults vary).
- `dpi`: Resolution.

## Demo Data
If a plot declares a `demo_data` path, you can render it in `"demo"` mode without supplying your own data. The demo data is bundled with the MCP service and is read‑only.

## Custom Mode
In `"custom"` mode you must provide all **required fields** (listed in the plot’s schema) as keys in the `config` object. The values should be column names (when using a data file) or literal arrays/lists of numbers if you wish to embed data directly (consult the MCP documentation for exact expectations).

## File Mode
Use `"file"` mode when your data lives in a CSV/TSV file inside the workspace. Include the `"file"` key in `config` pointing to the absolute or workspace‑relative path. The plot’s required fields then refer to column names in that file.

## Example Workflow (Agent‑Side)
1. `mcp__research-draw__list_plots` → select "volcano".
2. `mcp__research-draw__get_plot_schema` → see required fields: `log2fc_col`, `p_col`.
3. User provides a CSV with columns `log2FoldChange` and `pvalue`.
4. Agent builds `config`: { `"log2fc_col"`: `"log2FoldChange"`, `"p_col"`: `"pvalue"`, `"title"`: `"Volcano Plot"`, `"xlabel"`: `"log2FC"`, `"ylabel"`: `"-log10(p)"` }.
5. Calls `mcp__research-draw__render_plot` with mode `"file"`, plot `"volcano"`, and the above `config`, plus `"file"`: `"/workspace/my_data.csv"`.
6. Agent receives `{success:true, "path":"/mcp/outputs/volcano.png", "image":"base64..."}` (path is in the MCP service's output directory).

## Tips
- Always verify the schema before rendering to avoid missing‑field errors.
- When in doubt, start with `"demo"` mode to confirm the plot type works.
- Save the output path for later inclusion in reports or further processing.
