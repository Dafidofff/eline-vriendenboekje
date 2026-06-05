# Redesign Tracker — Eline's Vriendenboekje

Modernising the site from its current 90s look (Comic Sans, offset text-shadows,
heavy emoji) into a **modern 2026 style that stays cute**.

The site is a single `index.html` (embedded CSS + JS) backed by Supabase. It is a
digital friend-book: friends fill in a page about Eline, pages are stored, browsed
in a "book view", and exported to an A4→A5 PDF booklet.

## Decision: two design phases

We are building **two** distinct cute-but-modern themes. We start with the bold one
and keep the soft one as a follow-up so each gets proper attention.

| Phase | Theme | Status |
|-------|-------|--------|
| **1** | **Playful & bold** — chunky rounded cards, sticker accents, bouncy micro-interactions, thick rounded type | 🚧 In progress |
| **2** | **Soft & dreamy** — frosted-glass cards, soft pastel mesh gradients, gentle shadows, calm rounded type | 📋 Planned |

Both keep the existing pastel palette and the bicycle / farm illustration charm
(see `assets/background.jpg`: cyclists, princess, cows, sheep, tomatoes — all
personal to Eline).

---

## Hard constraints (must NOT break — both phases)

These are contracts the JavaScript and PDF export rely on. Any redesign must keep them.

- **A4 dimensions.** `.front-cover`, `.page` (form) and `.filled-page` must stay
  `297mm × 210mm` (landscape). The PDF splits each landscape page into two A5
  portrait halves, so column balance matters.
- **Element IDs** used by JS: `book-view`, `form-view`, `book-pages`, `page-counter`,
  `btn-prev`, `btn-next`, `btn-pdf`, `zone`, `photo-input`, `photo-preview`,
  `upload-icon`, `upload-label`, `booklet`, `btn-save`, `saving-msg`, `success`,
  and every form field id (`fn`, `how`, `hobbies`, `food`, `future-self`, `memory`,
  `words`, `unique`, `learned`, `song`, `future`, `research-theme`, `tip-muziek`).
- **Field `name`s** drive the Supabase row (`phd-type`, `phd-other`, …). Keep them.
- **Classes built by `renderFilledPage()`** and targeted by `shrinkFilledAnswers()`:
  `filled-page`, `bg-layer`, `bg-overlay`, `col-divider`, `filled-inner`,
  `filled-columns`, `filled-col`, `label-wrap`, `section-label` (`sl-*`),
  `filled-card` (`c-*`), `q`, `a`, `filled-photo`.
- **Auto-shrink logic.** Form textareas need a constrained (flex) height so
  `scrollHeight > clientHeight` triggers shrink; filled `.a` blocks shrink against
  `.filled-card` height. Keep cards as flex columns with bounded heights.
- **onclick handlers**: `prevPage`, `nextPage`, `showForm`, `showBook`, `savePage`,
  `afterSave`, `savePdf`.

## html2canvas / PDF gotchas (learned + planned)

- **`backdrop-filter` (frosted glass) does NOT render in html2canvas.** This is the
  key reason Phase 1 (solid/opaque cards) goes first. **Phase 2 must fake glass for
  print** — e.g. capture a pre-blurred semi-opaque layer or a flat translucent
  fallback for the `.filled-page`/`.front-cover` used in the PDF, while the live
  screen keeps real `backdrop-filter`.
- CSS gradients, `box-shadow`, `border-radius`, and `transform: rotate` render fine.
- **Fonts**: switching off system Comic Sans to Google Fonts (Fredoka + Nunito)
  means the PDF depends on fonts being loaded — `savePdf()` now `await`s
  `document.fonts.ready` before capturing.
- **Capture is decoupled from on-screen scaling**: the front cover is cloned into
  the off-screen temp container and captured there at natural 297mm size, so
  responsive scaling of the *displayed* cover can never distort the PDF.

## Responsive approach

A4 pages are fixed in mm. For screen display we wrap each page in an `.a4-stage`
and scale it to fit narrow viewports with a transform + height-collapse helper
(`fitStages()` on load/resize). PDF capture is unaffected (always off-screen,
natural size).

---

## Phase 1 — Playful & bold (spec)

- **Type**: `Fredoka` (rounded geometric display) for titles/labels, `Nunito`
  (rounded, very readable) for body & answers.
- **Colour**: keep the existing pastel tokens; use them more confidently with
  gradient pairs (`pink→purple→blue`). Deep-plum ink (`--ink`) for text.
- **Surfaces**: opaque white cards, `border-radius` ~20px, chunky layered soft
  shadows, a bold colour accent bar per card.
- **Section labels**: sticker chips — slight rotation, drop shadow, gradient fill.
- **Inputs**: rounded, soft 2px border, coloured focus glow ring.
- **PhD-type checkboxes**: rendered as pill toggle-chips via `:has(:checked)`.
- **Buttons**: gradient pills with depth; bounce on hover/active.
- **Background (screen only)**: soft multi-radial pastel mesh + a few floating blobs.
- **Cover**: bold gradient, organic blob shapes, modern sticker badge, big Fredoka
  title, emoji chips, credit chip.
- **Success overlay**: modern rounded celebration card with confetti.
- **Empty state**: friendly modern card.
- **Motion**: micro-interactions guarded by `prefers-reduced-motion`.

## Phase 2 — Soft & dreamy (spec, for later)

- Same fonts but lighter weights; more whitespace, calmer.
- Frosted-glass cards (`backdrop-filter: blur`) over a soft animated pastel mesh.
- Print/PDF fallback for glass (see gotcha above) — **the main extra work item**.
- Thinner, lower-contrast accents; pastel-on-pastel; gentle floating motion.
- Could ship as a theme toggle (`data-theme="bold" | "dreamy"`) sharing one layout
  so both live in the same file — decide at start of Phase 2.

## Open questions for later

- Should Phase 2 be a runtime theme switch or a separate page? (Lean: theme switch.)
- Do we want a light confetti library or keep CSS-only confetti? (Lean: CSS-only.)
