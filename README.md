# /explain

> ## ⚠️ Before you start
>
> Installing a Claude Code skill is just copying one folder into `~/.claude/skills/`. Your AI agent can do all of it. **The only human step:** after install, **restart Claude Code** so it picks up the new skill, then type `/explain`.


> An interactive HTML explainer for any concept, bug, or system — built right inside Claude Code. 12 design systems, 3 layouts, one self-contained file.

## What it does

`/explain` turns whatever you're talking about with Claude Code — a tricky bug, a data flow, a piece of architecture, an idea you're still circling — into a single, self-contained HTML file you can open in your browser and actually *look at*. Instead of a wall of text, you get a visual, interactive explainer: three meaningfully different visualizations of the same topic (e.g. a horizontal pipeline, a sequence diagram, and a detail catalog) with a built-in view switcher, plus a hamburger settings panel that lets you re-skin the whole thing across 12 design systems and flip between light and dark on the fly. No build step, no dependencies, no server — just one HTML file that opens in any browser.

It's part of [vibekit](https://github.com/brianharms), a showcase of small tools and Claude Code skills that make coding with AI feel better.

## Install

`/explain` is a Markdown-only skill — there's nothing to compile and no companion scripts to run. You just need its folder to live at `~/.claude/skills/explain/` so that `~/.claude/skills/explain/SKILL.md` exists.

Clone the repo and copy the folder into place:

```bash
git clone https://github.com/brianharms/skill-explain.git
mkdir -p ~/.claude/skills
cp -R skill-explain ~/.claude/skills/explain
```

Or, if you downloaded the repo as a ZIP, unzip it and copy the contents so the layout matches:

```bash
~/.claude/skills/explain/SKILL.md
```

That's it. Open (or restart) Claude Code and invoke the skill by typing:

```
/explain
```

## Usage

Invoke `/explain` with a topic, or with nothing at all to explain whatever you've been discussing:

```
/explain how the camera sync works
```

```
/explain this bug
```

```
/explain the WebSocket lifecycle
```

What happens next:

1. Claude distills the subject into its key entities, relationships, and the "aha" moment.
2. It designs **three** genuinely different visualizations (not three reskins of the same diagram) and builds **all of them** into one HTML file behind an A/B/C view switcher.
3. It writes the file to `/tmp/explain-<slugified-topic>.html` — a single file with all CSS and JS inline, zero external dependencies.
4. It opens the file in your browser with `open`.

In the browser you get:

- An **A/B/C view switcher** to flip between the three layouts. Your active view persists across reloads.
- A **hamburger menu** (top-right) with a **light/dark mode toggle** and a **style switcher** spanning 12 design systems: Teenage Engineering (default), Claude, Apple, Google, Hims, Bungie, Supreme, MUJI, Porsche, NASA, Nothing, and Linear.
- Instant theming — switching style or mode re-applies every CSS variable live, with no reload. Your style and mode choices persist via `localStorage`.

## Requirements / Dependencies

- **macOS** — the final step opens the explainer with the macOS `open` command. On other platforms the HTML file is still generated identically at `/tmp/explain-<slug>.html`; you'd just open it manually.
- **[Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)** — this is a Claude Code skill and runs inside it.
- **A web browser** — to view the generated HTML (any modern browser; no extensions or network access needed).

No npm packages, no MCP servers, no sibling skills required. The generated explainer has no runtime dependencies of its own.

## For AI coding agents

If you're an agent working *on* this skill, here's the orientation.

### Repo layout

```
skill-explain/
├── SKILL.md      # the entire skill — instructions Claude reads at invoke time
├── LICENSE       # MIT
├── .gitignore
└── README.md
```

This is a **pure Markdown skill**. There are no scripts, no `web/`, no `swift/`, and no `template.html` — everything the skill does is encoded as instructions in `SKILL.md`. Don't add a build step or runtime; the whole point is that it ships as text Claude follows.

### What SKILL.md is

`SKILL.md` is the contract. When a user types `/explain`, Claude Code loads `SKILL.md` and follows it verbatim to produce the HTML. The YAML frontmatter (`name`, `description`, `argument-hint`, `user-invocable`) controls how the skill is discovered and triggered; the body controls what gets built. Editing the body changes the output. Treat it like source code.

### How to test changes

There's no test harness — you verify by running the skill end to end:

```bash
cp -R . ~/.claude/skills/explain    # install your working copy
```

Then in Claude Code, invoke `/explain <some topic>` and inspect the file written to `/tmp/explain-<slug>.html` in a browser. Click through all three views, open the hamburger menu, cycle every style, and toggle light/dark. The skill's own **Quality Checklist** (bottom of `SKILL.md`) is the acceptance criteria — run against it.

### Invariants — do not break these

- **The 12 styles are a contract.** The `STYLES` object in the generated HTML must contain all twelve named styles (`teenage-engineering`, `claude`, `apple`, `google`, `hims`, `bungie`, `supreme`, `muji`, `porsche`, `nasa`, `nothing`, `linear`), each with `shared`, `dark`, and `light` palettes. If you add or rename a style here, update the README's style list and the count ("12 design systems") to match.
- **Teenage Engineering is the default**, dark is the default mode. Keep `:root` in the generated HTML seeded with the Teenage Engineering dark values.
- **The CSS Variable Contract is mandatory.** Every style-dependent rule in the generated HTML must use the variables in the `:root` contract (`--bg`, `--surface`, `--accent`, `--font-primary`, etc.). Zero hardcoded colors or fonts. This is what makes live theming work.
- **Three real views, one file.** Always build three meaningfully different visualizations into a single self-contained HTML file with an A/B/C switcher — never propose them in chat and wait, never emit external files or dependencies.
- **localStorage keys are stable.** Persistence relies on `explain-view` (`'a'|'b'|'c'`), `explain-style` (style key), and `explain-mode` (`'light'|'dark'`). Don't rename these.
- **Output path and open step.** Write to `/tmp/explain-<slugified-topic>.html` and finish by opening it. If you broaden platform support, keep the `/tmp` single-file output identical so macOS behavior is unchanged.
- **The layout/interaction rules are load-bearing**, not decoration. The viewport-awareness, no-scroll-to-discover, all-steps-visible, and no-text-overlap rules in `SKILL.md` exist because they keep the output usable. Don't relax them.

## License

MIT © 2026 Brian Harms / Ritual Industries — [ritual.industries](https://ritual.industries)
