# HTML email accessibility tester edge cases

1. **Normal HTML email**
   - Basic tables for layout
   - Inline CSS
   - Standard text, links and images
   - No dark mode handling
   - Good baseline/control specimen

2. **Authored dark mode**
   - `@media (prefers-color-scheme: dark)`
   - `color-scheme: light dark`
   - `supported-color-schemes`
   - Different foreground/background colours in dark mode
   - Dark-mode-specific button/link colours
   - Check contrast in both modes

3. **Outlook dark-mode hacks**
   - `[data-ogsc]` and `[data-ogsb]`
   - Outlook-specific overrides
   - Conditional comments
   - Different behaviour between Outlook desktop and Outlook.com
   - Colours deliberately chosen to resist forced inversion

4. **Image switching**
   - Light logo vs dark logo
   - `.light-img` / `.dark-img`
   - `display:none` switching
   - `<picture>` and `<source media="(prefers-color-scheme: dark)">`
   - Different alt behaviour when one version is hidden
   - Ensure duplicate image alternatives do not create duplicate accessible content

5. **Light-only email**
   - `color-scheme: light`
   - Explicit attempt to disable dark mode
   - Hard-coded white backgrounds
   - Check what happens when clients force dark mode anyway

6. **Huge nested table layout**
   - Many levels of nested `<table>`, `<tr>` and `<td>`
   - Spacer cells
   - Presentation tables with and without `role="presentation"`
   - Empty layout cells
   - Tables containing other tables
   - Useful for distinguishing layout tables from genuine data tables

7. **VML / MSO conditionals**
   - `<!--[if mso]>`
   - VML buttons
   - `v:rect`, `v:roundrect`, `v:fill`
   - Outlook-only background images
   - Alternative HTML for non-Outlook clients
   - Important for ensuring the tester does not report invisible fallback code twice

8. **Heavily image-based marketing email**
   - Hero image containing text
   - Multiple promotional banners
   - Product cards rendered mostly as images
   - Missing, poor or extremely long alt text
   - Images used as links
   - Email that becomes nearly meaningless with images disabled

9. **Image-only email**
   - Essentially one large image sliced into several pieces
   - Minimal actual text
   - Strong test for meaningful text alternatives and reading experience

10. **Background-image content**
   - CSS `background-image`
   - VML equivalent for Outlook
   - Text visually overlaid on background imagery
   - Background image contains meaningful information that has no HTML equivalent

11. **Responsive email**
   - Desktop and mobile layouts differ significantly
   - Media queries
   - Stacked columns
   - Hidden desktop/mobile variants
   - Elements reordered visually
   - Check whether source/read order remains sensible

12. **Duplicate responsive content**
   - Separate desktop and mobile copies of the same heading, CTA or image
   - One version visually hidden
   - Common source of duplicated accessible content if hiding techniques fail

13. **CSS-hidden content**
   - `display:none`
   - `visibility:hidden`
   - `opacity:0`
   - Zero-height containers
   - Off-screen positioning
   - Useful for testing whether intentionally hidden email-client content generates false positives

14. **Preheader text**
   - Genuine visible preheader
   - Hidden preheader
   - Long padding strings such as `&nbsp;`, zero-width characters or repeated symbols
   - Ensure junk used to control inbox previews is not mistaken for meaningful content

15. **Decorative images**
   - Tracking pixels
   - Spacer GIFs
   - Dividers
   - Decorative icons
   - Correct `alt=""`
   - Missing alt on genuine decorative content versus meaningful content

16. **Tracking-heavy email**
   - 1×1 tracking pixels
   - Tracking query strings
   - Redirected URLs
   - Multiple analytics links
   - Should not swamp reports with irrelevant image/link findings

17. **Complex linked images**
   - Image wrapped in `<a>`
   - Empty alt
   - Alt identical to nearby text
   - Several adjacent image links with identical accessible names

18. **CTA buttons implemented different ways**
   - Styled `<a>`
   - Table-based button
   - VML Outlook button
   - Image button
   - Plain text link styled as button
   - Useful for checking link purpose and accessible naming consistently

19. **Badly constructed links**
   - `click here`
   - `learn more`
   - Raw URLs
   - Empty links
   - Linked tracking images
   - Same link text pointing to different destinations
   - Different link text pointing to the same destination

20. **Long newsletter**
   - Large editorial newsletter with dozens of headings, links and sections
   - Tests performance as well as heading hierarchy and link reporting
   - Especially useful for seeing whether the report becomes unusably noisy

21. **Heading edge cases**
   - Proper heading hierarchy
   - `<h1>` followed by `<h3>`
   - Fake headings styled with `<div>` or `<strong>`
   - Multiple `<h1>` elements
   - No headings at all
   - Heading text rendered as images

22. **Semantic HTML mixed with email HTML**
   - `<header>`, `<main>`, `<section>`, `<article>`, `<footer>`
   - Traditional table markup surrounding modern semantic elements
   - Useful because support can vary significantly between email clients

23. **Lists**
   - Genuine `<ul>` / `<ol>`
   - Fake lists built with `<br>`
   - Table-based bullet lists
   - Unicode bullets
   - Lists deliberately reset with CSS because of email-client rendering issues

24. **Data table inside a layout table**
   - Layout email containing a genuine pricing, comparison or event table
   - Tests whether the tool can avoid treating every table identically
   - `th`, `scope`, captions and headers become relevant

25. **Forms in email**
   - Newsletter survey
   - AMP-style/interactivity-inspired markup
   - Text inputs, radio buttons or checkboxes
   - Missing labels
   - Unsupported fallbacks
   - Rare, but an excellent stress case

26. **Interactive email**
   - Accordion
   - Hamburger menu
   - CSS checkbox hacks
   - Carousel
   - Hover states
   - Hidden/revealed content
   - Tests how the accessibility checker handles duplicated and hidden DOM content

27. **Animated GIF**
   - Important message appears only in later frames
   - First frame meaningless
   - Flashing/motion
   - Alternative text inadequate
   - Relevant to both accessibility and reduced-motion concerns

28. **Motion-aware email**
   - `prefers-reduced-motion`
   - Animation removed or altered for users requesting less motion
   - Useful equivalent to the authored-dark-mode test

29. **Language edge cases**
   - Missing `<html lang>`
   - Correct document language
   - Sections containing another language
   - `lang` attributes on individual phrases

30. **Character/entity abuse**
   - `&nbsp;` used extensively for layout
   - Zero-width characters
   - Emoji
   - Unicode arrows
   - Decorative punctuation
   - Bullet characters used as structure

31. **Accessibility-hidden content**
   - `aria-hidden="true"`
   - `role="presentation"`
   - `role="none"`
   - Visually hidden text
   - Elements where ARIA conflicts with native semantics

32. **ARIA-heavy email**
   - Roles copied from web application code
   - `aria-label`
   - `aria-labelledby`
   - Redundant or invalid ARIA
   - Useful for making sure the checker does not assume more ARIA means better accessibility

33. **Broken HTML**
   - Missing closing tags
   - Invalid nesting
   - Tables repaired automatically by the browser parser
   - Duplicate IDs
   - Unquoted attributes
   - Real production email can be remarkably ugly

34. **CSS stripped/minimised email**
   - Almost everything inline
   - No `<style>` block
   - Represents output from some email platforms after processing

35. **Massive generated CSS**
   - Thousands of utility selectors
   - Client-specific resets
   - Repeated media queries
   - Tests parser/report performance rather than just accessibility rules

36. **Email-builder output**
   - Mailchimp
   - Campaign Monitor
   - HubSpot
   - Salesforce Marketing Cloud
   - Braze
   - Marketo
   - Different builders produce very different markup patterns

37. **Transactional email**
   - Password reset
   - Order confirmation
   - Invoice
   - Booking confirmation
   - Typically simpler than marketing email, but meaningful link names, reading order and critical information matter more

38. **Plain/simple newsletter**
   - Primarily text
   - Few images
   - Proper headings and links
   - Valuable as a good specimen so the test corpus is not composed entirely of pathological examples

39. **Intentionally excellent accessible email**
   - Semantic structure
   - Sensible tables
   - Useful alt text
   - Good contrast
   - Proper link text
   - Responsive design
   - Dark mode
   - Important for proving the tester can produce a quiet report when there is not much wrong

40. **Kitchen-sink torture test**
   - Nested tables
   - Responsive duplication
   - Conditional MSO
   - VML
   - Authored dark mode
   - Image switching
   - Animated GIF
   - Hidden preheader
   - Tracking pixels
   - Background images
   - Terrible alt text
   - Malformed markup
   - Useful for checking whether changes to the tester accidentally break several other behaviours