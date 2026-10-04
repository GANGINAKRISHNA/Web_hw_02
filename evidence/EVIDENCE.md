# Homework 2 – Evidence (Sri Krishna Teja Gangina, Web Programming)

## Validation (run locally)
Page is XHTML 1.1 (XML declaration, XHTML 1.1 DOCTYPE, xmlns, xml:lang, http-equiv content type).
| File        | Result |
|-------------|--------|
| index.html  | Valid against the official XHTML 1.1 DTD (`xmllint --valid`, DTD files from the W3C CSS validator jar); a deliberately broken copy (bare `<a>` in body, `<font>`) was correctly rejected, confirming the check works |
| styles.css  | W3C css-validator jar, profile css3svg: "No Error Found" |
| print.css   | "No Error Found" |
Re-run on the live codd.cs.gsu.edu URL with the online W3C validators before submitting and paste the screenshots here.

## Layout architecture
- **Grid (page regions):** `#content` uses named areas `rating | overview / reviews | overview / counter | counter`, columns `1fr 250px`.
  Measured at 1400px: columns `550px 250px`, frame 800px content + 4px borders.
- **Grid fallback:** `@supports (display: grid)` wraps the grid rules; without grid, floats give the same 550/250 split
  (verified by forcing the fallback: reviews 550px wide, overview 250px wide, side by side; evidence/screenshot-grid-fallback.png).
- **Flexbox (repeated components):** `#reviews` (two columns), each `.quote` and `.critic` (icon + text, `flex-wrap: wrap`,
  no spacer elements), `#banner`, `#rating`, `#validators`.

## Responsive measurements (Chromium via Playwright; scrollWidth == viewport width means no horizontal scroll)
| Viewport | scrollWidth | Grid columns | Review column width | Reviews direction | Validators |
|----------|-------------|--------------|---------------------|-------------------|------------|
| 1400 | 1400 | 550px 250px | 259px each | row | fixed |
| 701  | 701  | 443px 250px | 208px each | row | fixed |
| 700  | 700  | 692px (stacked) | 664px each | column | static |
| 480  | 480  | 472px (stacked) | 453px each | column | static |
| 375  | 375  | 367px (stacked) | 352px each | column | static |

Breakpoint decisions: 700px is the point where a 250px sidebar leaves reviews only ~200px wide, so the grid collapses to one column
and review columns stack (overview moves under the reviews, counter bar last). At 480px the title shrinks to 18pt and the
rating number to 36pt so "32% (88 reviews total)" stays on one line; the 974px banner is clipped and centered by `overflow: hidden`.

## Keyboard / focus audit
Tab order (verified): Skip to reviews -> The Official TMNT Site -> Valid XHTML 1.1 -> Valid CSS.
Every link shows a 2px solid outline on focus (computed style confirmed). The skip link is off-screen until focused (evidence/focus-skip-link.png).
All images have alt text.

## Print
print.css (media="print") hides banner, badges and sidebar poster, removes backgrounds, single column, black on white,
prevents review boxes splitting across pages, and prints link URLs. See evidence/print-preview.pdf (2 pages, Letter).

## Files in evidence/
screenshot-1400px.png, screenshot-700px.png, screenshot-480px.png, screenshot-grid-fallback.png, focus-skip-link.png, print-preview.pdf
