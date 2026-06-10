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

**js-yaml (Astro mode only):**
The generated `design-preview.astro` reads `DESIGN.md` at build time using `js-yaml`. This package is a transitive dependency of Astro and is typically present in `node_modules` without explicit installation. If a build error occurs (`Cannot find module 'js-yaml')`), run `npm install js-yaml` (or the project's equivalent) to add it as a direct dependency.

**Framework detection:**
If Tailwind is confirmed, check for `"astro"`:

- **Astro present** → **Astro mode** (run Steps 5B & 6B)
- **Astro absent** → **HTML mode** (run Steps 5A & 6A)

**Main CSS detection (for auto export.css):**
Search for a main CSS entry point that contains `@import "tailwindcss"` or `@tailwind base`:

```bash
grep -rl --include="*.css" "@import \"tailwindcss\"\|@tailwind base" \
  src/ public/ . --exclude-dir=node_modules --exclude-dir=dist 2>/dev/null | head -3
```

- **No main CSS found** → set `AUTO_EXPORT_CSS=true` (export.css will be generated automatically after the preview)
- **Main CSS found** → set `AUTO_EXPORT_CSS=false`

## Step 3: Determine the Template

Check whether `$ARGUMENTS` contains a file path.

- **Argument provided** → Read the specified file as the template
- **No argument (HTML mode)** → Use `template.html` in the same directory as this skill
- **No argument (Astro mode)** → Use `template.astro` in the same directory as this skill

The template path is relative to the skill directory — resolve it from wherever this skill is actually installed (e.g. project-level `.claude/skills/export-design-md/` or global `~/.claude/skills/export-design-md/`):
- HTML mode: `<skill directory>/template.html`
- Astro mode: `<skill directory>/template.astro`

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

### Token variables block (HTML mode)

Emit a `<style>` block in `<head>` that maps all DESIGN.md tokens to CSS custom properties. This centralizes every value so edits only require updating this one block:

```html
<style>
  :root {
    /* Colors */
    --color-primary: #XXXXXX;
    --color-secondary: #XXXXXX;
    /* …all colors from DESIGN.md… */
    /* Spacing */
    --spacing-xs: Xpx;
    /* …all spacing… */
    /* Rounded */
    --rounded-sm: Xpx;
    /* …all rounded… */
    /* Components */
    --btn-primary-bg: #XXXXXX;   /* resolved from components.button-primary.backgroundColor */
    --btn-primary-color: #XXXXXX;
    --btn-primary-padding: Xpx Xpx;
    /* …other components… */
  }
</style>
```

**Inline style rules (use `var()` instead of hardcoded hex values):**
- Color swatches: `style="background:var(--color-primary);height:7rem;border-radius:0.75rem;"`
- Spacing bars: `style="width:var(--spacing-xs);height:1rem;background:var(--color-primary);border-radius:2px;flex-shrink:0;"`
- Border-radius samples: `style="width:5rem;height:5rem;border-radius:var(--rounded-sm);background:linear-gradient(135deg,var(--color-primary),var(--color-text));"`
- Component buttons: `style="background:var(--btn-primary-bg);color:var(--btn-primary-color);padding:var(--btn-primary-padding);"`

**Typography samples — use Tailwind classes (not inline styles):**
- Map font-size to the closest Tailwind text size class (e.g. `text-xs`, `text-sm`, `text-base`, `text-lg`, `text-xl`, `text-2xl`, …)
- Map font-weight to a Tailwind font-weight class (e.g. `font-thin`, `font-light`, `font-normal`, `font-medium`, `font-semibold`, `font-bold`, `font-extrabold`, `font-black`)
- Map line-height to a Tailwind leading class (e.g. `leading-none`, `leading-tight`, `leading-snug`, `leading-normal`, `leading-relaxed`, `leading-loose`)

### Accessibility Rules

- Navigation must use `nav > ul > li > a` structure
- Uppercase text must not be written directly in HTML; write it capitalized and apply the Tailwind `uppercase` class instead
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

### Token injection (Astro mode)

Add the following to the frontmatter so all token values are read from `DESIGN.md` at build time — values can never drift from the source:

```astro
---
import { readFileSync } from 'node:fs';
import yaml from 'js-yaml';
// import Layout from '../layouts/Layout.astro'; // if a layout exists

const raw = readFileSync('./DESIGN.md', 'utf-8');
const match = raw.match(/^---\n([\s\S]*?)\n---/);
const tokens: any = match ? yaml.load(match[1]) : {};
const { colors = {}, typography = {}, rounded = {}, spacing = {}, components = {} } = tokens as any;

// Resolve token references: "{colors.primary}" → actual hex value
const r = (v: any): string => {
  if (typeof v === 'string' && v.startsWith('{') && v.endsWith('}')) {
    return v.slice(1, -1).split('.').reduce((o: any, k: string) => o?.[k], tokens) ?? v;
  }
  return String(v ?? '');
};

const title = `${tokens.name ?? 'Project'} Design System`;
const description = tokens.description ?? '';
const generatedDate = "YYYY-MM-DD"; // ← today's date
---
```

Use template literals for all inline styles instead of hardcoded hex values:
- Color swatch: `` style={`background:${colors.primary};height:7rem;`} ``
- Spacing bar: `` style={`width:${spacing.xs};height:1rem;background:${colors.primary};`} ``
- Shape sample: `` style={`width:5rem;height:5rem;border-radius:${rounded.sm};background:linear-gradient(135deg,${colors.primary},${colors.text});`} ``
- Component button: `` style={`background:${r(components['button-primary']?.backgroundColor)};color:${r(components['button-primary']?.textColor)};padding:${components['button-primary']?.padding};`} ``

### What to change

**Front matter — already covered by token injection above. Only fill in `generatedDate`:**

**Layout approach — only if a layout component exists:**
- Wrap the body content with `<Layout title={title}>…</Layout>` and remove the `<html>/<head>/<body>` wrapper
- Otherwise keep the standalone structure and copy font `<link>` tags from existing pages into `<head>`
- Do not use Tailwind CDN (handled via Astro's integration)

**In the body — same placeholder replacements as HTML mode** (see the table in Step 6A). For repeated token blocks, duplicate the example element from the template for each token.

**Inline style rules are identical to HTML mode** — color swatches, spacing bars, and border-radius samples use `style="..."` not Tailwind classes. Typography samples use Tailwind classes (see HTML mode rules above).

**Astro syntax reminders (do not alter the template beyond these):**
- `class=` is correct (not `className`)
- Self-closing tags need `/>` (e.g. `<br />`, `<hr />`, `<meta />`)
- No `<script type="module" src="/src/main.ts">` — Astro handles bundling automatically

---

## Step 7: Auto export.css (if AUTO_EXPORT_CSS=true)

If `AUTO_EXPORT_CSS=true`, generate `export.css` now (before the completion report) using the same logic as Step 8D below, then note it in the completion report.

## Step 8: Completion Report & Next Action

Report the generated file path(s), mode used, and how to view it in the browser:
- HTML mode: start the dev server, then open `http://localhost:[PORT]/design-preview.html`
- Astro mode: run `astro dev`, then open `http://localhost:4321/design-preview`

If `AUTO_EXPORT_CSS=true`, also report that `export.css` was generated automatically.

Then use `AskUserQuestion` to ask what to do next:

```
What would you like to do next?

1. Import into Figma as a capture
   → Transfer the browser-rendered result to Figma as an image (generate_figma_design)

2. Generate as Figma nodes (/figma:figma-use)
   → Create a design system with components, variables, and auto-layout

3. Done

4. Write export.css
   → Export DESIGN.md tokens as a Tailwind CSS v4 @theme block
```

If `AUTO_EXPORT_CSS=true`, **omit option 4** and present only options 1–3 (AskUserQuestion options cannot be made unselectable) — the completion report already states that `export.css` was generated.

---

## Step 9: Follow-up Based on Selection

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

### If 4 is selected — generate export.css (Step 8D)

**Step 8D: Generate export.css**

Write `export.css` to the project root. The file maps all DESIGN.md tokens to a Tailwind CSS v4 `@theme` block. It is a standalone file meant as a starting point for manually integrating tokens into a new project — it does not `@import "tailwindcss"` itself.

Structure:

```css
/* Generated from DESIGN.md — copy into your project's main CSS and adjust as needed */

@theme {
  /* Colors */
  --color-[key]: [value];   /* one entry per colors token */

  /* Spacing */
  --spacing-[key]: [value]; /* one entry per spacing token */

  /* Border radius */
  --radius-[key]: [value];  /* one entry per rounded token */

  /* Typography */
  --font-[family-key]: [value];  /* font-family tokens */
  --text-[size-key]: [value];    /* font-size tokens */
}

/* Component defaults (not Tailwind utilities — use as reference) */
@layer components {
  /* [component-name]: [resolved token values as comments] */
}
```

**Token mapping rules:**

| DESIGN.md key | CSS variable | Example |
|---|---|---|
| `colors.primary` | `--color-primary` | `--color-primary: #1e40af;` |
| `spacing.xs` | `--spacing-xs` | `--spacing-xs: 4px;` |
| `rounded.sm` | `--radius-sm` | `--radius-sm: 4px;` |
| `typography.fontFamily.*` | `--font-*` | `--font-sans: "Inter", sans-serif;` |
| `typography.fontSize.*` | `--text-*` | `--text-sm: 0.875rem;` |
| `components.*` | `@layer components` comment block | for reference only |

- Resolve `{token.references}` to their actual values before writing.
- Use kebab-case for all variable names.
- Components go into an `@layer components {}` block as CSS comments (e.g. `/* button-primary: bg #1e40af, color #fff, padding 0.5rem 1rem */`) — they are reference material, not utility classes.

Overwrite any existing `export.css` without confirmation. Report the file path when done.
