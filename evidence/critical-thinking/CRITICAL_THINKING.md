# Critical Thinking Lab – Graduate answers (Sri Krishna Teja Gangina, Web Programming)

All numbers below were measured on the submitted page (`index.html` + `styles.css`) in Chromium (Playwright),
using `getBoundingClientRect()` / computed styles, and for Q4 the Chrome DevTools Protocol "Matched Styles" data
(the same data the Styles/Computed panels show). Screenshots carry a banner with the viewport width and file path.

> BEFORE SUBMITTING: re-take the screenshots in the single browser you will use for your live defense (so the
> address bar is visible), open DevTools, and confirm the Computed values below match what you see. Be ready to
> explain ONE answer aloud for two minutes.

---------------------------------------------------------------------------------------------------------

## 03 – Architecture defense (Grid for regions, Flexbox for components)

### Evidence
- `Q3-annotated-700px.png` – viewport 700px, grid areas outlined/labelled, flex `.critic` rows and `.column`s outlined
- `Q3-annotated-480px.png` – viewport 480px, same annotations

### Code (styles.css)
```css
@supports (display: grid) {
  #content {
    display: grid;
    grid-template-columns: 1fr 250px;
    grid-template-rows: auto 1fr auto;
    grid-template-areas:
      "rating   overview"
      "reviews  overview"
      "counter  counter";
  }
}
#reviews { grid-area: reviews; display: flex; align-items: flex-start; padding: 10px 0; }
.column  { width: 47%; margin-left: 2%; }
.critic  { display: flex; align-items: center; flex-wrap: wrap; margin: 8px 0 0 5px; font-style: italic; }
@media (max-width: 700px) {
  #content { grid-template-columns: 1fr; grid-template-rows: auto;
             grid-template-areas: "rating" "reviews" "overview" "counter"; }
  #reviews { flex-direction: column; align-items: stretch; }
  .column  { width: 96%; }
}
```

### Measured results
| Measurement | 700px | 480px |
|---|---|---|
| Horizontal overflow (scrollWidth − clientWidth) | **0** | **0** |
| `#content` border-box width | 700 | 480 |
| Computed `grid-template-columns` | `692px` (single column) | `472px` (single column) |
| Computed `grid-template-areas` | rating / reviews / overview / counter (stacked) | same |
| Area placement (left, top, width) | rating 4,131,692 · reviews 4,214,692 · overview 4,997,692 · counter 4,1729,692 | rating 4,106,472 · reviews 4,189,472 · overview 4,1004,472 · counter 4,1760,472 |
| `.column` width | 664.3px | 453.1px |
| `#reviews` computed flex-direction / align-items | column / stretch | column / stretch |
| First `.quote` box height (text re-wraps) | 56px | 68px |
| `.critic` computed display / align-items / flex-wrap | flex / center / wrap | flex / center / wrap |
| `.critic` icon-centre minus text-centre (flex) | **0px** | **0px** |

Desktop reference (1400px): `#rating`/`#reviews` width 550, `#overview` 250, `#counter` 800 (= full inner frame).
`#overview` height 792 = `#rating` 83 + `#reviews` 709, i.e. it spans two grid rows.

### Defence – which relationship belongs to which system
**Grid owns the relationships between the page's regions (two-dimensional).**
- The overview must sit beside *two* stacked regions and be exactly as tall as both (792 = 83 + 709), and the counter bar must span *both* columns (800px). Those are row-span and column-span relationships; Grid expresses them in three lines of `grid-template-areas`.
- At 700px the whole page is re-arranged by redeclaring the areas only (overview moves from the right column to under the reviews). No HTML changed and no `order` hacks were needed. Measured: every area lands at left=4 with width equal to the single `1fr` track (692px / 472px), and overflow is 0.
- Flexbox could not do this without extra wrapper elements, because a flex container only distributes items along one axis.

**Flexbox owns the content-driven, one-dimensional alignment inside repeated components.**
- `.critic` (icon + name/publication): with `display:flex; align-items:center` the icon centre and text centre differ by 0px. When I forced `display:block` on the same element the difference became **−11.42px** (icon sits on the text baseline), at both 700px and 480px, so flex is what gives the aligned result with no spacer elements. `flex-wrap: wrap` lets the text drop below the icon if a long publication name ever runs out of room.
- `#reviews` holds the two 47%/2% columns: at desktop both columns start at the same y (224/224). When I set it to `display:block` the second column dropped to y=695 (471px lower), so flex is what puts the columns side by side; at ≤700px it switches to `flex-direction: column` so they stack at 664px / 453px wide.

**Measured trade-off:** at 480px the first review grows from 56px to 68px high because the same text wraps in a narrower box, but nothing overflows horizontally (0px) because every region is a single 472px track and the columns are 96% wide. The heading, rating and overview stay in the same order, so hierarchy is preserved.

---------------------------------------------------------------------------------------------------------

## 04 – Failure analysis (controlled defect in a separate copy)

The defect lives only in `evidence/critical-thinking/defect-copy/` (source: `/home/claude/hw2_defect_copy/`).
The original files in the submission were never edited, and its screenshots/evidence are untouched.

### Defect introduced (one rule, `defect-copy/styles.css`, line 161)
```css
.quote {
  height: 40px;        /* <-- defect */
  overflow: hidden;    /* <-- defect */
  display: flex;
  ...
}
```
Diff against the original is exactly those two lines.

### Defect state – `Q4-defect-1400px.png` (viewport 1400px, red outline = clipped)
Symptom: reviews 1, 4, 5, 7 and 8 are visibly cut off mid-sentence ("...but the been-there" / "...have been"); "It stinks!" has a large empty box.

### Diagnosis from browser output (DevTools Matched Styles / Computed + scroll metrics)
| Check | Result |
|---|---|
| Matched rule for `.quote` | `.quote { height: 40px; overflow: hidden }` – the only author rule setting `height`/`overflow` |
| Computed | `height: 40px; overflow-y: hidden; box-sizing: content-box; padding-top: 8px; border-top-width: 2px` |
| Review 1 clientHeight vs scrollHeight | **56 vs 100** → 44px of text hidden |
| Review 4 / 5 / 7 / 8 | 56 vs 64 / 88 / 64 / 64 → clipped by 8 / 32 / 8 / 8px |
| Reviews 2, 6 | 56 vs 56 → fit exactly (not clipped) |
| Review 3 ("It stinks!") | box is 56px instead of its natural 36px → 20px of dead space |
| Same defect at 480px | only review 1 clipped (56 vs 64; screenshot `Q4-defect-480px-mild.png`) because boxes are wide there and text wraps less |

### Root cause
A fixed `height` was put on an element whose content height depends on its width (text wraps differently at every
viewport). With `box-sizing: content-box`, 40px content + 16px padding = 56px client height, which is smaller than the
wrapped text in five of the eight boxes at desktop width. `overflow: hidden` then converts that overflow into silent
clipping (without it the text would spill over the reviewer line instead, which is the signal that the height is wrong).
The 480px result being milder shows why the bug is viewport-dependent and easy to miss if only one width is tested.

### Corrected state – `Q4-corrected-1400px.png` (the original page, viewport 1400px, green = fits)
Fix: delete the two declarations so `.quote` height is `auto`.
| Review | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---|---|---|---|---|---|---|---|
| clientHeight = scrollHeight | 100 | 52 | 36 | 64 | 88 | 52 | 64 | 64 |
| clipped? | no | no | no | no | no | no | no | no |

All eight boxes now size to their text at every width (including 480px), and "It stinks!" returns to 36px.
