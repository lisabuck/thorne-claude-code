---
name: thorne-custom-element-tabs
description: Use this skill when Lisa wants a Thorne Group tab embed (or any embed) delivered as a Wix Studio CUSTOM ELEMENT with auto-height — so the content always finishes exactly at the bottom with no secondary scroll and no white space at any screen width. Triggers include "custom element", "auto height", "content finishes exactly at the bottom", "no scroll no white space", "the container never matches", "Wix hydration lag", "FOUC", "reduce the delay", or when the fixed-lock approach (thorne-pill-tabs skill) keeps producing scroll-vs-gap complaints. This skill is the DELIVERY MECHANISM; pair it with thorne-pill-tabs for the tab shell/typography and copy-writing for wording.
---

# Thorne Group · Wix Custom Element Tabs · Auto-Height Delivery Spec

Proven on the Developments tabs (July 2026). Replaces the fixed-height iframe embed with a
Custom Element served from GitHub via jsDelivr. The element sizes itself to the active tab,
Wix's section hugs it, and the page below moves flush up/down on tab switch. This eliminates
the entire scroll-vs-white-space category: there is no fixed height to be wrong.

---

## WHEN TO USE THIS vs FIXED LOCKS

| Situation | Use |
|---|---|
| Lisa reports "scroll if short, white space if long" and no number works | THIS skill (auto-height CE) |
| Wix breakpoints (1000/750) don't match embed breakpoints (960/720) | THIS skill, or re-cut media queries to Wix bands |
| Simple embed, fixed height acceptable, no CE appetite | thorne-pill-tabs fixed locks |

With auto-height the tabs intentionally end at DIFFERENT points (taller tab = taller page).
That is correct: the container follows the open tab exactly, so nothing ever trails or scrolls.
The old "all tabs must end equal" rule exists only for fixed containers — state this to Lisa.

---

## 1 · ARCHITECTURE

One JS file defines a custom element, e.g. `<thorne-dev-tabs>`:

```js
(function () {
  var FONTS = "…@font-face blocks…";   // JSON-encoded strings via json.dumps
  var CSS   = "…all styles…";
  var HTML  = "…body markup…";
  class ThorneDevTabs extends HTMLElement {
    connectedCallback() {
      if (this._init) return; this._init = true;
      if (!document.getElementById('thorne-metric-fonts')) {
        var fs = document.createElement('style');
        fs.id = 'thorne-metric-fonts'; fs.textContent = FONTS;
        document.head.appendChild(fs);
      }
      var root = this.attachShadow({ mode: 'open' });
      root.innerHTML = '<style>' + CSS + '</style>' + HTML;
      /* run any behaviour (photo rotation etc.) HERE programmatically */
    }
  }
  if (!customElements.get('thorne-dev-tabs')) {
    customElements.define('thorne-dev-tabs', ThorneDevTabs);
  }
})();
```

### Shadow-DOM gotchas (all three caused real bugs)
1. **`:root` does NOT work in shadow DOM.** Tokens silently resolve to nothing → Times New
   Roman text, transparent active pill. Change the token block selector to **`:host {`**.
   Diagnostic: computed fontFamily "Times New Roman" = token scope bug.
2. **`@font-face` does NOT register inside a shadow root.** Inject the font-face CSS into
   `document.head` once (guarded by an id check), keep the rest of the CSS in the shadow.
3. **`<script>` tags inside `innerHTML` never execute.** Re-implement behaviour (e.g. the
   4s crossfade photo rotation) as real code in `connectedCallback`, querying via
   `root.getElementById(...)`.

### Replace the html/body rules with :host
```css
:host { display:block; background:#FAF9F6; width:100%; max-width:100%; overflow-x:hidden;
  color:var(--wg); font-family:var(--f); font-weight:300; -webkit-font-smoothing:antialiased; }
```

### Auto-height CSS (differences from the fixed-lock shell)
- `.tab-panels`: NO height, NO overflow:hidden
- `.tab-panel`: NO height:100%
- Delete every `@media … .tab-panels { height: }` lock
- No postMessage script needed — the element is in the page DOM; the page reflows natively.

---

## 2 · SPEED — the 528KB → 120KB rule (fixes the "Wix delay")

The element cannot define until the WHOLE script downloads and parses. Base64 photos inside
the JS delay first paint massively. Real case: 528KB inline → 120KB with photos external.

- Keep FONTS inline (text must render styled immediately).
- Move every photo out as a real .jpg in the repo; reference via
  `https://cdn.jsdelivr.net/gh/lisabuck/thorne-claude-code@main/<name>.jpg`.
- Photo cells are CSS-sized (aspect-ratio / explicit heights), so layout is complete before
  images arrive — zero layout shift while they stream in.
- Use clean lowercase hyphenated filenames (`dev-138a.jpg`) — Lisa uploads manually; exact
  names matter.

### Preload (second lever — Wix loads CE scripts late)
Wix Settings → Custom Code → + Add Custom Code → paste in **Head**, applied to the page:
```html
<link rel="preconnect" href="https://cdn.jsdelivr.net" crossorigin>
<link rel="preload" as="script" href="https://cdn.jsdelivr.net/gh/lisabuck/thorne-claude-code@main/<FILE>.js" crossorigin>
```

### FOUC mask (Lisa's standard — goes in Wix Custom CSS / global.css)
```css
thorne-dev-tabs:not(:defined) {
  opacity: 0; display: block;
  min-height: 80vh;          /* holds layout steady while Wix boots */
  background: #FAF9F6;       /* warm white — NEVER black */
}
thorne-dev-tabs:defined { animation: smoothReveal 0.6s ease-in-out forwards; }
@keyframes smoothReveal {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}
```

### Honest floor
Some delay is Wix's own boot sequence (it initialises custom elements after the native page
paints — proven April 2026 with a 9.9KB embed that still lagged 1–2s). The above cuts the
controllable part to near-minimum; the fade masks the rest. Never promise zero delay.

---

## 3 · jsDelivr / GitHub WORKFLOW

- URL pattern: `https://cdn.jsdelivr.net/gh/lisabuck/thorne-claude-code@main/<file>`
- jsDelivr caches `@main` hard. After ANY update, purge:
  `https://purge.jsdelivr.net/gh/lisabuck/thorne-claude-code@main/<file>`
  (or reference an immutable commit: `@<sha>`)
- **jsDelivr 403-blocks Claude's container IP.** Verify files via
  `raw.githubusercontent.com` (expect 200) and have Lisa open the cdn URL in her browser
  ("wall of code text = working"). Do not report the 403 as a file problem.
- Lisa cannot push from chat — deliver files via present_files; she uploads to repo root.

---

## 4 · WIX SETUP (give Lisa these exact non-technical steps)

1. Delete the old embed element on the page.
2. **+ Add Elements → Embed & Social → Custom Element** → drag into place.
3. Click it → **Choose Source** → **Server URL** = the jsDelivr .js URL;
   **Tag name** = exactly the element name (e.g. `thorne-dev-tabs`).
4. Layout: **X 0, width stretch/100%**, **Height = Fit content / Auto / Hug** — never a
   fixed px number. (If no fit-content option exists, screenshot and reassess.)
5. Section: height **Fit content**, padding 0 all sides, fill `#FAF9F6`,
   "Apply max width" ON (~1265px).
6. Custom Code preload snippet (section 2) into Head for that page.
7. FOUC CSS (section 2) into the site's Custom CSS.
8. **Publish and test on the LIVE page** — custom elements often don't run in the editor
   canvas. Check: fade-in appears; switching tabs moves the content below flush up/down
   with no gap and no scrollbar; photo rotation runs.

---

## 5 · BUILD & VERIFY PROCESS

1. Build the tab embed as HTML first per thorne-pill-tabs (auto-height variant).
2. Convert: extract `<style>` and `<body>` (minus scripts) → JSON-encode with
   `json.dumps` (backslash-safe; NEVER regex-substitute the encoded strings back in —
   use string slicing around the match to avoid `re.error: bad escape`).
3. Apply the three shadow-DOM fixes (§1) — especially `:root` → `:host`.
4. Externalise photos (§2), write the .jpg files for Lisa to upload.
5. Playwright verification with CDN routing:
   - `page.route('https://cdn.jsdelivr.net/**', …)` fulfilling from local files
   - Assert per width (test ≥4 widths incl. 320): element height == content-below top
     (flush), height CHANGES on tab switch (click the radio inside `shadowRoot`),
     no horizontal scroll, computed fontFamily starts "Metric", active label
     background rgb(57,53,51).
6. Deliver .js + photos via present_files with the Wix steps (§4).

---

## 6 · COPY NOTE

Body copy and headings are **sentence case** (per Lisa, July 2026 — she corrected the
title-case For Sale tab). Property names ("Contemporary Golf Course Residence") and place
names stay as proper names. Eyebrows are uppercase via CSS, not in the copy. All other
copy rules per the copy-writing skill: verbatim, NZ spelling, no em-dashes.

End of spec.
