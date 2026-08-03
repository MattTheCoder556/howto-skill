---
name: howto
description: >-
  Turn a tested feature — or a validation sheet's requirements — into a
  user-facing how-to document with annotated screenshots, so you can find out
  whether ordinary users can actually complete the tasks unaided. Always
  includes screenshots; always asks what output format the document should be.
  Trigger: the user types `/howto`, or says "write a how-to", "turn this into
  user instructions", "make a guide users can follow", "can users actually do
  this?".
---

# /howto — tested feature → user-facing how-to with screenshots

Produces a document a real user can follow with nobody sitting next to them,
and which doubles as a usability test: every task carries a *Can you do it?*
box, so what comes back tells you where the software fails to explain itself.

**Two things are non-negotiable: it always contains screenshots, and you always
ask what format the output should be.**

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

**Annotate before cropping.** Drive the UI, draw a highlight on the target, and
capture. If a `hl.mark()`-style helper is available (draws a coloured box, a
label above it, and dims the rest of the page) use it — the dimming removes all
ambiguity about which of several similar buttons is meant.

**Crop to the control, not the window.** A 1600px screenshot renders the button
about 8px wide once embedded. `scripts/crop_highlights.py` finds the highlight
colour and crops around it with padding:

```bash
python3 scripts/crop_highlights.py --map map.json --out <doc-dir>/screenshots
```

where `map.json` is `{"/tmp/shot1.png": "01-create-button", ...}` — numbered in
document order so the folder reads in sequence. Keep enough surroundings that
the reader can tell where they are (a nav tab, a page heading).

**Always tell the reader the annotation is yours.** Put a note near the top:
the coloured box and dimming are added by you and are not part of the product,
or someone will hunt for a pink box on their screen. Say if the captures come
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

## 5. Ask the format, then write

**Always ask — never assume.** Offer:

| Format | When |
|---|---|
| **Markdown + `screenshots/`** | lives in a vault or repo, edited often, diffable |
| **PDF** | circulated for signature or filed as a controlled record |
| **Word (.docx)** | someone downstream must edit it |
| **Single-file HTML** | emailed or intranet-hosted, images inlined |

Ask **where it goes** in the same breath.

Markdown is the working format; the rest are conversions.

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

### Word and HTML

```bash
pandoc how-to.md -o how-to.docx --resource-path=.
pandoc how-to.md -o how-to.html --embed-resources --standalone
```

If they choose any converted format, **still keep the Markdown and the
`screenshots/` folder** — they are the editable source, and the PDF must be
regenerated from them rather than edited. Say so.

## Rules

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
