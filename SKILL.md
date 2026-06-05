---
name: explain
description: "Create a visual, interactive HTML explainer for a concept, bug, system, or idea from the current conversation. Multi-style design with runtime style and light/dark switcher."
user-invocable: true
argument-hint: "[topic to explain, e.g., 'how the camera sync works', 'this bug', 'WebSocket lifecycle']"
---

# Visual Explainer Skill

Create a single self-contained HTML file that visually explains the current topic — a bug, system, concept, architecture, data flow, or idea.

## Step 1: Identify the Subject

From the `/explain` argument or recent conversation context. Distill into: key entities, relationships/flows, and the "aha" moment.

## Step 2: Design 3 Visualization Architectures

Pick 3 meaningfully different visual approaches to explain the topic. Each should use a different layout paradigm — don't just reskin the same diagram three times. Good variety examples:

- **Horizontal pipeline** vs **sequence diagram with swim lanes** vs **split overview + detail catalog**
- **Flowchart** vs **state machine** vs **annotated code walkthrough**
- **Architecture diagram** vs **timeline** vs **layered stack**

Each approach should have:
1. A short **title** (e.g., "Pipeline", "Sequence", "Catalog")
2. A distinct **layout and interaction model**

### Build All 3 Into the HTML

Do NOT propose the 3 approaches in chat and wait. Instead, **build all 3 visualizations into a single HTML file** with a view switcher that lets the user toggle between them.

#### View Switcher UI
- **Position**: Fixed bar at top-left or top-center, above the diagram content, below the header
- **Style**: Segmented control or pill-group with 3 options (A, B, C) showing the approach title
- **Active state**: `var(--accent)` background or underline, clear visual distinction
- **Behavior**: Clicking a view instantly swaps the `<main>` content. No page reload, no scroll reset.
- **Persistence**: Store active view in `localStorage.setItem('explain-view', 'a'|'b'|'c')`
- **Default**: Start on the view you think is best (your recommendation)

#### Implementation
- Each visualization lives in its own container: `<div class="view-content" data-view="a">`, `data-view="b"`, `data-view="c"`
- Only the active view is `display: block`, others are `display: none`
- All 3 views share the same CSS variable system (styles/modes apply to all views)
- Each view is a complete, self-contained visualization — not a partial or degraded version

## Step 2.5: Layout & Interaction Design (CRITICAL)

The user is a visual learner looking at this in a browser. Every layout and interaction decision must respect how someone actually uses a screen. Follow these rules:

### Viewport Awareness
- **Everything important must be visible without scrolling** at 1200px+ width. If content exceeds one screen, use spatial layout (horizontal flow, grid, tabs) rather than vertical stacking.
- **Never put controls at the top and content that scrolls below them.** If the user scrolls down to read step 6, they shouldn't lose access to navigation. Controls must either be sticky/fixed, or inline with the content.
- **Prefer horizontal progression** over vertical lists for sequential content. A left-to-right pipeline that fits in one viewport beats a tall vertical list every time.

### Step-by-Step / Sequential Content
- **Show all steps at once** when there are ≤8 steps. Use a horizontal rail, connected pipeline, or compact grid where each step is a card. Highlight the active one, dim the others — but the user can always see the full picture.
- **For >8 steps**, use a scrollable horizontal track with the active step centered, or a minimap/progress bar that shows position in the sequence.
- **Never require prev/next buttons as the ONLY way to navigate.** If you have step controls, they are a convenience layer ON TOP of a layout where all steps are already visible. The user should be able to click directly on any step.
- **If using prev/next buttons**, they must be `position: sticky` or `position: fixed` so they're always reachable. Place them at the horizontal edges of the viewport, not above or below scrollable content.

### Interaction Patterns That Work
- **Click-to-highlight on a visible layout** — all nodes/steps visible, clicking one zooms in or reveals detail in a side panel or overlay. The context is never lost.
- **Hover to reveal detail** — base diagram shows the overview, hover enriches with specifics. Zero navigation needed.
- **Accordion within a fixed frame** — expand one section, others collapse, total height stays constant. No scroll surprise.
- **Tabs or segmented control at the top** — switch between views of the same data. All content swaps within a fixed-height container.
- **Connected timeline rail** — horizontal line with nodes, click to expand detail below or beside, rail stays visible.

### Interaction Anti-Patterns (NEVER DO THESE)
- Vertical list of steps that grows beyond the viewport with navigation buttons stuck at the top
- Modals or overlays that obscure the diagram being explained
- Scroll-dependent reveals where the user has to scroll to discover content exists
- Prev/next as the only navigation when steps are not otherwise visible
- Tiny text or cramped layouts — if you need more space, use the full viewport width

### The Gut Check
Before finalizing the layout, imagine the user opens this in Chrome at 1400px wide. Can they:
1. See the entire concept at a glance without scrolling? (ideal)
2. Understand the structure immediately — what's clickable, what's informational?
3. Navigate to any part of the explanation in one click?
4. Never lose their place or the navigation controls?

If any answer is no, redesign the layout.

## Step 3: Build the HTML File

Write to: `/tmp/explain-<slugified-topic>.html` — single file, no external dependencies, all CSS/JS inline.

---

## CSS Variable Contract

ALL style-dependent CSS MUST use these variables. Never hardcode colors or fonts in component styles.

```css
:root {
  --bg; --surface; --surface-alt; --border;
  --text; --text-dim; --accent; --accent-dim;
  --success; --error; --info;
  --font-primary; --font-mono;
  --font-weight-normal; --font-weight-bold;
  --letter-spacing-label; --letter-spacing-body;
  --label-transform;
  --radius; --radius-small;
  --glow-color; --transition-speed;
}
```

---

## Style Definitions

Each style has `shared` properties (typography, shape) and `dark`/`light` color palettes. The JS `STYLES` object must contain ALL of these.

### Teenage Engineering (default)
```
shared:
  font-primary: 'SF Mono', 'JetBrains Mono', 'Fira Code', monospace
  font-mono: 'SF Mono', 'JetBrains Mono', 'Fira Code', monospace
  font-weight-normal: 400 | font-weight-bold: 600
  letter-spacing-label: 0.05em | letter-spacing-body: 0.02em
  label-transform: uppercase
  radius: 2px | radius-small: 1px
  transition-speed: 150ms
dark:
  bg: #0d0d0d | surface: #1a1a1a | surface-alt: #242424 | border: #333333
  text: #e0e0e0 | text-dim: #888888
  accent: #ff6600 | accent-dim: #cc5200
  success: #00cc66 | error: #ff3333 | info: #4da6ff
  glow-color: rgba(255,102,0,0.3)
light:
  bg: #f2f0ed | surface: #ffffff | surface-alt: #e8e5e0 | border: #cccccc
  text: #1a1a1a | text-dim: #777777
  accent: #ff6600 | accent-dim: #cc5200
  success: #00994d | error: #cc2200 | info: #2277cc
  glow-color: rgba(255,102,0,0.2)
```

### Claude
Warm, intellectual. Sand/cream with terracotta. Humanist serif typography.
```
shared:
  font-primary: Georgia, 'Times New Roman', serif
  font-mono: 'SF Mono', Menlo, monospace
  font-weight-normal: 400 | font-weight-bold: 700
  letter-spacing-label: 0.03em | letter-spacing-body: 0.01em
  label-transform: none
  radius: 8px | radius-small: 4px
  transition-speed: 200ms
dark:
  bg: #1c1917 | surface: #292524 | surface-alt: #34302c | border: #4a4540
  text: #f5f0e8 | text-dim: #9a9088
  accent: #d97757 | accent-dim: #b8604a
  success: #5a9a6e | error: #c4503a | info: #5b7fa6
  glow-color: rgba(217,119,87,0.25)
light:
  bg: #f5f0e8 | surface: #ffffff | surface-alt: #ebe5d9 | border: #d4cabb
  text: #2d2a26 | text-dim: #8a8279
  accent: #d97757 | accent-dim: #b8604a
  success: #5a9a6e | error: #c4503a | info: #5b7fa6
  glow-color: rgba(217,119,87,0.2)
```

### Apple
Precise, minimal, premium. SF Pro, blue accent, obsessive spacing.
```
shared:
  font-primary: -apple-system, 'SF Pro Display', 'Helvetica Neue', sans-serif
  font-mono: 'SF Mono', Menlo, monospace
  font-weight-normal: 400 | font-weight-bold: 600
  letter-spacing-label: 0.02em | letter-spacing-body: -0.01em
  label-transform: none
  radius: 12px | radius-small: 8px
  transition-speed: 250ms
dark:
  bg: #000000 | surface: #1c1c1e | surface-alt: #2c2c2e | border: #38383a
  text: #f5f5f7 | text-dim: #86868b
  accent: #0a84ff | accent-dim: #0070e0
  success: #30d158 | error: #ff453a | info: #64d2ff
  glow-color: rgba(10,132,255,0.25)
light:
  bg: #ffffff | surface: #f5f5f7 | surface-alt: #e8e8ed | border: #d2d2d7
  text: #1d1d1f | text-dim: #86868b
  accent: #0071e3 | accent-dim: #005bb5
  success: #28a745 | error: #ff3b30 | info: #5ac8fa
  glow-color: rgba(0,113,227,0.15)
```

### Google
Material Design 3. Systematic, tonal surfaces, rounded containers.
```
shared:
  font-primary: 'Google Sans', Roboto, 'Segoe UI', sans-serif
  font-mono: 'Roboto Mono', 'Fira Code', monospace
  font-weight-normal: 400 | font-weight-bold: 500
  letter-spacing-label: 0.02em | letter-spacing-body: 0.01em
  label-transform: none
  radius: 16px | radius-small: 8px
  transition-speed: 200ms
dark:
  bg: #1f1f1f | surface: #2b2b2b | surface-alt: #333333 | border: #444444
  text: #e3e3e3 | text-dim: #9aa0a6
  accent: #8ab4f8 | accent-dim: #669df6
  success: #81c995 | error: #f28b82 | info: #78d9ec
  glow-color: rgba(138,180,248,0.2)
light:
  bg: #f8f9fa | surface: #ffffff | surface-alt: #f1f3f4 | border: #dadce0
  text: #202124 | text-dim: #5f6368
  accent: #1a73e8 | accent-dim: #1557b0
  success: #34a853 | error: #ea4335 | info: #4285f4
  glow-color: rgba(26,115,232,0.15)
```

### Hims
Muted, organic, wellness editorial. Sage and cream, soft serif.
```
shared:
  font-primary: 'Playfair Display', Georgia, serif
  font-mono: 'DM Mono', 'Courier New', monospace
  font-weight-normal: 400 | font-weight-bold: 700
  letter-spacing-label: 0.08em | letter-spacing-body: 0.01em
  label-transform: uppercase
  radius: 4px | radius-small: 2px
  transition-speed: 300ms
dark:
  bg: #1a1f1a | surface: #242b24 | surface-alt: #2d352d | border: #3d453d
  text: #e0e4db | text-dim: #8a917f
  accent: #7d9b7e | accent-dim: #5f7d60
  success: #7d9b7e | error: #c47a6e | info: #7a9eb5
  glow-color: rgba(125,155,126,0.25)
light:
  bg: #f7f5f0 | surface: #ffffff | surface-alt: #eee9df | border: #d5cfc3
  text: #2a2a2a | text-dim: #8a8a7a
  accent: #7d9b7e | accent-dim: #5f7d60
  success: #7d9b7e | error: #c47a6e | info: #7a9eb5
  glow-color: rgba(125,155,126,0.2)
```

### Bungie (Marathon)
Cyberpunk, high-contrast, angular, neon on black.
```
shared:
  font-primary: Impact, 'Arial Narrow', 'Helvetica Neue', sans-serif
  font-mono: 'Courier New', monospace
  font-weight-normal: 400 | font-weight-bold: 900
  letter-spacing-label: 0.12em | letter-spacing-body: 0.03em
  label-transform: uppercase
  radius: 0px | radius-small: 0px
  transition-speed: 100ms
dark:
  bg: #08080a | surface: #111114 | surface-alt: #1a1a1f | border: #2a2a33
  text: #eaeaef | text-dim: #6a6a77
  accent: #ff2a6d | accent-dim: #cc1f57
  success: #00f0b5 | error: #ff2a2a | info: #00d4ff
  glow-color: rgba(255,42,109,0.35)
light:
  bg: #f0f0f3 | surface: #ffffff | surface-alt: #e5e5ea | border: #c5c5d0
  text: #111114 | text-dim: #6a6a77
  accent: #e01555 | accent-dim: #b01045
  success: #00b888 | error: #d41a1a | info: #0099cc
  glow-color: rgba(224,21,85,0.2)
```

### Supreme
Brutalist. Futura Heavy Oblique, red box, stark contrast, zero decoration.
```
shared:
  font-primary: 'Futura', 'Trebuchet MS', 'Arial Black', sans-serif
  font-mono: 'Courier New', monospace
  font-weight-normal: 700 | font-weight-bold: 900
  letter-spacing-label: 0.06em | letter-spacing-body: 0.02em
  label-transform: uppercase
  radius: 0px | radius-small: 0px
  transition-speed: 0ms
dark:
  bg: #0a0a0a | surface: #161616 | surface-alt: #1e1e1e | border: #333333
  text: #ffffff | text-dim: #888888
  accent: #e51e2c | accent-dim: #b81823
  success: #ffffff | error: #e51e2c | info: #ffffff
  glow-color: rgba(229,30,44,0.3)
light:
  bg: #ffffff | surface: #f5f5f5 | surface-alt: #eeeeee | border: #cccccc
  text: #0a0a0a | text-dim: #666666
  accent: #e51e2c | accent-dim: #b81823
  success: #0a0a0a | error: #e51e2c | info: #0a0a0a
  glow-color: rgba(229,30,44,0.15)
```

### MUJI
Extreme reduction. Barely-there earth tones, negative space IS the design. Whisper-quiet.
```
shared:
  font-primary: 'Helvetica Neue', Helvetica, Arial, sans-serif
  font-mono: 'Menlo', 'Courier New', monospace
  font-weight-normal: 300 | font-weight-bold: 400
  letter-spacing-label: 0.1em | letter-spacing-body: 0.02em
  label-transform: uppercase
  radius: 0px | radius-small: 0px
  transition-speed: 400ms
dark:
  bg: #1a1917 | surface: #23221f | surface-alt: #2b2a27 | border: #3a3935
  text: #c8c4bc | text-dim: #7a7770
  accent: #a89888 | accent-dim: #8a7d6f
  success: #8a9a80 | error: #b08080 | info: #8090a0
  glow-color: rgba(168,152,136,0.15)
light:
  bg: #f7f5f2 | surface: #ffffff | surface-alt: #f0ede8 | border: #e0ddd6
  text: #3a3835 | text-dim: #a09a90
  accent: #8a7d6f | accent-dim: #6e6358
  success: #7a8a70 | error: #a07070 | info: #708090
  glow-color: rgba(138,125,111,0.1)
```

### Porsche
Automotive precision. Silver/black/red, engineered luxury. Clean and authoritative.
```
shared:
  font-primary: 'Helvetica Neue', 'Arial', sans-serif
  font-mono: 'SF Mono', 'Menlo', monospace
  font-weight-normal: 400 | font-weight-bold: 700
  letter-spacing-label: 0.06em | letter-spacing-body: 0.01em
  label-transform: uppercase
  radius: 4px | radius-small: 2px
  transition-speed: 180ms
dark:
  bg: #0c0c0c | surface: #171717 | surface-alt: #212121 | border: #3a3a3a
  text: #e8e8e8 | text-dim: #8a8a8a
  accent: #d5001c | accent-dim: #aa0016
  success: #00a37a | error: #d5001c | info: #b0b0b0
  glow-color: rgba(213,0,28,0.25)
light:
  bg: #f4f4f4 | surface: #ffffff | surface-alt: #eaeaea | border: #d0d0d0
  text: #1a1a1a | text-dim: #6a6a6a
  accent: #d5001c | accent-dim: #aa0016
  success: #008060 | error: #d5001c | info: #555555
  glow-color: rgba(213,0,28,0.15)
```

### NASA
Mission control. Technical readouts, system labels, data-forward. Trust and precision.
```
shared:
  font-primary: 'Helvetica Neue', 'Roboto', sans-serif
  font-mono: 'SF Mono', 'Courier New', monospace
  font-weight-normal: 400 | font-weight-bold: 700
  letter-spacing-label: 0.08em | letter-spacing-body: 0.02em
  label-transform: uppercase
  radius: 2px | radius-small: 1px
  transition-speed: 120ms
dark:
  bg: #0b1026 | surface: #121a36 | surface-alt: #1a2444 | border: #2a3558
  text: #d0d8e8 | text-dim: #6878a0
  accent: #fc3d21 | accent-dim: #cc3018
  success: #22dd66 | error: #fc3d21 | info: #4dc9f6
  glow-color: rgba(252,61,33,0.3)
light:
  bg: #eef0f5 | surface: #ffffff | surface-alt: #e0e4ed | border: #c0c8d8
  text: #0b1026 | text-dim: #5a6580
  accent: #e03010 | accent-dim: #b02808
  success: #1aaa50 | error: #e03010 | info: #2070c0
  glow-color: rgba(224,48,16,0.15)
```

### Nothing
Dot matrix, retro-futuristic. Red on black, mechanical/digital hybrid. Carl Pei's vision.
```
shared:
  font-primary: 'Space Mono', 'SF Mono', 'Courier New', monospace
  font-mono: 'Space Mono', 'SF Mono', 'Courier New', monospace
  font-weight-normal: 400 | font-weight-bold: 700
  letter-spacing-label: 0.08em | letter-spacing-body: 0.04em
  label-transform: uppercase
  radius: 0px | radius-small: 0px
  transition-speed: 80ms
dark:
  bg: #000000 | surface: #0a0a0a | surface-alt: #141414 | border: #2a2a2a
  text: #e0e0e0 | text-dim: #555555
  accent: #ff0000 | accent-dim: #cc0000
  success: #00ff00 | error: #ff0000 | info: #e0e0e0
  glow-color: rgba(255,0,0,0.3)
light:
  bg: #f5f5f5 | surface: #ffffff | surface-alt: #eaeaea | border: #d0d0d0
  text: #0a0a0a | text-dim: #888888
  accent: #dd0000 | accent-dim: #aa0000
  success: #008800 | error: #dd0000 | info: #333333
  glow-color: rgba(221,0,0,0.15)
```

### Linear
Developer precision. Purple accent, clean dark UI, sharp and focused.
```
shared:
  font-primary: 'Inter', -apple-system, 'Segoe UI', sans-serif
  font-mono: 'JetBrains Mono', 'SF Mono', monospace
  font-weight-normal: 400 | font-weight-bold: 600
  letter-spacing-label: 0.03em | letter-spacing-body: 0em
  label-transform: none
  radius: 8px | radius-small: 4px
  transition-speed: 150ms
dark:
  bg: #111113 | surface: #1b1b1f | surface-alt: #232329 | border: #33333a
  text: #eeeeef | text-dim: #7a7a85
  accent: #8b5cf6 | accent-dim: #7340e0
  success: #34d399 | error: #ef4444 | info: #60a5fa
  glow-color: rgba(139,92,246,0.25)
light:
  bg: #f9fafb | surface: #ffffff | surface-alt: #f1f2f4 | border: #e0e1e6
  text: #111113 | text-dim: #6b7280
  accent: #7c3aed | accent-dim: #6525d0
  success: #10b981 | error: #ef4444 | info: #3b82f6
  glow-color: rgba(124,58,237,0.15)
```

---

## Hamburger Menu (Settings Panel)

Every explainer MUST include a hamburger menu in the **top-right corner**. Persistent, floats above content.

### Layout
- **Icon**: Three horizontal lines (CSS-only), `var(--text-dim)`, 24x24px hit target
- **Position**: `position: fixed; top: 16px; right: 16px; z-index: 9999`
- **Content padding**: All page content (headers, titles, text) must have `padding-right: 60px` or equivalent to prevent text from overlapping the hamburger icon area. No text should ever render behind or under the hamburger button.
- **Panel**: Slides in from right. `var(--surface)` bg, `var(--border)` border, `var(--radius)` corners
- **Close**: Click hamburger again or click outside

### Menu Contents

**1. Mode Toggle (top of panel)**
- Simple toggle switch: light / dark
- Label: `MODE`
- Toggling swaps the color palette while keeping the same style
- Store in `localStorage.setItem('explain-mode', 'light'|'dark')`

**2. Style Switcher**
- Label: `STYLE`
- Grid or list of all style names
- Active style gets `var(--accent)` color + indicator
- Clicking instantly swaps all CSS variables
- Store in `localStorage.setItem('explain-style', key)`

**3. Future Settings Slot**
- Separator + JS comment: `// Future settings go here`

### JS Implementation

```js
const STYLES = {
  "teenage-engineering": {
    label: "Teenage Engineering",
    shared: { /* font, spacing, radius, transition vars */ },
    dark: { /* color vars */ },
    light: { /* color vars */ }
  },
  // ... all 12 styles
};

let currentStyle = localStorage.getItem('explain-style') || 'teenage-engineering';
let currentMode = localStorage.getItem('explain-mode') || 'dark';

function applyTheme() {
  const style = STYLES[currentStyle];
  const root = document.documentElement;
  // Apply shared vars
  Object.entries(style.shared).forEach(([k, v]) => root.style.setProperty(k, v));
  // Apply mode-specific vars
  Object.entries(style[currentMode]).forEach(([k, v]) => root.style.setProperty(k, v));
  // Update UI indicators
  document.querySelectorAll('.style-option').forEach(el =>
    el.classList.toggle('active', el.dataset.style === currentStyle));
  document.getElementById('mode-toggle')?.classList.toggle('light', currentMode === 'light');
  localStorage.setItem('explain-style', currentStyle);
  localStorage.setItem('explain-mode', currentMode);
}

function setStyle(key) { currentStyle = key; applyTheme(); }
function toggleMode() { currentMode = currentMode === 'dark' ? 'light' : 'dark'; applyTheme(); }
```

---

## Page Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EXPLAIN — [Topic]</title>
    <style>
      :root { /* Teenage Engineering dark vars as defaults */ }
      * { margin: 0; padding: 0; box-sizing: border-box; }
      body {
        background: var(--bg); color: var(--text);
        font-family: var(--font-primary);
        transition: background var(--transition-speed) ease, color var(--transition-speed) ease;
      }
      /* ALL styles use var() — zero hardcoded values */
    </style>
</head>
<body>
    <!-- Hamburger + settings panel -->
    <!-- Header with EXPLAIN label + topic -->
    <!-- View switcher: segmented control with 3 visualization options -->
    <!-- Context (optional) -->
    <!-- Main diagram area: 3 view containers, only active one visible -->
    <div class="view-content active" data-view="a"><!-- View A --></div>
    <div class="view-content" data-view="b"><!-- View B --></div>
    <div class="view-content" data-view="c"><!-- View C --></div>
    <!-- Takeaways (optional) -->
    <script>
      const STYLES = { /* all 12 styles with shared/dark/light */ };
      // Theme engine + menu logic
    </script>
</body>
</html>
```

---

## Design Standards

Hold yourself to a high design standard. These are not optional:

- **Text must never overlap other text.** No element's text content should visually collide with any other text, label, badge, icon, or the hamburger button. If a container is too small for its text, truncate with `text-overflow: ellipsis`, reduce font size, or restructure the layout — never let text bleed over or under another element.
- **Ensure adequate spacing between all elements.** Labels, badges, titles, and body text must have clear visual separation. Use `gap`, `margin`, and `padding` intentionally — not as afterthoughts.
- **Test mental model at every breakpoint.** At 1200px, 1400px, and 1800px widths, all text must remain legible, separated, and non-overlapping. Cards and containers must not crush their content.
- **Badges and indicators must not overlap titles.** If a box has both a title and a badge (e.g., bug indicator), ensure the title has enough `padding-right` to clear the badge. Use `position: absolute` badges only when the parent has explicit space reserved.
- **Responsive text sizing.** If a label is too long for its container, use `overflow: hidden; text-overflow: ellipsis; white-space: nowrap` rather than letting it wrap into adjacent elements.

## Quality Checklist

- [ ] Diagram tells the story alone — visual-first explanation
- [ ] No external dependencies — fully self-contained
- [ ] Works at 1200px+ viewport
- [ ] ALL colors/fonts use CSS variables — zero hardcoded values
- [ ] All 12 styles render correctly with no layout breakage
- [ ] Light and dark modes both have readable contrast
- [ ] Hamburger menu present with mode toggle + style list
- [ ] **3 visualization views built in** with view switcher (A/B/C segmented control)
- [ ] Each view is a meaningfully different layout, not a reskin
- [ ] Active view persists via localStorage
- [ ] Preferences persist via localStorage
- [ ] Animations only when they teach something
- [ ] **No text overlaps any other text** — verified across all 3 views and all styles
- [ ] **Badges/indicators have reserved space** — titles pad around them, never under them

## Step 4: Open in Browser

```bash
open /tmp/explain-<slug>.html
```

One short sentence to the user about what was explained.
