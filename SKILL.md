---
name: excalidraw
description: "Generate hand-drawn Excalidraw-style flowcharts, architecture diagrams, sequence flows, entity-relationship diagrams, and decision trees as SVG + PNG from text prompts. Use when the user requests diagrams, flowcharts, architecture sketches, system designs, hand-drawn visuals, or says /excalidraw."
user_invocable: true
---

# Excalidraw Diagram Generator

Generate hand-drawn style diagrams rendered as SVG + PNG.

## Configuration

Set your preferred output directory below. Defaults to `~/Downloads/Excalidraw/`.

```
OUTPUT_DIR: ~/Downloads/Excalidraw/
```

To customize, edit the `OUTPUT_DIR` line above to any absolute path (e.g., `~/Documents/Diagrams/`, `~/Desktop/Sketches/`).

| Key | Value |
|-----|-------|
| Formats | `.svg` + `.png` (2x resolution) |
| Fonts | Virgil (handwritten) + Cascadia (monospace) |
| Renderer | `scripts/render.mjs` (roughjs + xmldom + resvg) |
| Reference | `references/element-format.md` |

## Workflow

### Step 1: Analyze the request

Identify the components, relationships, and flow direction from the user's prompt. Decide on:
- Shape types (rectangles for services, ellipses for actors, diamonds for decisions)
- Connections (arrows between components)
- Layout direction (top-to-bottom, left-to-right)
- Appropriate canvas size

### Step 2: Read the element format reference

Read `references/element-format.md` (relative to this skill's directory) for the full JSON format specification, color palette, and layout rules.

### Step 3: Generate diagram JSON

Create a JSON object following the element format. Key rules:

**IDs and references:**
- Give every shape that receives/sends arrows a unique `id`
- Arrow `from`/`to` reference shape IDs

**Text and labels (CRITICAL for clean output):**
- Use the `subtitle` property on title `text` elements for descriptive text below titles — never create separate overlapping text elements at similar Y positions.
- Use the `annotation` property on shapes for technical details (endpoint, protocol, size) — never place separate `text` elements near shapes for descriptions.
- Use `sectionLabel` property on zone rectangles — never use a separate `text` element for zone/section names.
- Keep arrow labels to 1-3 words. The renderer auto-positions them to avoid collisions.
- If the diagram has notes or key findings at the bottom, use a single `text` element with `subtitle` — never stack multiple text elements.

**Sizing shapes for content:**
- Shapes without annotations: min height 60-80px
- Shapes with annotations: min height 90-120px (annotations need room below the label). Never use fontSize below 13 for any text.
- Increase width for long labels (the renderer wraps to shape width minus 28px padding)

**Spacing and layout:**
- Space shapes 60-80px apart minimum
- Keep 40px padding from canvas edges
- Calculate canvas dimensions from content (see element-format.md for formula)
- Never stack more than 2 text blocks in the same vertical column at the bottom of the diagram

**Style:**
- Use `rounded: true` for service/component rectangles
- Use pastel fills from the color palette for readability
- Zone rectangles use very faint fills (e.g., `#e7f5ff`) with `sectionLabel`

### Step 4: Validate JSON

Before rendering, verify the generated JSON is valid: parse it with `JSON.parse()` or `node -e` and confirm every arrow `from`/`to` references an existing shape `id`.

### Step 5: Render

1. Read the `OUTPUT_DIR` from the Configuration section above. Create the directory if it doesn't exist.

2. Save the JSON to a temp file:
   ```bash
   cat > /tmp/excalidraw-diagram.json << 'DIAGRAMEOF'
   { ...the JSON... }
   DIAGRAMEOF
   ```

3. Run the renderer (use the absolute path to `scripts/render.mjs` within this skill's directory):
   ```bash
   node <SKILL_DIR>/scripts/render.mjs /tmp/excalidraw-diagram.json "<OUTPUT_DIR>"
   ```

4. If the renderer exits non-zero, read the error output, fix the JSON (common issues: missing IDs, invalid color hex, shapes outside canvas bounds), and re-run.

5. Clean up the temp file.

### Step 6: Report output

Tell the user the file paths for both SVG and PNG. Open the PNG for visual inspection. If the diagram looks wrong (overlapping elements, missing labels), return to Step 3 and adjust.

## Usage Examples

- `/excalidraw user -> API gateway -> auth service -> database`
- `/excalidraw microservices architecture with 4 services`
- `/excalidraw decision flowchart for code review process`
- `draw me a hand-drawn diagram of the deployment pipeline`

## Requirements

- Node.js 18+
- Dependencies installed: `cd scripts && npm install`
