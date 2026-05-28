# export-design-md — DESIGN.md Visual Export Skill

> **Experimental:** This skill is a work in progress. Behavior may change as the [Google design.md specification](https://github.com/google-labs-code/design.md) evolves.

A Claude Code skill that converts a `DESIGN.md` file into a blog-style visual HTML document. Detects Astro projects automatically and generates a `.astro` page instead. After generation, optionally exports the result to Figma.

---

## What this is

`DESIGN.md` is a machine-readable design specification encoding color tokens, typography, spacing, and component guidelines. It's useful for AI coding agents, but hard for humans to read at a glance.

This skill renders it into a visual design system reference — color swatches, typography scale samples, spacing bars, border-radius previews, and component docs — as a standalone HTML file that runs inside your existing project setup.

```
DESIGN.md  →  export-design-md  →  design-preview.html (or .astro)
                                           ↓ (optional)
                                       Figma capture / Figma nodes
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
    template.html     — default visual template
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

If no argument is provided, the skill uses the built-in `template.html`. The report is written in whatever language you are using in the conversation.

---

## After generation

Once the file is generated, the skill asks what to do next:

| Option | What happens |
|--------|--------------|
| **Import into Figma as a capture** | Opens the dev server, renders the page in browser, and sends it to Figma as an image via `generate_figma_design` |
| **Generate as Figma nodes** | Invokes `/figma:figma-use` with token data from DESIGN.md to build components, variables, and auto-layout |
| **Done** | Exits |

---

## Tips

- **`DESIGN.md` is required.** If the file doesn't exist, generate it first.
- **Color swatches always use inline styles.** Tailwind JIT won't generate classes for dynamic hex values — inline `style="background:#XXXXXX"` is intentional, not a bug.
- **Custom templates follow the default structure.** The skill uses your template's visual style and layout, then populates it with the actual DESIGN.md content. Keep the same section order (Header → Overview → Colors → Typography → Spacing → Shapes → Components → Do's & Don'ts → Footer) for best results.
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

---

## Related skills

- [create-design-md-skill](https://github.com/gaspanik/create-design-md-skill) — Generate `DESIGN.md` from your codebase or a Figma file

---

Built by Masaaki Komori - [@cipher](https://x.com/cipher) · Skill for [Claude Code](https://claude.ai/code)

