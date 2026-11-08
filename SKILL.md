---
name: doc-visuals
description: Create or update visual documentation for this repo (interactive HTML pages, SVG mock-ups, icon-based diagrams, expandable project guides). Use when asked to organize multiple topics in one documentation page, collapse sections by topic, replace wording with icons, mimic a product UI (Databricks, Airflow, Anomalo) as SVG, add clickable steps, draw arrows, or explain topics with remote reference images and screenshots.
---

# Visual documentation workflow

## Principles
- Prefer **text-based, editable** output (SVG, HTML, Mermaid) over screenshots or traced bitmaps, so AI and humans can read and change it. Never auto-trace a screenshot: text becomes outlines.
- Diagrams with arrows: **hand-written SVG** with named groups (`<g id="task-...">`). Text-heavy cards and tables: HTML + CSS. Interactive diagrams: **inline SVG inside HTML**, with click handlers on the `id`ed groups. Keep the standalone `.svg` for sharing and note that the HTML holds a copy.
- Mock-ups of real products are **illustrative**: invent plausible names/IDs/states, add a one-line footnote saying so, and do not claim pixel accuracy.
- Keep the stage or step text as readable labels. Icons accompany wording; if titles are removed, keep an `aria-label` and tooltip.
- Visualize concepts as a short icon flow (4–6 nodes, arrows, red tint for failure and green for good outcome) plus a "context stored/decided" row and a one-line output.
- Show triggers between systems as visible arrows with the real call (for example `run-now`, `start update`), not just a line.

## Icons: avoid slow searches
The icon libraries are large (service-icons has ~9,400 files on `/mnt/c`, which is slow). Never glob, list, or open them broadly.
1. Build a filename index once, outside the repo: `find <repo> -type f \( -name '*.svg' -o -name '*.png' \) | sort > ~/icons-index.txt`.
2. `grep -i` the index for specific keywords; never print or read the whole index.
3. Pick icons from context yourself; do not ask the user to choose each one.
4. Sources: `../service-icons` (Azure/AWS/etc. product icons, Databricks logos only), `../tabler-icons/icons/outline` (generic concept icons, MIT), `../core` (Phosphor). Use product icons for stages and tools, Tabler for abstract steps.
5. Verify every chosen filename exists before use.

## Multi-topic project documentation
- Extend the existing project guide rather than creating a separate page per topic unless requested. Keep its existing filename and working links.
- Group each complete topic in a native `<details class="topic" id="topic-...">` with a descriptive `<summary>`, icon, and short scope description. Topics start collapsed, so readers expand only what they need. Native details/summary supplies keyboard and expanded-state semantics; do not replace it with a custom button.
- Add a compact topic index linking to stable topic IDs. Open the enclosing topic when navigating to a fragment (including nested section IDs), on initial load and hash changes. Keep topics independently expandable.
- Keep overview/status information outside topics. Put all topic-specific explanations, diagrams, commands, caveats, references, and sub-sections inside the relevant topic.
- Preserve existing interactivity and unique IDs. Ensure diagrams/detail panels affect only their own topic; native disclosures must work without JavaScript.
- Distinguish illustrative examples from actual project wiring, verified checks, remaining blockers, and untested end-to-end behavior. Never place credentials or personal information in the guide.
- Validate default collapsed state, summary keyboard activation, fragment expansion, existing controls, independent topics, local assets, and narrow-screen layout in the browser when available.

## Referencing assets
- The page viewer serves the repo folder as its web root, so `../` paths to sibling repos fail ("not found"). **Copy only the chosen icons** into the repo (`icons/`, `icons/steps/`, `icons/visual/`) and reference them with relative paths. Do not copy whole libraries.
- Tabler SVGs use `currentColor`; inside `<img>` they render black. Inline them if they must take a theme colour.
- URL-encode spaces (`%20`) in `src` paths.

## Interactive HTML conventions
- Clickable elements get `tabindex="0"`, `role="button"`, `aria-expanded`/`aria-pressed`, Enter/Space handling, hover and `:focus-visible` styles, and a "Click to visualize" hint when they open something.
- Clicking again, or opening another stage, hides the visual. Build DOM with `textContent`/`createElement`, not `innerHTML`.
- Store content in a JS data object (`visuals`, `info`) keyed by id, not in markup.

## Internet image references
- Proactively use relevant internet screenshots and diagrams wherever they meaningfully improve an explanation. Choose them from the documentation topic and place them beside the matching section, not as unrelated decoration.
- Prefer official product documentation and stable, publicly accessible HTTPS image URLs. Respect source licensing and embedding restrictions; use a source-page link instead when embedding is not permitted.
- Reference the remote URL directly in HTML with `<img src="https://...">`. Do not download, copy, or commit these images; downloading is only appropriate if the user explicitly requests local or offline assets.
- Verify each image URL returns an image successfully and that its source page explains the relevant topic. Do not guess image paths or assume an image is current.
- Use a descriptive section/figure ID reflecting the product, view, and topic, meaningful `alt` text, a caption explaining the connection, and a link to the original source page. Make images responsive and use lazy loading.
- Clearly distinguish reference screenshots from the user's actual setup. Do not imply that an example image proves a deployment or configuration succeeded.
- Explain that remote images require internet access and may change or disappear. On load failure, show an explicit message and retain the source link.
- Keep editable diagrams and explanatory text alongside the images; remote screenshots supplement rather than replace them.

## Validating
- The Microsoft Viewer plugin must be installed in VS Code to open the project HTML guide using the user's preferred viewer. Do not assume it is installed or install it automatically. Let the user open/refresh the guide there; do not start browser integration or a preview server unless requested.
- SVG: `python3 -c "import xml.dom.minidom as m;m.parse('file.svg')"`. For browser checks, serve only the intended documentation and assets on a local HTTP port if file URLs fail. Reuse a shared browser page when available, and stop temporary preview servers afterward. If browser rendering cannot be checked, state that limitation plainly.
- Check icon files exist with a small script after generating paths.

## Working from images or UI references
- Do not ask for a whole saved HTML page of a product UI (huge, private). Ask for a screenshot plus copied styles (colours, fonts, sizes) for 3–5 key elements.
- To draw lines between parts of two images: place both in one SVG with `<image>`, overlay arrows. Use OCR (tesseract, install via `apt`) for exact text boxes; otherwise estimates are off by tens of pixels, so verify.
- Ask for screenshots with a descriptive file name that carries the metadata (product, view, and the doc section it belongs to, for example `airflow-graph-view_trigger-databricks.png`), so each image can be matched to its section and reflected there when needed. If a screenshot is unnamed, ask for the name or the section before using it.
- Do not trace screenshots. If an image must become text, ask for the source or re-create it by hand as SVG (shapes and `<text>`) or Mermaid.

## Constraints from the repo
- WSL only: Linux commands, never Windows executables.
- Follow the repo's AGENTS/custom instructions for Databricks bundles and Kafka; do not touch them for documentation tasks.
