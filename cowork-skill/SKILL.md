---
name: howto
description: >-
  Turn a tested feature — or a validation sheet's requirements — into a
  user-facing how-to document with annotated screenshots, so you can find out
  whether ordinary users can actually complete the tasks unaided. Always
  includes screenshots; always delivers both a .pdf in the qmsWrapper house
  style and the .md it was built from.
  Trigger: someone says "how-to", "/howto", "write a how-to", "turn this into
  user instructions", "make a guide users can follow", "can users actually do
  this?", or shares a tested feature and wants it written up for users.
---

# howto — tested feature → user-facing how-to with screenshots

Produces a document a real user can follow with nobody sitting next to them,
and which doubles as a usability test: every task carries a *Can you do it?*
box, so what comes back tells you where the software fails to explain itself.

**Two things are non-negotiable: it always contains screenshots, and it always
comes out as both a `.pdf` and the `.md` that produced it.**

The PDF is the readable artefact — house style, circulated, signed, filed as a
controlled record. The Markdown is the editable source, and the `screenshots/`
folder belongs with it. Hand over one without the other and the next revision
has nowhere to start from: nobody edits a PDF, they regenerate it.

## 1. Gather the source

In order of preference:

1. **A test run in this session** — the click paths are already known, and so is
   which steps are fiddly. This is the best case: write the warnings from what
   actually went wrong.
2. **A validation sheet** (`… Use Requirements.xlsx`) — one section per
   requirement, its `steps` column becoming the instructions. Read the sheet
   rather than guessing at it.
3. **Module guides or the QMS manual** — thinner, but workable.

Never invent a click path. If you have not seen the screen and no document
describes it, the honest move is to drive the UI and find out, or leave that
task out and say so.

**Carry the known defects across.** If the source records a failure, the how-to
must warn the reader before they hit it — see §4.

## 2. Screenshots — always, and take them yourself

A how-to without pictures is a list of guesses about what the user is looking
at. Capture them.

**What to shoot.** Every click that is not obvious from the page: the entry
point, anything easy to confuse with a neighbour, any multi-step interaction,
and the thing the reader is being asked to *check*. Skip screenshots of typing
into a plainly-labelled text box.

**Every figure must point at something.** A plain screenshot shows the reader
the page; it does not show them *which* of six similar buttons to press. So each
one gets a pink box round the target, a labelled tag, and the rest of the page
dimmed. The dimming is what removes the ambiguity — the eye goes to the one
place still at full brightness. A screenshot with nothing marked on it is a
screenshot the reader has to search, and searching is the failure this document
exists to detect.

Two ways to draw it, same output either way:

**a. While you drive the UI** — `scripts/highlight.py`:

```python
from highlight import mark, clear
mark(page, "document.querySelector('button.create')", "1. Click + Create Form")
page.screenshot(path="/tmp/shots/01.png")
clear(page)                     # or it appears in every later capture
```

The position comes from the element's own bounding box, so it cannot be a few
pixels out. Prefer this whenever you have a live page. `mark()` returns False
and says so when the selector finds nothing — do not ship that capture.

**b. On a capture you already have** — `scripts/annotate_screenshot.py`:

```bash
python3 scripts/annotate_screenshot.py --spec figures.json
python3 scripts/annotate_screenshot.py in.png out.png \
        --box 1315,130,128,29 --label "1. Click + Create Form"
```

Use it when the screenshot predates the document, when the target has no element
of its own (a pair of table columns, a region of a canvas), or when one capture
has to illustrate two different steps. Several regions on one figure show an
ordered pair of clicks — number the labels, and set `"label_pos": "below"` on
the second when a tag above it would cover the first.

Coordinates are hand-entered here, so **look at the result** before shipping it.
A box a hundred pixels off points confidently at the wrong thing, which is worse
than no box at all.

**Then crop to the control, not the window.** A 1600px screenshot renders the
button about 8px wide once embedded. `scripts/crop_highlights.py` finds the pink
and crops around it with padding:

```bash
python3 scripts/crop_highlights.py --map map.json --out <doc-dir>/screenshots
```

where `map.json` is `{"/tmp/shot1.png": "01-create-button", ...}` — numbered in
document order so the folder reads in sequence. Keep enough surroundings that
the reader can tell where they are (a nav tab, a page heading).

All three agree on `#e6007e`, so annotate → crop chains without configuration.

**Always tell the reader the annotation is yours.** Put a note near the top:
the pink box, the pink label and the dimming are added by you and are not part
of the product, or someone will hunt for a pink box on their screen. Say if the captures come
from a test environment with test data.

**Image format is PNG.** Do not offer this as a choice — it is settled:

- lossless text, so small labels and thin borders survive; JPEG puts halos on
  exactly the high-contrast edges UI is made of
- UI is large flat colour areas, PNG's best case — it beats JPEG on *size* here
  as well as quality
- universal support: Word, Confluence, Obsidian, GitHub, PDF export

WebP is ~25% smaller but still trips older tooling, which is a poor trade for a
controlled document. Only revisit if the user raises a specific constraint.

## 3. Write the document

**Voice.** Address the reader directly. Short numbered steps, one action each.
Name what they should end up seeing, not just what to click — *"You should end
up in the form builder, with Version: 1 under the name"* — because that is what
lets someone self-correct.

**Structure.**

- **Before you start** — required role or permissions, which tier or plan, what
  the reader will create and that it is disposable, how to name things so they
  can be cleaned up, and the note about annotations from §2.
- **One section per task**, numbered to match the source (`1.4`, `2.1`). Give
  each a short imperative heading.
- **A `> **Can you do it?** ☐` box** closing every task. Add extra boxes where a
  task has a distinct sub-check worth recording separately.
- **When you are done** — cleanup instructions, what to send back, and an
  explicit ask for *anywhere they had to guess, backtrack, or ask a colleague*,
  framed as more valuable than the pass/fail ticks. That is the actual output.
- **A footer** naming the source document and run date, and stating which tasks
  are expected to fail.

**Warn at the point of danger, not in a preamble.** A step that trips people
gets its warning immediately above it, in bold. Write these from what actually
went wrong during testing — those are the real ones.

**Mark skippable tasks** where they depend on tier, role or org configuration,
and say what the reader will see instead.

## 4. Known defects

Where the source records a failure, the reader must be told **before** they meet
it, and asked to record what they saw:

> **They should match.** They will not — expect roughly a two-hour gap.
> **This is a known defect. You are confirming it, not diagnosing it.**
>
> **Do the two times match?** ☐ Yes ☐ No — A: ......... B: .........

Never quietly omit a broken step. A how-to that walks someone into a defect with
no warning wastes their time and produces a bug report you already have.

## 5. Write it, then produce both files

**The output is settled: `.pdf` + `.md`. Do not ask which format.** The filename
is settled too — it follows the convention below. The only thing to confirm is
**which module** the how-to is filed under, and you do that before writing
anything.

Every run produces three things, handed over together:

| | |
|---|---|
| `qmsWrapper_<Module>_<date>_<time>_v<N>.md` | the working source, with relative `screenshots/…` links |
| `screenshots/` | the annotated PNGs from §2 |
| `qmsWrapper_<Module>_<date>_<time>_v<N>.pdf` | the house-style PDF, built from that Markdown |

Write the Markdown first and build the PDF from it — never the other way round,
and never author the PDF's HTML by hand. The Markdown is what gets edited when
the UI changes; the PDF is regenerated. An intermediate `<doc>.html` falls out
of the build and is scaffolding, not a deliverable — do not hand it over as if
it were one of the two outputs.

**If the PDF step fails**, say so plainly, hand over the Markdown and the
screenshots, and name what is missing (a CDP Chrome, Playwright, the logo
asset). Do not quietly deliver Markdown alone as though that were the whole job.

### Filename convention

```
qmsWrapper_<Module>_<YYYY-MM-DD>_<HHMM>_v<N>.<ext>
qmsWrapper_FormBuilder_2026-08-10_1432_v1.pdf
qmsWrapper_FormBuilder_2026-08-10_1432_v1.md
```

| Field | Rule |
|---|---|
| `qmsWrapper` | Fixed prefix, this exact casing. |
| `<Module>` | The module's filename token — CamelCase, no spaces, from `reference/modules.md`. Never invent one. |
| `<YYYY-MM-DD>` | The date the document was built. |
| `<HHMM>` | 24-hour build time, so two runs on one day never collide. |
| `v<N>` | Version, from 1, bumped on each redraft of the same how-to. |

**The `.pdf` and the `.md` share a name**, differing only in extension — they
are one document in two formats, and a reader holding the PDF has to be able to
find the source that produced it. Stamp the time once and use it for both; do
not let the two files pick up timestamps minutes apart.

The `screenshots/` folder keeps its plain name beside them — it is shared by
every version of the how-to, so it takes no timestamp.

A how-to spanning several modules takes the module it is *about*; if there is
genuinely no single one, ask which to file it under rather than inventing a
compound token.

No spaces, and no underscore inside any field: the name splits on `_` into
exactly five parts.

### Name the module, exactly

`<Module>` is not yours to phrase. Take it from **`reference/modules.md`**, the
registry lifted from the Validation Test Matrix workbook, which lists every
module's exact name and its filename token. A how-to filed under a name the
matrix does not use cannot be tied back to the test cases it came from, which is
the whole point of naming it after a module.

| Do not write | Write |
|---|---|
| Form Editor, Forms, Forms (Builder & Submissions) | **Form Builder** |
| Process Editor, Process / Workflow Engine, Workflow Engine | **Process Builder** |

This applies to the prose as well as the filename: if the document calls the
screen "the form editor", the reader is looking for something the product does
not call that. **If the module is not in the registry, stop and ask** — never
invent a name to get a document out.

### Other formats

Word and single-file HTML are **extras, produced only if the user explicitly
asks** — and in addition to the PDF and Markdown, never instead of them:

```bash
pandoc how-to.md -o how-to.docx --resource-path=.
pandoc how-to.md -o how-to.html --embed-resources --standalone
```

### PDF — use the house style, not pandoc's default

qmsWrapper documentation PDFs have an established look (see
`WrapperTesting/NewValidation/BenceHowTos/` for reference copies): a logo +
"Documentation" header rule, near-black headings on white, humanist sans body at
generous line-height, and full-width screenshots with a hairline border. They are
produced by **printing HTML from Chrome**, not by LaTeX.

Match it with the two scripts here:

```bash
python3 scripts/md_to_qmswrapper_html.py <doc.md> <out.html> <screenshots-dir> assets/qmswrapper-logo.png
python3 scripts/html_to_pdf_chrome.py <out.html> <out.pdf>     # needs a CDP Chrome on :9223
```

The first inlines every image as base64, so the HTML is self-contained and the
PDF never has missing-image boxes. The second drives Chrome's own print engine,
which is what produced the reference PDFs.

**Do not reach for `pandoc --pdf-engine=pdflatex`.** It cannot render `☐`, `→`,
`▾` or `⚠` without a per-character `\DeclareUnicodeCharacter` preamble, it needs
`adjustbox` to stop oversized images overflowing, and the result looks like a
LaTeX article rather than product documentation.

Tell the reader the PDF is generated: it must be rebuilt from the Markdown when
the UI changes, not edited in place.

## Rules

- **Both files, every time.** A how-to is a `.pdf` *and* the `.md` it was built
  from, handed over together with `screenshots/`. Do not ask which format, do
  not ship one alone, and do not treat the PDF as the document and the Markdown
  as a by-product — the Markdown is the source of the next revision.
- **The module name comes from the registry**, character-for-character, in the
  filename and in the prose. Not in `reference/modules.md`? Stop and ask. It is
  **Form Builder** and **Process Builder** — never "Editor".
- **Filenames follow `qmsWrapper_<Module>_<YYYY-MM-DD>_<HHMM>_v<N>`**, the `.pdf`
  and `.md` differing only in extension.
- **Every screenshot is annotated**, by `highlight.py` on a live page or
  `annotate_screenshot.py` on an existing capture, then cropped to the target.
  An unmarked full-window capture is not a figure — it is homework for the
  reader. Check hand-entered boxes by eye before shipping.
- **Screenshots are mandatory.** If you cannot capture them, say so plainly and
  ask whether to proceed without — do not silently ship a text-only document.
- **Never invent a click path.** Every step must come from a screen you drove or
  a document that describes it.
- **Relative image links** (`screenshots/…`) so the document and its folder
  travel together. Warn that moving one without the other breaks the pictures.
- **Warn before defects**, never after.
- **Do not grade the reader.** The framing is that failure to complete a task is
  the software's problem. Say so in the document.
- **Flag test-environment captures.** Screens showing test data and a DEV banner
  are fine internally, but must be retaken on a clean tenant before anything
  customer-facing — and they become controlled content, needing recapture
  whenever the UI changes.
