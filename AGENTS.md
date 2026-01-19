## Project

This repo is a Slidev deck about **Ast-Grep** for the **Paraflow** team/project, plus a light intro to compiler frontend concepts (lexer/parser → AST).

## Target

- Keep the deck centered on *practical usage* in **Paraflow** (Motiff AI is a previous product; avoid presenting it as current).
- Structure: (1) How I use Ast-Grep (2) In which we need it (3) Lexer/Parser empowers Ast-Grep.

## Tooling

- Use **Bun** (not pnpm/npm) for install/build/dev:
  - `bun install`
  - `bun run dev`
  - `bun run build`

## Slide Structure

- Main entry: `slides.md`
- Sections live in `pages/` and are imported from `slides.md` via `src: ./pages/...`.
- Keep content **short** on slides; put details in speaker notes (`<!-- ... -->`) when needed.
- Detail slides can be left as `TODO` placeholders until content is ready.

## Markdown Rules (Slidev)

- Slide separators: `---` creates a *new slide* (don’t use it inside `slides.md` unless you really mean a new slide).
- Line breaks in text/notes:
  - New paragraph: leave a **blank line** between sentences/blocks.
  - Forced line break in one paragraph: end the line with **two spaces** then newline, or use `<br/>`.
- Speaker notes live in HTML comments: `<!-- ... -->`. Keep bilingual notes in the same block (no `---` divider); separate languages with blank lines.

## Style Conventions

- Prefer modern Slidev features: layouts (`cover`, `section`, `image-right`, `two-cols`, `fact`, `quote`), `v-click`, Mermaid diagrams, MDC syntax.
- Use Unsplash backgrounds via `https://source.unsplash.com/1600x900/?<keywords>` (pick keywords relevant to the section).
- Keep each section opening slide as a clear “startup” (title + one-line purpose).

## Presenter Timing

- Set overall timing with top-level `duration:` and (optionally) `timer: countdown`.
- Per-slide target time can be set with slide frontmatter `time: 45s` (or `pace: 45s`); shown in presenter view via `slide-bottom.vue`.
