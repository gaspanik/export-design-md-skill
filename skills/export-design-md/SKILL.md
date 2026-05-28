---
name: export-design-md
description: Converts DESIGN.md into a blog-style visual HTML document. Includes a default template; pass an HTML file path as an argument to use a custom one. In Astro projects, generates src/pages/design-preview.astro. After generation, optionally exports to Figma as a capture or as Figma nodes.
argument-hint: "[template.html]"
allowed-tools: Bash, Read, Write, Edit, AskUserQuestion, Skill
---

# Convert DESIGN.md into a Visual HTML Document

**Response language:** Respond in the same language the user used to invoke this skill.

## Step 1: Prerequisites Check

Check whether `DESIGN.md` exists in the project root.

```bash
ls DESIGN.md 2>/dev/null || echo "not found"
```

If not found, tell the user "DESIGN.md not found. Please generate DESIGN.md first." and stop.

## Step 2: Environment Check

Read `package.json` and check `dependencies` / `devDependencies`.

**Tailwind CSS (required):**
If `"tailwindcss"` is not present, tell the user "Tailwind CSS not found. This skill requires a project using Tailwind CSS." and stop.

**Framework detection:**
If Tailwind is confirmed, check for `"astro"`:

- **Astro present** → **Astro mode** (run Steps 5B & 6B)
- **Astro absent** → **HTML mode** (run Steps 5A & 6A)

## Step 3: Determine the Template

Check whether `$ARGUMENTS` contains a file path.

- **Argument provided** → Read the specified file as the template
- **No argument** → Use `template.html` in the same directory as this skill

The template path is relative to the skill directory. If the skill is at `.claude/skills/export-design-md/`, the default template path is `.claude/skills/export-design-md/template.html`.

## Step 4: Read and Parse DESIGN.md

Read `DESIGN.md` and extract:

**From the YAML front matter (between `---` delimiters):**
- `name` — project name
- `description` — one-line summary
- `colors` — color tokens (key: hex value)
- `typography` — typography tokens
- `rounded` — border-radius tokens
- `spacing` — spacing tokens
- `components` — component tokens

**From the Markdown body:**
- Content of each section (Overview, Colors, Typography, Layout, Elevation & Depth, Shapes, Components, Do's and Don'ts)

Resolve token references like `{colors.primary}` to their actual values before use.

---

## Step 5A: [HTML Mode] Collect `<head>` Tags from the Project

Gather the `<link>` / `<script>` tags to include in the generated HTML's `<head>`.

```bash
# Find HTML files in the root, excluding node_modules / dist
find . -maxdepth 2 -name "*.html" \
  ! -path "*/node_modules/*" ! -path "*/dist/*" \
  ! -name "design-preview.html" | head -5
```

Read the found HTML file (typically `index.html`) and extract from `<head>`:
- External `<link rel="stylesheet">` tags (fonts, etc.)
- `<script type="module" src="...">` tags (entry points)

Reuse these as-is in the generated HTML's `<head>` and end of `<body>`. If no HTML file is found, grep `src/` for the entry point:

```bash
grep -rl --include="*.ts" --include="*.js" \
  --exclude-dir=node_modules --exclude-dir=dist \
  "main" src/ | head -3
```

## Step 6A: [HTML Mode] Generate and Write the HTML

Generate HTML following the template's **visual style and layout structure**, populated with the actual content from DESIGN.md. Write it to the project root as `design-preview.html`, overwriting any existing file without confirmation.

### Requirements for the Generated HTML

**Reuse the project's setup:**
- Do not use Tailwind CDN
- Use the `<link>` and `<script>` tags collected in Step 5A
- Color swatches must use inline `style="background:#XXXXXX"` (Tailwind JIT may not scan generated content)
- Typography samples must use inline `style` for `font-size` / `line-height` / `letter-spacing`
- Use Tailwind classes for general utilities (layout, spacing, text color, etc.)

**Page structure (following the template's style):**

1. **Header** — project name, navigation (anchor links to each section)
2. **Article head** — badge ("Design System"), title (project name + "Design System"), generation date
3. **Overview section** — Overview content from DESIGN.md
4. **Colors section** — color swatch list (swatch + name + hex value + description)
5. **Typography section** — sample text for each typography level (rendered at actual font size/family)
6. **Spacing section** — visual spacing scale (width bars)
7. **Shapes section** — visual border-radius samples (sample box for each value)
8. **Components section** — Components content from DESIGN.md
9. **Do's and Don'ts section** — Do's and Don'ts content from DESIGN.md
10. **Footer** — project name + generation date

**Color swatch implementation pattern:**
```html
<!-- ✅ Correct (inline style ensures reliable rendering) -->
<div style="background:#2957c8;height:7rem;border-radius:1rem;"></div>

<!-- ❌ Avoid (Tailwind class may not be generated) -->
<div class="bg-primary h-28 rounded-2xl"></div>
```

**Typography sample implementation pattern:**
```html
<p style="font-size:60px;font-weight:400;line-height:1.3;">
  Heading sample text
</p>
```

**Spacing bar implementation pattern:**
```html
<!-- spacing.lg = 40px -->
<div style="width:40px;height:1rem;background:#2957c8;border-radius:4px;"></div>
<span>lg — 40px</span>
```

**Border-radius sample implementation pattern:**
```html
<!-- rounded.md = 28px -->
<div style="width:5rem;height:5rem;border-radius:28px;background:linear-gradient(135deg,#2957c8,#1ba39c);"></div>
<span>md — 28px</span>
```

### Accessibility Rules

- Navigation must use `nav > ul > li > a` structure
- Uppercase text must not be written directly in HTML; use `style="text-transform:uppercase"` instead
- Icon buttons must have `aria-label`

---

## Step 5B: [Astro Mode] Understand the Project Structure

Inspect the Astro project's conventions:

```bash
# Check src/pages/ and src/layouts/
find src -name "*.astro" \
  ! -path "*/node_modules/*" ! -path "*/dist/*" | head -10
```

Read an existing page (e.g. `src/pages/index.astro`) and check:
- Whether it imports a layout component (e.g. from `src/layouts/`)
- Whether font `<link>` tags are in the page or managed by the layout
- How Tailwind classes are used

If a layout component exists, read it to understand the `<head>` structure.

## Step 6B: [Astro Mode] Generate and Write the .astro File

Generate `src/pages/design-preview.astro` based on the structure found above. Overwrite any existing file without confirmation.

### Requirements for the Generated .astro File

**Layout approach:**
- If the project has a layout component, import and use it
- If not, generate a standalone page starting from `<html>`, including font `<link>` tags following existing pages
- Do not use Tailwind CDN (handled via Astro's integration)

**Front matter (`---` block):**
```astro
---
// design-preview.astro
const title = "[Project Name] Design System";
---
```

**Content structure is the same as HTML mode:**
Page sections (Header, Overview, Colors, Typography, Spacing, Shapes, Components, Do's and Don'ts, Footer) are identical to HTML mode. Note Astro's JSX syntax (`class` is used as-is; inline styles use `style="..."`).

**Color swatches and typography samples must use inline styles (same reason as HTML mode).**

---

## Step 7: Completion Report & Next Action

Report the generated file path, mode used, and how to view it in the browser:
- HTML mode: start the dev server, then open `http://localhost:[PORT]/design-preview.html`
- Astro mode: run `astro dev`, then open `http://localhost:4321/design-preview`

Then use `AskUserQuestion` to ask what to do next:

```
What would you like to do next?

1. Import into Figma as a capture
   → Transfer the browser-rendered result to Figma as an image (generate_figma_design)

2. Generate as Figma nodes (/figma:figma-use)
   → Create a design system with components, variables, and auto-layout

3. Done
```

---

## Step 8: Follow-up Based on Selection

### If 1 is selected — call `generate_figma_design` directly

**Do NOT invoke the `/figma:figma-generate-design` skill. Do NOT call `use_figma`. Pure capture only.**

Steps:
1. Start the dev server and get the preview URL (HTML mode: `npm run dev` etc., Astro mode: `astro dev`, use the project's configured port)
2. Call `generate_figma_design` without `outputMode` first to present destination options to the user
3. Once the user selects a destination, call it again with `outputMode` and the target URL
4. Poll using the returned `captureId` every 5 seconds, up to 10 times, until complete
5. Report the Figma URL to the user

To find the correct tool name (prefix varies by installation method):
```
ToolSearch: "generate_figma_design"
→ Use the exact tool name returned
```

### If 2 is selected — invoke the `figma:figma-use` skill

Call the `figma:figma-use` skill via the `Skill` tool, passing DESIGN.md token information (colors, typography, spacing) as context.

### If 3 is selected — done

No further action.
