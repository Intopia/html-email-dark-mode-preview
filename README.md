# HTML email dark mode preview

A single-page tool that shows what dark-mode support an HTML email declares, and what that email's own dark-mode CSS actually produces when it is switched on.

It is a viewer, not a test. Nothing in it passes or fails, and it makes no claim about how any particular email client will render the email.

https://intopia.github.io/html-email-dark-mode-preview/

---

## Purpose

The question this tool answers is not "does dark mode work". It is **did the team declare dark-mode support at all, and if so, what does it produce**.

For a lot of teams the honest answer is "we never thought about it". Showing that plainly is a legitimate wake-up call on its own. The tool does not need to judge success to be useful. It needs to make an invisible gap visible.

This matters in two settings:

- **Auditing.** Dark mode is a real accessibility concern and it is routinely skipped. A one-line finding backed by a screenshot is more persuasive than a paragraph of explanation.
- **Training.** Flipping a real email between its light and dark states in front of a room makes the point faster than any slide.

## What this tool is not

- Not a test. No badges, no pass/fail, no scoring vocabulary, no summary count.
- Not part of the HTML email accessibility tester. Separate tool, separate file, no shared code.
- Not a prediction of any email client's rendering. It shows the author's CSS, not Gmail's or Outlook's interpretation of it.

---

## Scope

**Input is HTML email only.** No URL or general webpage input, and that restriction is deliberate rather than a shortcut.

General web dark mode has a failure mode this tool cannot have. Plenty of sites implement dark mode with a JavaScript toggle and `localStorage` rather than the CSS media query. Those sites appear to do nothing under any `prefers-color-scheme` emulation, which is confusing and not fixable, because a preview tool has no way to trigger a JavaScript toggle it does not know exists.

Email cannot have this problem. **Email clients never execute JavaScript**, so every client's dark mode is necessarily built on static CSS. That is not just a smaller scope, it is a more reliable one.

### Why email makes the extraction faithful

Almost every email client strips or ignores `<link rel="stylesheet">`, so real-world email HTML is forced to carry all of its CSS either inline or in a `<style>` block in the head. That is a structural fact about how email has to be built, not a convenient assumption.

It means whatever HTML this tool receives already contains every rule that will ever apply. Nothing external to fetch, nothing that could silently fail to load, nothing hidden from view.

---

## How the dark view works

This is the part most likely to be misunderstood, including by a future version of this document.

The tool does **not** force a browser preference, and it does **not** copy rules out of the media query and paste them somewhere else. It switches the author's existing media query on, in place.

### The mechanism

1. The email HTML is rendered in an iframe via `srcdoc`. This is the "as authored" view, unmodified.
2. Because a `srcdoc` iframe is same-origin, the tool can read `iframe.contentDocument.styleSheets`.
3. It walks every rule recursively, looking for any `CSSMediaRule` whose `conditionText` mentions `prefers-color-scheme: dark`. The original `mediaText` string is stored against the rule.
4. To switch to dark, it rewrites `rule.media.mediaText`, removing only the `prefers-color-scheme: dark` condition and leaving everything else in the query intact.
5. To switch back, it restores the stored original string.

### Why this approach

**No CSS parsing.** The browser has already parsed the stylesheet correctly. Working on the normalised `conditionText` means no regex against raw CSS, no brace counting, and no failures on nested blocks.

**The cascade is preserved exactly.** The rules never move. This matters more than it sounds, because email CSS is overwhelmingly inline, and an inline `style` attribute beats any stylesheet rule regardless of source order. Real dark-mode email CSS therefore has to use `!important` to win. Leaving the rules where the author put them reproduces the cascade a real client would face. Extracting the rules and appending them as a new stylesheet would promote them by source order, which can make a rule win here that would lose in a real client. That is a small dishonesty that would never be visible.

**Compound conditions survive.** `@media (prefers-color-scheme: dark) and (max-width: 600px)` becomes `(max-width: 600px)`, so it still only applies at narrow widths. Flattening it to `all` would show a dark state that no client would ever produce.

### Alternatives considered

**Native `color-scheme` propagation.** Setting `color-scheme: dark` on the iframe element genuinely makes `prefers-color-scheme: dark` match inside the embedded document. It is real preference propagation, not emulation, and it is supported in Chromium and in Firefox 105 and later.

It was not used as the primary mechanism for two reasons. Safari does not implement it and resolves the query against the system setting instead, which would make the tool behave differently for different people on the team. And it also changes the default canvas and system colours, so an email with no dark CSS at all would still shift appearance, which undermines the whole point of the "nothing was declared" state.

**Regex extraction and re-injection.** Rejected for the cascade and parsing reasons above.

---

## What it detects

Five things, reported independently, because they are diagnostically interesting on their own rather than as a single yes/no.

| Reported | Read from |
|---|---|
| `<meta name="color-scheme">` | the raw source via `DOMParser` |
| `<meta name="supported-color-schemes">` | the raw source via `DOMParser` |
| `color-scheme` CSS property | CSSOM, any rule with `style.colorScheme` set |
| `@media (prefers-color-scheme: dark)` blocks | CSSOM, with each condition listed |
| Client-specific dark targeting | CSSOM, matched against `selectorText` |

Meta tags are read from the raw source rather than the rendered document, so nothing is missed if the browser relocates head content during parsing.

### The two meta tags change nothing here

Both are signals to email clients' own proprietary auto-dark systems, broadly meaning "respect my CSS, do not auto-invert me". Neither has any effect on how a browser renders the page. Only the `@media` block changes what this tool shows.

They are reported anyway. An email with the meta tags but no dark CSS, or the reverse, is exactly the kind of gap worth surfacing.

### Client-specific targeting, and why it is there

This check was added after the first build, and it fixes the tool's worst possible error.

Several clients implement dark mode by injecting an attribute into the email rather than honouring the media query. Outlook.com adds `data-ogsc` and `data-ogsb`, and various build tools use `.dark-mode` or `data-darkmode` hooks. An email can have substantial, deliberate dark-mode work done entirely through those selectors.

Without this check, such an email would be reported as having no dark-mode CSS at all. Telling a team they never thought about dark mode when they did is the one error that would burn the tool's credibility, so these are detected and reported honestly as declared but not previewable.

### The no dark CSS message

When no `@media (prefers-color-scheme: dark)` block is found, the dark radio and the compare checkbox both disable, and this takes their place:

> **No dark-mode CSS found.** There is nothing to apply, so this shows the email exactly as authored. Real email clients may still auto-darken it in ways this tool cannot predict or replicate, and none of them share a common, documented approach.

The control never sits in a dead state where clicking it does nothing. The absence of a difference must never be mistaken for a guarantee of anything, which is the same honesty principle the accessibility tester was built on.

---

## Interface decisions

**Single pane with a toggle, not permanent side by side.** Because the dark state is a `mediaText` flip, toggling is instant and preserves scroll position. Swapping the same pixels in place is perceptually stronger than comparing two panes, where the eye has to jump between positions and subtle colour changes get lost. It also gives the email full width, which removes the need for zoom and scroll syncing.

**Side by side survives as a compare mode.** Reports do read better with the juxtaposition in one image, so it is available as a checkbox rather than the default working view. It uses a second iframe, loaded only when dark CSS was actually found.

**Width control at 375, 600 and full.** Not decoration. Dark rules gated behind a compound condition such as `and (max-width: 600px)` will appear to do nothing at full width, and without this control that looks like a tool bug rather than an email behaviour.

**Two views, not one long page.** Intake first, then results. Matches the pattern of the HTML email accessibility tester and the EML converter.

---

## Accessibility decisions

The tool is built by an accessibility team, so these are not incidental.

**No change of context on input (WCAG 3.2.2).** Choosing a file loads the source into the textarea and stops. A `role="status"` message confirms what was loaded and directs the person to the "Show preview" button. Selecting a file never moves anyone to a new view on its own.

**Focus moves on submission.** Focus goes to the "Preview" heading, which carries `tabindex="-1"`. Returning via "Load a different email" sends focus back to the textarea.

**The focus ring on that heading needed an explicit rule.** Programmatic focus on a `tabindex="-1"` element does not match `:focus-visible`, so the global `:focus:not(:focus-visible) { outline: none }` was stripping it. Focus was moving silently, which is worse than not moving it. `.workbar h2:focus` at specificity 0,2,1 restores the ring over the 0,2,0 global rule.

**Native controls throughout.** The view and width switchers are real radio groups in fieldsets with legends, visually styled as segments. The compare control is a real checkbox. Nothing is a div pretending to be a control.

**Live region.** View changes happen inside an iframe where the visual change is invisible to a screen reader, so a `role="status"` region announces which view is showing.

**No colour-only meaning.** Findings are reported in words. There is no green or red anywhere, which is also consistent with the tool not scoring anything.

---

## Build details

Single self-contained HTML file. No build step, no framework, no npm. Open it locally or drop it on a server.

**External dependencies:** the Intopia logo, the favicon, and a Google Fonts import for Space Grotesk, IBM Plex Sans and IBM Plex Mono. All three token values carry fallbacks, so the tool degrades rather than breaks without a connection, but it will not look right offline. This matches the tester.

**Iframe sandbox:** `sandbox="allow-same-origin"` with no `allow-scripts`. Scripts cannot run in the preview, which matches every email client and enforces the property the whole scope rests on. `allow-same-origin` is what keeps CSSOM access working, so both attributes are load-bearing.

**Browser support:** anything current. The only unusual API is writing to `CSSMediaRule.media.mediaText`, which is wrapped in try/catch. If a rule ever silently fails to flip, that is the first place to look.

### A CSS trap worth remembering

`[hidden] { display: none !important; }` is in the stylesheet deliberately. The `hidden` attribute only gets `display: none` from the browser's own stylesheet, so any author rule such as `.workbench { display: grid }` or `.notice { display: flex }` beats it and the element stays visible.

This bit the first build in three places at once, and it fails quietly because the page still looks plausible. If something will not hide, check this first.

---

## Test files

Three versions of the same fictional library email, text and background colours only, no images.

**`test-1-no-dark-mode.html`** — no meta tags, no `color-scheme` property, no media query. Everything should report not found, both controls should disable, and the callout should appear.

**`test-2-full-dark-mode.html`** — both meta tags, `:root { color-scheme: light dark }`, and one media block that recolours all ten surfaces with `!important` throughout. All five findings report found and the toggle flips cleanly.

**`test-3-partial-dark-mode.html`** — declares intent, then half delivers, in four specific ways:

- the `.h1` rule deliberately omits `!important`, so the inline colour wins and the heading stays dark on a dark card
- the button background flips but the label colour is never touched
- the footer is never targeted and keeps its light band
- a promo strip sits behind `@media (prefers-color-scheme: dark) and (max-width: 600px)`, so it only darkens at 375 or 600

It also carries `[data-ogsc]` and `[data-ogsb]` rules, so the client-hook finding fires.

If `.h1` in test 3 ever does flip to light green, the `mediaText` rewrite is changing cascade position and something is wrong.

---

## Known limitations

- **Remote images load.** Tracking pixels in a client's real email will fire from whoever is running the preview. Worth knowing before pasting a client's newsletter.
- **Client-specific dark modes cannot be shown**, only reported, because they depend on an attribute the browser never adds.
- **External stylesheets cannot be read.** `cssRules` throws on cross-origin sheets. This is caught and surfaced as its own finding, which is useful in itself since real clients strip those anyway.
- **No prediction of auto-darkening.** Clients that invert emails do so with undocumented, inconsistent heuristics. The tool says so rather than guessing.

## Worth testing before wider use

Real `.eml` files converted through the EML converter, rather than only hand-written HTML. Quoted-printable soft line breaks can split a declaration across lines, and if `prefers-color-scheme` arrives with a `=` and a newline through the middle of it, the browser will not parse the block and detection reports nothing found.

That false negative looks identical to an email that genuinely has no dark mode. Running two or three emails through both paths, converted and hand-extracted, gives a control. If they differ, the fix belongs in the converter rather than here.
