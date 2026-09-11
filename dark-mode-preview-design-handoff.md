# Handoff: HTML email dark mode preview, visual design

Design-only supplement to `dark-mode-preview-handoff.md` (the technical/mechanism handoff). That doc covers the CSS-extraction logic, this one covers everything visual: same Intopia identity as the tester and the EML converter, adapted to a comparison-viewer layout rather than a results screen.

## Design tokens, verbatim from the tester's live source

```css
:root {
  --ink: #16233d;
  --ink-soft: #4a5875;
  --paper: #efefef;
  --panel: #ffffff;
  --line: #d9d2be;
  --border-strong: #dddddd;
  --red: #a5322a;
  --red-bg: #fbeae7;
  --green: #29614a;
  --green-bg: #e9f2ec;
  --amber: #93641a;
  --amber-bg: #faf1dc;
  --neutral: #5b6472;
  --neutral-bg: #f4f3ef;
  --info: #29506b;
  --info-bg: #e6eef3;
  --focus: #1b4fbf;
  --brand: #c03c0c;
  --font-brand: 'Source Sans Pro', 'Helvetica Neue', Helvetica, Arial, Frutiger, 'Frutiger Linotype', Univers, Calibri, 'Gill Sans', 'Gill Sans MT', 'Myriad Pro', Myriad, 'DejaVu Sans Condensed', 'Liberation Sans', 'Nimbus Sans L', Tahoma, Geneva, sans-serif;
  --font-display: 'Space Grotesk', sans-serif;
  --font-body: 'IBM Plex Sans', sans-serif;
  --font-mono: 'IBM Plex Mono', monospace;
  --radius: 4px;
}
```

Font import:
```css
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@500;600;700&family=IBM+Plex+Sans:wght@400;500;600&family=IBM+Plex+Mono:wght@400;500&display=swap');
```

Logo, sits above the h1, not beside it:
```html
<img src="https://intopia.digital/wp-content/themes/intopia-refresh/assets/images/intopia.png" alt="Intopia logo" class="intopia-logo">
```
```css
.intopia-logo { width: 180px; height: auto; }
.brand { display: flex; flex-direction: column; align-items: flex-start; gap: 16px; }
```

Favicon:
```html
<link rel="shortcut icon" href="https://intopia.digital/wp-content/uploads/2017/06/cropped-Intopia_FINAL_icon-32x32.png" sizes="32x32">
```

Page title, brand suffix matters, it's what shows in the tab and in bookmarks:
```html
<title>HTML email dark mode preview - Intopia</title>
```

h1: brand colour, brand font, no letter-spacing. Keep every other heading on the page black, not brand colour:
```css
.brand-text h1 {
  font-family: var(--font-brand);
  color: var(--brand);
  font-size: 2.4rem;
  font-weight: 600;
  margin: 0;
}
```

Buttons, solid brand colour, white text, no dashed/outline style anywhere:
```css
button {
  font-family: var(--font-body);
  font-weight: 600;
  font-size: 1.1rem;
  cursor: pointer;
  border-radius: var(--radius);
  padding: 12px 22px;
}
.btn-primary {
  background: var(--brand);
  color: #ffffff;
  border: 2px solid var(--brand);
}
```

Focus ring, applies globally, don't strip it:
```css
:focus { outline: 3px solid var(--focus); outline-offset: 2px; }
:focus:not(:focus-visible) { outline: none; }
:focus-visible { outline: 3px solid var(--focus); outline-offset: 2px; }
```

Body background `var(--paper)` (#efefef). Any panel/card content sits on `var(--panel)` (#ffffff) with a `2px solid var(--border-strong)` (#dddddd) border and `var(--radius)` corners.

## Layout, this tool's actual shape

Not a results screen, so don't borrow the tester's test-card/badge structure. This is a comparison viewer:

```
Header: logo (180px, above heading) → h1 "HTML email dark mode preview" →
  one-line explanation of what the tool does and its real limits (see below)
Body:
  - File input / textarea, same pattern as the tester and the EML converter
  - Detection summary: three plain lines, not badges, stating what was found
    - color-scheme meta: present / not present
    - supported-color-schemes meta: present / not present
    - @media (prefers-color-scheme: dark) block: found / not found
  - Two panes, light and dark, each an iframe showing the rendered email.
    Side-by-side is the likely default (better for screenshotting into a
    client report), but confirm against real 600-800px email widths once
    building, may need to stack or become a toggle instead
  - If no dark CSS was found: a visible callout directly on the dark pane
    (see below), not a footnote
```

## The "no dark CSS found" callout, styled deliberately louder than the tester's usual note

The tester's `.test-note` box (used for explanatory asides like the Tables caption/th guidance) uses a blue/info treatment:
```css
.test-note {
  margin: 0 0 16px;
  padding: 12px 14px;
  background: var(--info-bg);
  border-left: 3px solid var(--info);
  border-radius: 2px;
  font-size: 1rem;
  color: var(--ink);
}
```

For this specific message, don't reuse that blue treatment as-is. The functional handoff is explicit that this disclaimer "needs to be loud, not buried", it's flagging a real gap, not just adding context. Use the same box shape but swap to the tester's amber pairing instead (`--amber` / `--amber-bg`, the same colours behind its "Needs review" badge elsewhere), which already carries a "pay attention" association in this design system without borrowing the literal `.badge` component the functional handoff says not to use here:

```css
.dark-mode-callout {
  margin: 0 0 16px;
  padding: 12px 14px;
  background: var(--amber-bg);
  border-left: 3px solid var(--amber);
  border-radius: 2px;
  font-size: 1rem;
  color: var(--ink);
}
```

## What not to import from the tester

- No `.badge` / pass-fail vocabulary anywhere. This tool doesn't score anything.
- No test-card structure, no summary list, no "Test N" numbering. There's nothing to enumerate here, just one file and two renders of it.
- No focus-management choreography built for a multi-screen results flow (the tester's fade-out focus ring on a "Results" heading, for instance), that solved a problem specific to a paste-screen-to-results-screen transition this tool doesn't have.

## The trust statement

Same reasoning as the EML converter: if this becomes discoverable rather than purely internal, include a plain-language line near the top, matching the established tone:

> "Nothing you paste or upload here is sent anywhere. This preview runs entirely in your browser."
