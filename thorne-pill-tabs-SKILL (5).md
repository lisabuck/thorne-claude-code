---
name: thorne-pill-tabs
description: Use this skill when Lisa asks for any Thorne Group multi-tab embed that uses the centred pill toggle and equal-height locking system — e.g. the Services tabs (Architecture + Build / Build Only / Renovation), Developments tabs, or any new tabbed section where each tab holds different content but all tabs must end at exactly the same height with no secondary scroll. Triggers include "pill tabs", "tabs like the services tab", "tabs like developments", "make a tabbed section", "equal height tabs", "tab toggle", or any request for 2-5 tabs that switch content under a rounded pill control. This is the tab-shell + locking spec; pair it with the copy-writing skill for wording and the values-tab skill only if matching Why-page proportions.
---

# Thorne Group · Pill-Tabs Embed · Master Spec

This skill encodes the reusable tab system built for `thorne_services_tabs.html`: a centred pill toggle, fluid 50+-readable typography, and height handling that avoids any gap or secondary scroll under the tabs. Height is handled EITHER by auto-height (embed reports its size to Wix — preferred, §3.4d) OR by a per-breakpoint fixed lock (fallback when Wix can't auto-resize, §3.3). Use it for any new tabbed Wix embed. The **content** of each tab varies; the **shell, type scale, and height method** stay identical.

---

## ABSOLUTE NON-NEGOTIABLES

| Rule | Value |
|---|---|
| File format | Single self-contained HTML file, no external dependencies |
| Wix iframe budget | **Under 720 KB total** (hard limit — embed fails above this) |
| All fonts | Metric Light 300, Regular 400, Medium 500 — embedded base64 WOFF2 |
| Background | `#FAF9F6` via `!important` on `html` AND `body` (defeats Wix dark mode) |
| Company name | "Thorne Group" — never "Thorne Group Homes" |
| Top nav / footer | Stripped — Wix provides header and footer |
| Text alignment | Left-aligned only, never justified |
| Strong text | `<strong>` is weight **500**, never 700 |
| Copy | Verbatim from brief — never reword, summarise, or invent. Flag any change before applying. NZ spelling, no em-dashes, no exclamation marks, no luxury language, first person plural. |
| Photos / testimonials | Never fabricate or mislabel. Use only supplied assets. |

---

## 1 · CSS ROOT TOKENS (copy verbatim)

```css
:root {
  --ww:    #FAF9F6;   /* warm white background */
  --soft:  #EDEAE3;   /* pill capsule bg, card bg */
  --wg:    #393533;   /* warm grey — headings, active pill, strong text */
  --mid:   #7a746e;   /* mid grey — eyebrows, inactive labels, captions */
  --read:  #54504c;   /* darker reading colour — body copy (50+ readability) */
  --rule:  #c8c3bb;   /* hairline rules, borders */
  --olive: #7a7752;   /* accent — sub-eyebrows, quote marks */
  --f:     'Metric','Helvetica Neue',Helvetica,Arial,sans-serif;
  --pad:   clamp(28px, 6vw, 96px);   /* side gutters */
}
```

---

## 2 · THE PILL TAB CONTROL (current standard — copy verbatim)

Centred rounded capsule; active tab is a solid warm-grey pill, inactive tabs are plain spaced text. Gap and inner padding are fluid so the pill **widens on larger screens** instead of bunching in the middle.

```css
.tab-bar {
  display:inline-flex; justify-content:center; align-items:center;
  gap:clamp(4px, 0.8vw, 16px); margin:40px auto 0; padding:clamp(6px, 0.5vw, 9px);
  background:var(--soft); border-radius:999px;
  box-shadow:0 1px 2px rgba(57,53,51,0.06);
}
.tab-bar-wrap { display:flex; justify-content:center; }
.tab-label {
  display:flex; align-items:center; justify-content:center; text-align:center;
  padding:14px clamp(32px, 3.4vw, 64px); border-radius:999px;
  font-size:13px; font-weight:500; letter-spacing:0.2em; text-transform:uppercase;
  color:var(--mid); cursor:pointer; transition:all 0.25s ease; line-height:1.2;
  white-space:nowrap;
}
.tab-label:hover { color:var(--wg); }
/* one selector line per tab */
#tab-1:checked ~ .tab-bar-wrap .tab-bar label[for="tab-1"],
#tab-2:checked ~ .tab-bar-wrap .tab-bar label[for="tab-2"],
#tab-3:checked ~ .tab-bar-wrap .tab-bar label[for="tab-3"] {
  background:var(--wg); color:var(--ww); font-weight:500;
}
```

### Tab markup (radio-driven, no JS)

```html
<!-- Hidden radios: siblings of .tab-bar-wrap AND .tab-panels -->
<input type="radio" name="tab" id="tab-1" class="tab-radio" checked>
<input type="radio" name="tab" id="tab-2" class="tab-radio">
<input type="radio" name="tab" id="tab-3" class="tab-radio">

<div class="tab-bar-wrap">
  <div class="tab-bar">
    <label for="tab-1" class="tab-label">Architecture + Build</label>
    <label for="tab-2" class="tab-label">Build Only</label>
    <label for="tab-3" class="tab-label">Renovation</label>
  </div>
</div>

<div class="tab-panels">
  <div class="tab-panel" id="panel-1"> <div class="sec-split">…</div> </div>
  <div class="tab-panel" id="panel-2"> <div class="sec-split">…</div> </div>
  <div class="tab-panel" id="panel-3"> <div class="sec-split">…</div> </div>
</div>
```

**Critical:** the radio inputs, `.tab-bar-wrap`, and `.tab-panels` must all be siblings (direct children of `<body>`) so the `~` sibling selectors work. Use `display:none/flex` to toggle panels (more reliable than `visibility` in Wix iframes). For 2 or 4-5 tabs, add/remove a radio + label + panel and the matching `:checked` selector line.

### Mobile pill (inside `@media (max-width:720px)`)

```css
.tab-bar { flex-wrap:wrap; gap:3px; padding:5px; border-radius:24px; max-width:calc(100% - 32px); }
.tab-label { padding:11px 16px; font-size:11px; letter-spacing:0.1em; }
```
Long labels wrap to two lines inside the capsule on narrow phones — acceptable. If single-line is required, shorten labels at mobile or allow horizontal scroll.

---

## 3 · THE EQUAL-HEIGHT LOCK SYSTEM (the core of this skill)

Every tab must end at **exactly the same height** at any given screen width, with **no content overflow** (which would cause an internal/secondary scroll) and **no horizontal scrollbar**. Lisa places native Wix content directly beneath, so a shifting bottom edge or stray scrollbar is unacceptable.

### How it works

```css
.tab-panels {
  height:1670px;        /* fixed per breakpoint — see §3.2 */
  overflow:hidden;       /* guarantees no secondary scroll */
}
.tab-panel {
  display:none;
  height:100%;
  box-sizing:border-box;
  padding:32px var(--pad) 48px;   /* small top pad => eyebrow sits just under tabs */
}
#tab-1:checked ~ .tab-panels #panel-1,
#tab-2:checked ~ .tab-panels #panel-2,
#tab-3:checked ~ .tab-panels #panel-3 {
  display:flex;
  align-items:flex-start;          /* TOP-aligned, not centred — kills the mid-panel gap */
}
.tab-panel > .sec-split { width:100%; }
```

`.tab-panels` has a **fixed height** (not `min-height`) at each breakpoint. `overflow:hidden` makes a secondary scroll structurally impossible. `align-items:flex-start` pins content to the top so the "01 —" eyebrow sits just under the pill and shorter tabs trail their gap at the bottom (not floating in the middle).

### 3.2 · Setting the locked heights — MEASURE, never guess

The lock height for each breakpoint = the **tallest tab's natural content height at the WORST-CASE width in that band**, plus a small buffer. The worst case is the **narrowest width still in that band** (text wraps most there) — for desktop that's ~961px, NOT 1280px.

**Always balance content first.** If one tab is much shorter (e.g. fewer testimonials), the lock leaves a big gap on it; if one is much taller, it forces ALL tabs up and creates gaps on the rest. Get the tabs naturally close BEFORE locking. Two levers: ADD genuine content to a short tab (a real extra testimonial / before-after pair / longer supplied copy), or REMOVE content from the tallest outlier. Real lesson: Build Only's 3rd testimonial made it the tallest and forced every tab to 1670; removing it dropped the desktop lock to 1510. Never pad or trim with fabricated content — only real assets, and surface removals to Lisa.

**Workflow (Playwright):**
1. Set `.tab-panels` height to `auto` and `overflow:visible` via JS.
2. For each tab, measure `getBoundingClientRect().height` of `#panel-N` (the full panel incl. padding) at every test width.
3. Take the tallest panel across all widths in each band = that band's required height.
4. Set the lock to that value + ~10-30px buffer.
5. Re-measure with the lock in place: confirm all 3 tabs report identical `.tab-panels` height and zero overflow at every width.

### 3.3 · Breakpoint bands & verified lock values (services tabs reference)

These are the values for the services-tabs content. **Re-measure for any new tab set** — different content = different locks. The *band structure* is fixed; the *numbers* are content-specific.

| Band | Media query | Worst-case width | Services lock (current build) |
|---|---|---|---|
| Desktop | (default, ≥961px) | 961px | `height:1700px` |
| Tablet | `@media (max-width:960px)` | 721px | `height:2115px` |
| Mobile | `@media (max-width:720px)` | 361px | `height:2540px` |
| Small phone | `@media (max-width:360px)` | 321px | `height:2540px` |
| Tiny phone | `@media (max-width:320px)` | 300px | `height:2820px` |

**Always include the ≤360px and ≤320px bands** — small phones (older iPhone SE, small Androids) wrap text more and overflow the main mobile lock if not given their own taller height. This was a real bug caught only by testing down to 320px.

It is correct and expected that the height *steps* between bands (taller on narrower screens). Within any single width, all tabs are pixel-identical — that is what matters.

### 3.4 · Wix container height = TAB BAR + PANELS, not just panels (critical — this caused a real scroll)

The locked `.tab-panels` height is NOT the embed's total height. The pill tab bar sits ABOVE the panels and adds roughly **96–100px on desktop/tablet and ~160px on mobile** (56px bar + a 40px top margin + wrap). So:

> **Total embed height = tab-bar block (~96–160px) + .tab-panels lock**

If Lisa sets the Wix HTML element to the *panel* number (e.g. 1510), the tab bar overflows by ~100px and Wix shows a secondary scrollbar. This is the #1 cause of "I still have a secondary scroll." Always give Lisa the **TOTAL** heights to put in Wix, measured with `document.documentElement.scrollHeight`, not the panel locks.

**Always measure and hand over total heights.** Example from services tabs (2 testimonials desktop, 1 mobile):

| Screen | .tab-panels lock | TOTAL embed (set Wix Section + iFrame to this) |
|---|---|---|
| Desktop ≥961 | 1700 | ~1800 |
| Tablet 721–960 | 2115 | ~2210 |
| Mobile 361–720 | 2540 | ~2665 |
| Small ≤360 | 2540 | ~2665 |
| Tiny ≤320 | 2820 | ~2945 |

If Wix only allows ONE fixed height across all breakpoints, that single number cannot serve both desktop (~1610) and mobile (~2640) without either a scroll or a big gap. In that case, advise per-breakpoint height in Wix, OR reduce content so the spread is acceptable. Do not silently trim real content to hit a number Lisa gave — surface the trade-off (raise the Wix height vs. cut content) and let her choose.

### 3.4b · Wix has TWO elements to size: the Section AND the embedded iFrame (this was the real scroll cause)

When Lisa reports a persistent scroll AND the embed verifies clean in Playwright, the problem is almost always the Wix setup, not the code. There are TWO nested elements and BOTH must be sized to match the total height:

1. **The Section** (e.g. "Section #section390") — the outer wrapper. Set its height to the TOTAL per breakpoint, Responsive behavior = **Fixed height**.
2. **The embedded iFrame / HtmlComponent** (e.g. "iFrame #html17") — the actual element holding the code, INSIDE the section. This is the one people miss. Click directly on the rendered tabs to select it (not the section).

**Real diagnosis from this build:** the section was correctly 1800 tall / 1265 wide, but the iFrame inside was **885 wide × 1127 tall** with Responsive behavior = **"Scale proportionally"**. That caused: (a) a secondary scroll (iframe 1127 < content ~1796, so the iframe scrolled internally), (b) a right-side gap (iframe 885 < section 1265), and (c) distortion on wide screens (scale-proportionally stretches the whole block).

**Correct iFrame settings:**
- Responsive behavior: **Fixed** (NOT "Scale proportionally" and NOT "Relative width")
- **X: 0** (so it starts at the section's left edge — any X offset leaves a side gap)
- **W:** match the section width (e.g. 1265), or Stretch
- **H:** the TOTAL height for that breakpoint (same number as the section)

Rule: **iFrame and Section must have identical width and height at every breakpoint.** iFrame shorter than section → internal scroll. iFrame narrower than section → side gap. Section taller than iFrame → gap underneath.

### 3.4c · Wide-screen trailing gap & the safe-lock principle

Because the lock must cover the WORST case (tallest tab at the narrowest desktop width ~961px, where text wraps most), wide screens — where the same content wraps less and is shorter — show trailing gap below the shorter tab. This is structural: one fixed height across a width range can't be tight at both ends.

- **Do NOT lower the desktop lock to kill the wide-screen gap** unless you confirm the embed never renders below the lock's safe width. Lowering it reintroduces overflow (→ scroll) at narrower desktop widths. Lisa's standing preference: **play safe — keep the higher lock, accept the wide gap, never risk the scroll.**
- **The proper fix for the wide gap is in Wix, not the code:** turn ON **"Apply max width"** on the section (~1265px). This stops the embed stretching on large monitors — wide screens get even side margins instead of short-content gaps — with zero scroll risk. Recommend this whenever the wide-screen gap matters.

### 3.4d · Auto-height (the real solution to "white space, but scroll if I reduce it")

The white-space-vs-scroll tension is fundamental to ANY fixed lock: the lock must cover the tallest tab at the narrowest desktop width, so every shorter tab/wider screen shows slack; lower it and the tallest tab overflows → scroll. **There is no single fixed height that avoids both.** The only true fix is to drop the fixed lock entirely and let the embed report its own height to Wix so the iframe resizes to fit the active tab.

**Code side (already built into the reference file):**
- Remove the fixed `height` and `overflow:hidden` from `.tab-panels`; remove `height:100%` from `.tab-panel`; delete ALL per-breakpoint `.tab-panels { height: }` locks. Each panel then sizes to its natural content (≈48px bottom padding is the only trailing space).
- Add a script before `</body>` that measures `document.documentElement.scrollHeight` and `postMessage`s it to the parent, firing on `load`, `DOMContentLoaded`, each radio `change` (tab switch), `resize`, and via `ResizeObserver`. (See reference file for the exact script.)

**Wix side — THE CRITICAL CAVEAT (be upfront with Lisa):**
Wix Studio's standard **Embed HTML / iframe element historically does NOT auto-resize from a postMessage sent inside the iframe** (browser security). The script is necessary but may not be sufficient. Before promising auto-height works:
1. Have Lisa check the embed's **Responsive behavior** dropdown for an "auto height" / "fit content" option. If present → use it; the script feeds it.
2. If the only options are Fixed / Scale proportionally / Relative width / Stretch, the basic iframe likely won't auto-resize. Reliable auto-height then needs EITHER (a) **Velo** code on the Wix page to receive the message and set the element's height, OR (b) embedding as a Wix **Custom Element** instead of an iframe.
3. **Test:** set section + embed to a starting height, open live preview, switch tabs. If the container resizes per tab → auto-height works. If it stays fixed → it doesn't; fall back to fixed locks (§3.3/§3.4).

Do NOT tell Lisa auto-height is done until the tab-switch resize is confirmed live. If it isn't supported, the honest answer is the fixed-lock trade-off (equal heights + accept the wide gap, safe-lock per §3.4c), or drop equal-height so each tab is natural (no gap, ends differ — fine since her native Wix content sits below the whole embed, not below each tab).

**Auto-height script (place before `</body>`):**
```html
<script>
(function () {
  function report() {
    var h = Math.max(document.documentElement.scrollHeight,
                     document.body ? document.body.scrollHeight : 0);
    try { window.parent.postMessage({ type:'WIX_EMBED_HEIGHT', height:h }, '*'); } catch(e){}
    try { window.parent.postMessage('embed-height:' + h, '*'); } catch(e){}
  }
  window.addEventListener('load', function(){ report(); setTimeout(report,300); });
  document.addEventListener('DOMContentLoaded', report);
  document.querySelectorAll('input[name="tab"]').forEach(function(r){
    r.addEventListener('change', function(){ report(); setTimeout(report,60); });
  });
  if (window.ResizeObserver) new ResizeObserver(report).observe(document.body);
  window.addEventListener('resize', report);
  setTimeout(report, 100);
})();
</script>
```
For auto-height, ALSO remove from CSS: the `height` + `overflow:hidden` on `.tab-panels`, the `height:100%` on `.tab-panel`, and every `@media … .tab-panels { height: }` lock.

### 3.5 · Reduce content on mobile to shorten tabs

Mobile tabs get long because the aside (testimonials / before-after) stacks under the text. Show **one testimonial / one before-after pair per tab on mobile** by hiding extras in the `@media (max-width:720px)` block — keeps desktop rich, mobile short:

```css
@media (max-width:720px) {
  .tab-aside .testimonial:nth-of-type(2) { display:none; }
  .ba-stack .ba-pair:nth-of-type(2) { display:none; }
}
```
After hiding, re-measure the mobile bands and re-lock — the asides shrink so text usually becomes the height driver. (This cut services mobile from 3000 → 2540.)

### 3.6 · Padding a structurally short tab with a text-only pull-quote

Sometimes one tab just has less content than the others (e.g. Build Only has ~700px less copy than the text-heavy A+B and Renovation tabs). No testimonial arrangement fixes this — testimonials are too "chunky" as height units (one is 650–1100px tall on a phone), so adding/removing one always overshoots or undershoots, and trimming quotes to single sentences BREAKS the testimonial card layout (photo/name float beside a tiny quote).

The clean fix: add a **text-only pull-quote** (a real client quote, verbatim, no photo/card) into the short tab's text column, between the body copy and the CTA. It pads height without the card-layout problem and reads as editorial. Make it breakpoint-scoped if only needed on some sizes:

```css
.mobile-pullquote { display:none; }
@media (max-width:720px){
  .mobile-pullquote { display:block; margin:8px 0 36px; padding:28px 0 0;
    border-top:1px solid var(--rule); font-size:21px; line-height:1.6;
    font-weight:300; color:var(--wg); }
  .mobile-pullquote cite { display:block; margin-top:18px; font-size:13px;
    font-weight:500; letter-spacing:0.14em; text-transform:uppercase;
    color:var(--mid); font-style:normal; }
}
```
```html
<blockquote class="mobile-pullquote">&ldquo;…verbatim client quote…&rdquo;<cite>Name</cite></blockquote>
```
This cut the services mobile gap from ~700–800px down to ~250px. Quote text must be verbatim from a real testimonial — never invented.

**Honest limit to state to Lisa:** equal endpoints AND zero gap cannot both be true when tabs have genuinely different content volumes. The shortest tab will always either trail (if heights are locked equal) or end early (if heights are natural). Pull-quotes and content-balancing narrow the gap; they don't eliminate it. Surface this rather than iterating endlessly.

---

## 4 · TYPOGRAPHY (50+ readable — copy verbatim)

Reading text is **Metric Regular (400)** (not Light) for legibility at size. Display H2 stays Light (300). **Body and lead are fixed sizes (18px / 20px), not fluid** — Lisa found the fluid `clamp` versions grew too large on wide screens. Keep body at 18px, lead at 20px, across all desktop/tablet widths. (Eyebrows and H2/H3 may still use `clamp` as below.)

```css
/* eyebrows / small labels — readable: larger, less tracking, weight 500 */
.sec-eyebrow { font-size:clamp(15px,1.15vw,17px); font-weight:500; letter-spacing:0.16em;
  text-transform:uppercase; color:var(--mid); margin-bottom:24px; line-height:1.5; }
.sub-eyebrow { font-size:clamp(15px,1.15vw,17px); font-weight:500; letter-spacing:0.16em;
  text-transform:uppercase; color:var(--olive); margin-bottom:18px; line-height:1.5; }

.sec-h2 { font-size:clamp(36px,3.6vw,56px); font-weight:300; line-height:1.08;
  letter-spacing:-0.02em; color:var(--wg); }
.sub-h3 { font-size:clamp(26px,2.2vw,32px); font-weight:400; line-height:1.3;
  letter-spacing:-0.01em; color:var(--wg); margin-bottom:28px; }

.sec-text .sec-lead { font-size:20px; font-weight:400; line-height:1.7;
  color:var(--wg); margin-bottom:40px; }
.sec-text .sec-lead strong { font-weight:500; }
.sec-text .sec-body p { font-size:18px; font-weight:400; line-height:1.85;
  color:var(--read); margin-bottom:24px; }
.sec-text .sec-body p strong { font-weight:500; color:var(--wg); }

/* testimonial labels — name readable, service tag visible (not pale) */
.testimonial-attr { font-size:14px; font-weight:500; letter-spacing:0.14em;
  text-transform:uppercase; color:var(--wg); line-height:1.5; }
.testimonial-attr .label { display:block; font-size:12px; color:var(--mid);
  margin-top:5px; letter-spacing:0.14em; font-weight:400; }
.testimonial-quote { font-size:clamp(17px,1.35vw,20px); font-weight:400; line-height:1.9; color:var(--wg); }

/* before/after tags & captions */
.ba-tag { font-size:12px; font-weight:500; letter-spacing:0.14em; text-transform:uppercase;
  color:var(--ww); background:rgba(57,53,51,0.85); padding:6px 12px; }
.ba-tag.after { background:rgba(122,119,82,0.92); }
.ba-caption { font-size:13px; font-weight:500; letter-spacing:0.12em; text-transform:uppercase;
  color:var(--mid); padding-top:6px; }
```

### Mobile typography (inside `@media (max-width:720px)`)
```css
.sec-h2 { font-size:32px; }
.sec-text .sec-lead { font-size:20px; }
.sec-text .sec-body p { font-size:19px; }
.testimonial-quote { font-size:18px; }
```

---

## 5 · PANEL LAYOUT

Each panel is a 2-column split: text left, aside (testimonials or before/after) right.

```css
.sec-split { display:grid; grid-template-columns:7fr 5fr; gap:64px; }
.sec-split.photo-left { grid-template-columns:5fr 7fr; }   /* aside on left variant */

@media (max-width:960px) {
  .sec-split, .sec-split.photo-left { grid-template-columns:1fr; gap:40px; }
  .sec-split.photo-left .sec-text { grid-column:1; grid-row:1; }
  .sec-split.photo-left .tab-aside, .sec-split.photo-left .ba-stack { grid-column:1; grid-row:2; }
}
```

- **Testimonial card**: `.testimonial` > `.testimonial-photo` (`.testimonial-avatar img` + `.testimonial-attr` with `.label` service tag) + `.testimonial-body` (`.testimonial-mark` quote glyph + `.testimonial-quote`).
- **Before/after**: `.ba-stack` > `.ba-pair` > `.ba-cell` (`.ba-tag` Before/After + `img`) + `.ba-caption`.
- **CTA**: `.cta-link` — uppercase, bordered, arrow `::after`, fills warm-grey on hover.

---

## 6 · IMAGE PROCESSING

- PIL + `ImageOps.exif_transpose`, convert RGB.
- Avatars: square crop, face-biased top (~0.05-0.10 top bias for couples/faces), 300px, JPEG q72.
- Before/after & landscape: 4:3 or 3:2, q70, progressive + optimize.
- Base64-embed inline. Keep total file under 720 KB.
- Repo images: pull via `raw.githubusercontent.com/lisabuck/thorne-claude-code/main/<EXACT NAME>` (case + spaces matter; the web listing can lag/miss files). API/codeload often rate-limited; raw URLs are reliable.

---

## 7 · BUILD CHECKLIST (run every time)

1. Read this skill + the copy-writing skill. Confirm exact verbatim copy with Lisa.
2. Build shell: tokens, fonts, pill control, panels, typography — copy CSS verbatim.
3. Drop in the supplied copy verbatim. Wire in supplied photos (cropped). Never fabricate.
4. **Balance content** so tabs are naturally close in height (add to short tabs / remove from the tallest outlier; only real assets; surface removals to Lisa).
5. **Mobile reduction**: hide the 2nd testimonial / 2nd before-after pair in the `@media (max-width:720px)` block so mobile tabs stay short.
6. Measure natural heights (Playwright) at worst-case widths per band: ~961 / 721 / 361 / 321 / 300.
7. Set the 5 locked heights (desktop / ≤960 / ≤720 / ≤360 / ≤320) to tallest + small buffer.
8. **Verify**: sweep widths 320→2560px. Confirm at every width: all tabs identical `.tab-panels` height, zero content overflow, no horizontal scroll. (Reusable check script below.)
9. **Measure TOTAL heights** (`document.documentElement.scrollHeight`) per band = panels + tab bar. Hand THESE to Lisa for the Wix element, not the panel locks (see §3.4). If Wix allows only one fixed height, surface the desktop-vs-mobile spread trade-off.
10. Confirm file < 720 KB, div tags balanced.
11. Deliver via `present_files` with the per-breakpoint TOTAL heights for Wix. Tell Lisa to set BOTH the Section AND the embedded iFrame to those totals (Fixed, X:0, full width) — and if a wide-screen gap bothers her, to turn on the section's "Apply max width" (~1265px). See §3.4b/3.4c.

### Reusable verification script
```python
from playwright.sync_api import sync_playwright
widths=[320,360,390,414,480,560,720,721,768,860,960,961,1024,1200,1280,1440,1600,1920,2560]
with sync_playwright() as p:
    b=p.chromium.launch()
    for w in widths:
        pg=b.new_page(viewport={"width":w,"height":700})
        pg.goto("file:///tmp/FILE.html"); pg.wait_for_timeout(250)
        hs=[]; ovf=[]; hscroll=False
        for i in (1,2,3):
            pg.evaluate(f"document.getElementById('tab-{i}').checked=true"); pg.wait_for_timeout(90)
            hs.append(round(pg.evaluate("()=>document.querySelector('.tab-panels').getBoundingClientRect().height"),1))
            ovf.append(pg.evaluate(f"""()=>{{const c=document.querySelector('.tab-panels');const s=document.getElementById('panel-{i}').firstElementChild;return Math.max(0,Math.ceil(s.getBoundingClientRect().bottom-c.getBoundingClientRect().bottom));}}"""))
            if pg.evaluate("()=>document.documentElement.scrollWidth")>w+1: hscroll=True
        print(w, "same" if len(set(hs))==1 else f"DIFF {hs}", "clean" if max(ovf)<=2 else f"OVF {max(ovf)}", "hscroll!" if hscroll else "")
        pg.close()
    b.close()
```
(Adjust the tab range `(1,2,3)` to the number of tabs.)

---

## 8 · REFERENCE FILE

`thorne_services_tabs.html` (repo: `lisabuck/thorne-claude-code`) is the canonical implementation: 3 tabs, pill control, fixed 18px/20px 50+ type. Build Only has 3 testimonials on desktop/tablet, 1 testimonial + a mobile pull-quote on phones. **Current version uses AUTO-HEIGHT** (§3.4d): no fixed `.tab-panels` locks, panels size to natural content, a postMessage script reports height to Wix. Pending confirmation that Wix resizes on tab switch — if it doesn't, fall back to the fixed 5-band lock (≈1700 / 2115 / 2540 / 2540 / 2820 panels; TOTAL Wix heights ≈ +100px tab bar, desktop ~1800). Both the Wix Section AND the iFrame inside it must match each other (Fixed, X:0, full width). Copy the shell for any new tab set and swap content; re-measure if using fixed locks.
