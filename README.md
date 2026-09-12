# HTML email dark mode preview

A single-page tool that shows what dark-mode support an HTML email declares, and what its own dark-mode CSS actually produces when switched on.

It is a viewer, not a test. Nothing passes or fails, and it makes no claim about how any particular email client will render the email.

---

## Purpose

The question is not "does dark mode work". It is **did the team declare dark-mode support at all, and if so, what does it produce**.

For a lot of teams the honest answer is "we never thought about it". Showing that plainly is a legitimate finding on its own. The tool does not need to judge success to be useful; it needs to make an invisible gap visible.

## Scope

**HTML email only.** Not a general web page previewer.

General web dark mode is often implemented with a JavaScript toggle and `localStorage` rather than the CSS media query, which no preview tool can trigger. Email cannot have that problem: email clients never execute JavaScript, so every client's dark mode is built on static CSS.

Email also carries all of its CSS inline or in a `<style>` block, because clients strip `<link rel="stylesheet">`. Whatever HTML the tool receives already contains every rule that will ever apply.

---

## How the dark view works

It does **not** force a browser preference, and it does **not** copy rules out of the media query. It switches the author's existing query on, in place.

1. The email renders in a `srcdoc` iframe. That is the "as authored" view.
2. Because the iframe is same-origin, the tool reads `iframe.contentDocument.styleSheets`.
3. It walks every rule recursively, finds each `CSSMediaRule` mentioning `prefers-color-scheme: dark`, and stores its original `mediaText`.
4. To go dark, it rewrites `mediaText` to remove only the `prefers-color-scheme` condition, per query. To go back, it restores the stored string.
5. At the same time, `prefers-color-scheme: light` blocks are **suppressed**, because in a client set to dark they would not match at all.

### Why this approach

**No CSS parsing.** The browser already parsed it. Working on the normalised `conditionText` means no regex over raw CSS and no brace counting.

**The cascade is preserved.** Rules never move. Email CSS is overwhelmingly inline, and an inline `style` attribute beats any stylesheet rule regardless of source order, so real dark-mode CSS has to use `!important`. Leaving rules in place reproduces the cascade a real client faces. Extracting and appending them would promote them by source order and could make a rule win here that would lose in a client.

**Compound conditions survive.** `@media (prefers-color-scheme: dark) and (max-width: 600px)` becomes `(max-width: 600px)`, so it still only applies at narrow widths.

**Light blocks are handled too.** Without suppressing them, light and dark rules apply simultaneously and source order picks the winner. A real email was found where the image swap worked only because its light block happened to come first.

### Alternative considered

Setting `color-scheme: dark` on the iframe genuinely makes `prefers-color-scheme: dark` match inside it, in Chromium and Firefox 105+. Rejected because Safari does not implement it, and because it also shifts the default canvas, so an email with no dark CSS would still change appearance, undermining the point of the "nothing declared" state.

---

## What it reports

Eight findings, independently:

| Reported | Source |
|---|---|
| `<meta name="color-scheme">` | raw source via `DOMParser` |
| `<meta name="supported-color-schemes">` | raw source via `DOMParser` |
| `color-scheme` CSS property | CSSOM |
| `@media (prefers-color-scheme: dark)` blocks | CSSOM, with conditions and nesting context |
| `@media (prefers-color-scheme: light)` blocks | CSSOM |
| Client-specific dark targeting | CSSOM selectors |
| `<picture>` dark-mode sources | `source[media]` in the iframe document |
| Dark CSS inside conditional comments | raw source scan |

Condition lists are deduplicated with counts, since a real template can repeat one condition across fifty blocks.

### The meta tags change nothing here

Both are signals to email clients' own auto-dark systems. Neither affects browser rendering. They are reported anyway, because an email with the meta tags but no dark CSS, or the reverse, is exactly the gap worth surfacing.

### Declared but not previewable

Three mechanisms are reported and cannot be rendered, all for the same reason: a browser cannot act on them.

- **Client hooks.** `[data-ogsc]`, `[data-ogsb]` and similar depend on the client injecting an attribute the browser never adds.
- **`<picture>` with `source media`.** An HTML attribute, not a stylesheet rule, so the media rewrite cannot reach it.
- **Dark CSS in downlevel-hidden conditional comments.** A browser treats these as ordinary comments, so the CSS never exists. Block counts exclude it.

Reporting these matters. Without the first, an email with real Outlook dark-mode work would be reported as having none, which is the worst error the tool could make.

### When nothing is found

The dark radio and compare checkbox both disable, and a callout takes their place saying there is nothing to apply and that real clients may still auto-darken the email unpredictably. The control never sits in a dead state. If non-previewable work was declared, the callout says so.

---

## Interface

**Single pane with a toggle.** The flip is instant and preserves scroll position. Swapping the same pixels in place reads more clearly than comparing two panes.

**Side by side** is available as a compare checkbox for report screenshots.

**Width control** at 375, 600 and full, because dark rules gated behind a breakpoint would otherwise appear to do nothing. The live pane width is shown in the toolbar, since "full" means the width of the preview pane and not the window.

---

## Accessibility

- **No change of context on input.** Choosing a file loads it into the textarea and stops. A `role="status"` message confirms what loaded and points at the Show preview button.
- **Focus moves on submission** to the "Preview" heading, which carries `tabindex="-1"`. Programmatic focus does not match `:focus-visible`, so an explicit rule restores the ring; without it focus moved silently.
- **Native controls throughout.** Real radio groups in fieldsets, a real checkbox. Nothing is a div pretending to be a control.
- **Live region** announces view changes, since the visual change happens inside an iframe.
- **No colour-only meaning.** Findings are reported in words, with no green or red anywhere.

---

## Build notes

Single self-contained HTML file. No build step, no framework.

**External dependencies:** the Intopia logo, favicon, and a Google Fonts import. All font tokens carry fallbacks, so it degrades rather than breaks offline.

**Iframe sandbox:** `sandbox="allow-same-origin"` with no `allow-scripts`. Scripts cannot run in the preview, matching every email client. `allow-same-origin` is what keeps CSSOM access working, so both are load-bearing.

**A CSS trap worth remembering.** `[hidden] { display: none !important; }` is in the stylesheet deliberately. The `hidden` attribute only gets `display: none` from the browser's own stylesheet, so any author rule such as `display: grid` beats it and the element stays visible. This bit the first build in three places and failed quietly.

---

## Testing

See `TEST-CORPUS.md`. Fifteen synthetic files, each isolating one mechanism, plus notes on the real emails worth keeping as fixtures.

Two bugs were found by running that corpus and real emails against it: a client-hook false positive, and light blocks never being suppressed.

## Known limitations

- **Remote images load.** Tracking pixels in a client's real email will fire from whoever runs the preview.
- **No prediction of auto-darkening.** Clients that invert emails use undocumented, inconsistent heuristics. The tool says so rather than guessing.
- **External stylesheets cannot be read.** Caught and surfaced as its own finding, which is useful since real clients strip them anyway.
- **`.eml` conversion.** Quoted-printable soft line breaks can split a declaration across lines. If the converter fails to rejoin them the block is dropped and the tool honestly reports nothing found, which looks identical to an email that has none. Edge case 15 tests this specifically.
