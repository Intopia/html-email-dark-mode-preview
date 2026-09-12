# Test corpus, HTML email dark mode preview

Fifteen files for testing the dark mode preview tool. Each isolates one mechanism, so when something fails you know immediately what broke.

Every file carries its own expected findings in an HTML comment at the top, including exact block and rule counts. The comment is the spec, so it cannot drift out of sync with the file.

---

## How to run one

1. Load the file into the preview tool.
2. Check the five findings against the file's header comment, including the counts.
3. Check the state of the "With dark-mode CSS applied" radio and the "Compare side by side" checkbox.
4. Check whether the amber callout is showing.
5. Toggle to dark and cycle the width control where the file says width matters.

Steps 3 and 4 matter most on the four files where nothing is declared. A dead but clickable toggle would otherwise slip through.

---

## The files

### 01, control
`edge-01-control.html`

Ordinary table-based email, inline CSS, no dark mode handling of any kind.

**Tests:** false positives. It is the only file where a wrong answer would be invisible on all the others.
**Expect:** five Not found, both controls disabled, callout shown.

### 02, authored dark mode, complete
`edge-02-authored-complete.html`

Both meta tags, `color-scheme` property, one media block covering every surface with `!important` throughout.

**Tests:** the happy path, and the accuracy of the rule count. Every other count in the set is measured against this working.
**Expect:** all five found, 1 block / **10 rules**, controls enabled, no callout. Every surface flips.

### 03, client hooks only
`edge-03-client-hooks-only.html`

Real dark-mode work done entirely through `[data-ogsc]` and `[data-ogsb]`. No media query anywhere.

**Tests:** the false-negative guard. Before this check existed, an email like this was reported as having no dark mode at all.
**Expect:** no media block, controls disabled, callout shown, **and** client hooks found: 2 kinds, 5 rules and 3 rules. Both statements must appear together.

### 04, image switching
`edge-04-image-switching.html`

Two swap methods in one file. A CSS class swap inside the media block, and a `<picture>` with `<source media="(prefers-color-scheme: dark)">`. Images are inline SVG data URIs, so the file is self-contained and fires no network requests.

**Tests:** what the tool can and cannot follow.
**Expect:** 1 block / **7 rules**. The wordmark swaps. The banner does **not**, because `<picture>` is an HTML attribute rather than a stylesheet rule. That is correct but it is a silent gap.

### 05, light-only
`edge-05-light-only.html`

A deliberate opt-out. `content="light only"`, `supported-color-schemes="light"`, `:root { color-scheme: light }`, every background hard-coded.

**Tests:** that meta content strings report verbatim. The `only` keyword is the strong form and must not be normalised away.
**Expect:** both meta tags found with their exact strings, no media block, controls disabled, callout shown.

Run alongside 01. The preview is identical in both, and the findings panel is the only thing separating "nobody thought about it" from "somebody decided against it".

### 06, MSO conditional comments
`edge-06-mso-conditional.html`

Two dark blocks. One in a downlevel-revealed wrapper that browsers parse, one in a downlevel-hidden wrapper that they do not.

**Tests:** silent under-reporting. The source contains 2 blocks and 7 rules; the tool reports 1 and 5, with nothing in the interface indicating a miss.
**Expect:** 1 block / **5 rules**. A labelled amber strip in the middle stays light after toggling, because only the invisible block styles it.

### 07, compound conditions
`edge-07-compound-conditions.html`

Five dark blocks gated five different ways, including two that share an identical bare condition, and one carrying a media type alongside two features.

**Tests:** condition rewriting. Only the `prefers-color-scheme` part may be removed.
**Expect:** **5 blocks / 11 rules**, all conditions listed verbatim with their width parts intact.

| | Strip 1 | Strip 2 | Strip 3 | Strip 4 |
|---|---|---|---|---|
| Full | dark | light | dark | light |
| 600 | dark | dark | light | light |
| 375 | dark | dark | light | dark |

Every strip dark at every width means conditions are being flattened. The footer is untargeted by design and never changes.

### 08, broken CSS
`edge-08-broken-css.html`

Three parse faults, all found in real emails: an unclosed comment that swallows a rule, a stray closing brace that leaks a rule out of its media query, and a missing colon that drops one declaration from a rule.

**Tests:** that the tool reports what exists rather than what was typed. Reading the source suggests 3 blocks and 7 rules.
**Expect:** 3 blocks / **5 rules**. Strip 2 is washed out at 2.1:1 in the **as authored** view, before you toggle. Strip 3 drops to 1.17:1 once toggled. Most of the email correctly stays light.

Note: a stray closing brace consumes forward to the next opening brace, so it swallows whatever follows it. It is deliberately the last group in that stylesheet.

### 09, no style block
`edge-09-no-style-block.html`

No `<style>`, no `<link>`, no meta declarations. Everything inline or in presentational attributes.

**Tests:** the empty path. `doc.styleSheets` is empty, which is a different code path from 01 reaching the same answer.
**Expect:** five Not found, controls disabled, callout shown, and **no** "Stylesheets that could not be read" line.

### 10, massive generated CSS
`edge-10-massive-css.html`

248KB, 5,255 rules, 17 media blocks of which one is dark. Decoys include 13 breakpoints plus `print`, `prefers-reduced-motion` and `prefers-contrast`.

**Tests:** performance of the recursive walk, and filtering. The two `prefers-` decoys are the trap.
**Expect:** 1 block / **6 rules**. Any count above 1 means a decoy is getting through. Should feel instant, including with compare mode on.

### 11, selectors that match nothing
`edge-11-selectors-match-nothing.html`

Reconstructed from a real newsletter. Five class selectors, of which one matches anything. The missing one carries the only background rule. Six element selectors all match, so text colour does change.

**Tests:** the gap between declared and applied.
**Expect:** 1 block / **11 rules**, client hooks Not found. The email stays white, body text drops from 10.9:1 to **2.6:1**, and one strip flips correctly as the single working island.

Most likely file to be misread as the tool doing nothing. Use compare side by side.

*This file caught a real bug: `.darkmode` inside a dark block was being double-reported as a client hook. Fixed by only matching hooks outside dark blocks.*

### 12, two blocks that cancel out
`edge-12-blocks-cancel-out.html`

Reconstructed from a real retail email. Two dark blocks using different naming conventions, both landing on the same two elements with opposite intent. Identical specificity, all `!important`, so source order decides.

**Tests:** cascade fidelity.
**Expect:** **2 blocks / 8 rules**, listed separately. Page and card flip, the logo does **not** swap, body text goes unreadable on the dark card.

A selector-match check would not catch this. Every selector matches, every rule is live, and they destroy each other. Only looking at the result finds it.

### 13, nested media queries
`edge-13-nested-media.html`

Four nesting arrangements: dark inside a breakpoint, dark inside `print`, a breakpoint inside dark, and dark inside dark.

**Tests:** the recursive walk. A top-level-only walk finds 2 blocks instead of 5.
**Expect:** **5 blocks / 9 rules**. N4 must flip, otherwise the walk is not recursing. N2 never changes at any width, being inside `print`.

Two caveats. The rule count counts direct children, so a nested at-rule counts as one. And `.h1` and `.body-text` are untargeted, so they stay dark on a dark card.

### 14, commas and `@supports`
`edge-14-comma-and-supports.html`

Four blocks: a comma list whose other branch does not match on screen, a comma list whose other branch does, a dark block nested inside `@supports`, and a query combining a comma with compound conditions.

**Tests:** per-query rewriting, and recursion through an at-rule that is not `@media`.
**Expect:** 4 blocks / **8 rules**, conditions listed with commas intact.

S2 is the one to look at. `screen, (prefers-color-scheme: dark)` is an OR, `screen` matches, so it is **already dark before you toggle**. Reported as a dark block, not conditional at all.

### 15, converted `.eml`
`edge-15-converted.eml` and `edge-15-expected.html`

A full MIME message with three quoted-printable soft line breaks placed deliberately inside `@media`, inside `prefers-color-scheme`, and inside a class attribute. Plus an encoded-word Subject and a pound sign.

**Tests:** whether dark-mode CSS survives the EML converter. This is the only failure that reaches you disguised as a legitimate result: a decoder that fails to rejoin lines produces a broken at-rule, the block is dropped, and the tool honestly reports "No dark-mode CSS found".

**How to run:** convert the `.eml`, diff the output against `edge-15-expected.html` ignoring line endings, then load both into the preview and compare findings. Both should report 1 block / **6 rules**.

A failure here is a converter problem, not a preview tool problem. The distinction matters because the symptom shows up in the preview.

---

## Extra file

`diagnostic-nested-vs-width.html` is not part of the numbered set. It separates a nested-media bug from a pane-width explanation by expressing the same condition in nested and flat form alongside a pure-CSS width readout.

---

## What the run changed in the tool

Every issue below was found by running this corpus, or by the one real email that followed it. All are fixed.

- **Client-hook false positive** (found by 11). `.darkmode` sitting inside a dark media block was reported both as a rule in that block and as a client hook. Hooks are now only matched outside `prefers-color-scheme` blocks.
- **Light blocks were never switched off** (found by a real email). `@media (prefers-color-scheme: light)` still matched while the dark view was showing, so light and dark rules applied together and source order picked the winner. Light blocks are now suppressed in the dark view and restored on the way back.
- **Two silent gaps closed.** Dark CSS in downlevel-hidden conditional comments (06) and `<picture>` with `source media` (04) are now reported as declared but not previewable, the same way client hooks already were.
- **Nesting context** now appears in the conditions list, so a block inside `@media print` is visibly distinguishable.
- **Condition lists are deduplicated** with counts. A real template showed 52 identical lines; it now shows one line with a count.
- **Pane width** is shown live in the toolbar. "Full" means the width of the preview pane, not the window, and on a laptop it can land under 601px, which silently stops desktop breakpoints firing. This cost three test runs on 13 before it was diagnosed.

## Remaining limitations, by design

- `<picture>` swaps and MSO-hidden blocks are reported but cannot be rendered. A browser cannot act on either.
- Client hooks cannot be rendered, since the attribute is injected by the client.

## Real specimens worth keeping alongside these

Synthetic files only test what you thought of. Real emails found two of the issues above. Worth keeping as fixtures:

- An email doing dark mode properly across four mechanisms at once: 52 dark blocks, 4 light blocks, 114 Outlook hook rules, and 18 matched `<picture>` pairs.
- One where two dark blocks cancel out on source order, which 12 reconstructs.
- One where the background rule targets a class never applied to the markup, which 11 reconstructs.
- Emails with no dark CSS that still render dark, because the client auto-inverted them. These are the proof that absence of dark CSS does not mean the email renders light.

---

## What the corpus is for

Synthetic files only test what you thought of. Every genuine surprise so far came from real emails, and none of the three would have been on a list written in advance.

Use these to know immediately what broke when something changes, and keep testing against emails from the wild for coverage.
