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
- **No argument (HTML mode)** → Use `template.html` in the same directory as this skill
- **No argument (Astro mode)** → Use `template.astro` in the same directory as this skill

The template path is relative to the skill directory. If the skill is at `.claude/skills/export-design-md/`, the default template paths are:
- HTML mode: `.claude/skills/export-design-md/template.html`
- Astro mode: `.claude/skills/export-design-md/template.astro`

Read the appropriate template **before** generating the output file.

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

**Start from `template.html` verbatim. Do NOT redesign or rewrite it.** Replace only the `[placeholder]` values with actual content from DESIGN.md. The Tailwind classes, layout structure, and visual style must be preserved exactly as-is.

Write to the project root as `design-preview.html`, overwriting any existing file without confirmation.

### What to replace

**In `<head>`:**
- Replace the CDN `<link>` tags with the `<link>` / `<script>` tags collected in Step 5A

**In the body — replace each `[placeholder]` with real content from DESIGN.md:**

| Placeholder | Replace with |
|---|---|
| `[Project Name]` | `name` from front matter |
| `[Description]` | `description` from front matter |
| `[Date]` | today's date |
| `[Overview text from DESIGN.md]` | Overview section body |
| `[Colors section text from DESIGN.md]` | Colors section body |
| `[Typography section text from DESIGN.md]` | Typography section body |
| `[Shapes section text from DESIGN.md]` | Shapes section body |
| `[Components section text from DESIGN.md]` | Components section body |
| `[Do item]` / `[Don't item]` | Do's and Don'ts items |

**For repeated token blocks** (colors, spacing, shapes, typography samples) — duplicate the example `<div>` blocks from the template for each token, filling in the actual values. Keep the surrounding wrapper elements unchanged.

**Inline style rules (do not use Tailwind classes for these):**
- Color swatches: `style="background:#XXXXXX;height:7rem;border-radius:0.75rem;"`
- Typography samples: `style="font-size:Xpx;font-weight:N;line-height:N;"`
- Spacing bars: `style="width:Xpx;height:1rem;background:#XXXXXX;border-radius:2px;flex-shrink:0;"`
- Border-radius samples: `style="width:5rem;height:5rem;border-radius:Xpx;background:linear-gradient(135deg,#A,#B);"`

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

**Start from `template.astro` verbatim. Do NOT redesign or rewrite it.** The Tailwind classes, layout structure, and visual style must be preserved exactly as-is.

Write to `src/pages/design-preview.astro`, overwriting any existing file without confirmation.

### What to change

**Front matter — update values only:**
```astro
---
// If a layout exists, add:
import Layout from '../layouts/Layout.astro';
const title = "[Project Name] Design System";   // ← fill in
const generatedDate = "YYYY-MM-DD";             // ← today's date
---
```

**Layout approach — only if a layout component exists:**
- Wrap the body content with `<Layout title={title}>…</Layout>` and remove the `<html>/<head>/<body>` wrapper
- Otherwise keep the standalone structure and copy font `<link>` tags from existing pages into `<head>`
- Do not use Tailwind CDN (handled via Astro's integration)

**In the body — same placeholder replacements as HTML mode** (see the table in Step 6A). For repeated token blocks, duplicate the example element from the template for each token.

**Inline style rules are identical to HTML mode** — color swatches, typography samples, spacing bars, and border-radius samples all use `style="..."` not Tailwind classes.

**Astro syntax reminders (do not alter the template beyond these):**
- `class=` is correct (not `className`)
- Self-closing tags need `/>` (e.g. `<br />`, `<hr />`, `<meta />`)
- No `<script type="module" src="/src/main.ts">` — Astro handles bundling automatically

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
