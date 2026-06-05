# export-design-md — DESIGN.md Visual Export Skill

> **Experimental:** This skill is a work in progress. Behavior may change as the [Google design.md specification](https://github.com/google-labs-code/design.md) evolves.

A Claude Code skill that converts a `DESIGN.md` file into a blog-style visual HTML document. Detects Astro projects automatically and generates a `.astro` page instead. When no main CSS entry point is found, also auto-generates an `export.css` with Tailwind CSS v4 `@theme` tokens. After generation, optionally exports the result to Figma.

---

## What this is

`DESIGN.md` is a machine-readable design specification encoding color tokens, typography, spacing, and component guidelines. It's useful for AI coding agents, but hard for humans to read at a glance.

This skill renders it into a visual design system reference — color swatches, typography scale samples, spacing bars, border-radius previews, and component docs — as a standalone HTML file that runs inside your existing project setup.

```
DESIGN.md  →  export-design-md  →  design-preview.html (or .astro)
                                  ↓ (auto, if no main CSS found)    ↓ (optional)
                               export.css (@theme tokens)    Figma capture / Figma nodes
```

---

## Requirements

- **`DESIGN.md`** must exist in the project root
- **Tailwind CSS** must be present in `package.json` (the skill uses Tailwind classes for layout; color/typography samples use inline styles for reliability)
- **Dev server with module script support** — the skill detects your project's entry point from `index.html` (or `src/`) and reuses it in the generated file. Vite is the primary tested environment.

---

## Framework modes

| Mode | When it activates | Output |
|------|-------------------|--------|
| **HTML mode** | Default (no Astro detected) | `design-preview.html` in project root |
| **Astro mode** | `"astro"` found in `package.json` | `src/pages/design-preview.astro` |

In Astro mode, the skill inspects existing pages and layouts to match the project's conventions (layout imports, font tags, class patterns).

---

## Repo structure

```
skills/
  export-design-md/
    SKILL.md          — skill definition loaded by Claude Code
    template.html     — default visual template (HTML mode)
    template.astro    — default visual template (Astro mode)
```

---

## Getting started

**1. Clone this repo**

```bash
git clone https://github.com/gaspanik/export-design-md-skill
```

**2. Install the skill into Claude Code**

```bash
cp -r skills/export-design-md ~/.claude/skills/
```

**3. Run the skill**

```
/export-design-md
```

With a custom template:

```
/export-design-md path/to/my-template.html
```

Or in natural language:

```
DESIGN.md をビジュアルドキュメントに変換して
```

If no argument is provided, the skill uses the built-in `template.html` (HTML mode) or `template.astro` (Astro mode). The report is written in whatever language you are using in the conversation.

---

## export.css — Tailwind CSS v4 token export

When the skill detects no main CSS entry point (e.g. `DESIGN.md` was imported from Figma or an external source rather than generated from an existing project), it automatically writes `export.css` to the project root alongside the preview file.

`export.css` is a **standalone starting point** for manually integrating design tokens into a new project. It does not `@import "tailwindcss"` — copy the relevant blocks into your actual CSS file.

```css
/* Generated from DESIGN.md — copy into your project's main CSS and adjust as needed */

@theme {
  --color-primary: #1e40af;
  --color-secondary: #7c3aed;
  --spacing-xs: 4px;
  --radius-sm: 4px;
  --font-sans: "Inter", sans-serif;
  /* … */
}

/* Component defaults (not Tailwind utilities — use as reference) */
@layer components {
  /* button-primary: bg #1e40af, color #fff, padding 0.5rem 1rem */
}
```

Even when a main CSS file exists, you can generate `export.css` on demand from the post-preview menu (option 4).

---

## After generation

Once the file is generated, the skill asks what to do next:

| Option | What happens |
|--------|--------------|
| **Import into Figma as a capture** | Opens the dev server, renders the page in browser, and sends it to Figma as an image via `generate_figma_design` |
| **Generate as Figma nodes** | Invokes `/figma:figma-use` with token data from DESIGN.md to build components, variables, and auto-layout |
| **Done** | Exits |
| **Write export.css** | Generates `export.css` with Tailwind CSS v4 `@theme` tokens (shown as "already generated" if auto-generated) |

---

## Tips

- **`DESIGN.md` is required.** If the file doesn't exist, generate it first.
- **Color swatches always use inline styles.** Tailwind JIT won't generate classes for dynamic hex values — inline `style="background:#XXXXXX"` is intentional, not a bug.
- **Custom templates follow the default structure.** The skill copies your template verbatim and replaces `[placeholder]` values with actual DESIGN.md content. Keep the same section order (Header → Overview → Colors → Typography → Spacing → Shapes → Components → Do's & Don'ts → Footer) for best results. Pass a `.astro` file for Astro projects.
- **Figma capture requires the Figma MCP server.** Connect the official [Figma MCP server](https://github.com/figma/mcp-server-guide) in Claude Code settings before selecting the Figma export option.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| "DESIGN.md not found" | Generate `DESIGN.md` first before running this skill. |
| "Tailwind CSS not found" | This skill requires a project with Tailwind CSS. Add it to your project, or generate the HTML manually. |
| Color swatches not rendering | Ensure `design-preview.html` is served via dev server, not opened as a local file (Tailwind needs the dev pipeline). |
| Astro page has wrong layout | The skill reads your existing pages to infer the layout. If the result looks off, pass a custom template or edit the generated file. |
| Figma capture returns no result | Ensure the dev server is running and the Figma MCP server is connected in Claude Code settings. |
| `export.css` not auto-generated | The skill looks for `@import "tailwindcss"` or `@tailwind base` in CSS files under `src/`, `public/`, or the project root. If your main CSS uses a different pattern, use the manual option 4 from the post-preview menu. |

---

## Related skills

- [create-design-md-skill](https://github.com/gaspanik/create-design-md-skill) — Generate `DESIGN.md` from your codebase or a Figma file

---

Built by Masaaki Komori - [@cipher](https://x.com/cipher) · Skill for [Claude Code](https://claude.ai/code)

