# howto-skill

A Claude Code plugin that turns a feature you have just tested — or the
requirements in a validation sheet — into a **user-facing how-to document with
annotated screenshots**.

The document doubles as a usability test: every task ends with a
*Can you do it?* box, so what comes back tells you where the software fails to
explain itself, not whether the reader is capable.

Ships the same skill for **Claude Cowork** too.

## Install (Claude Code)

```bash
claude plugin marketplace add MattTheCoder556/howto-skill
claude plugin install howto@howto-skill
```

Restart Claude Code. You now have `/howto` in every project.

Update later with `claude plugin update howto`.

## Install (Claude Cowork)

Copy `cowork-skill/` into your Cowork skills directory as `howto/`.

## What it produces

Every run produces **both**, never one or the other:

- **PDF in the qmsWrapper *Documentation* house style** — logo header, near-black
  headings on white, humanist sans, full-width bordered screenshots
- **Markdown + a `screenshots/` folder** — the editable source the PDF is built
  from, and what you edit for the next revision

Word (`.docx`) and single-file HTML are available on request, in addition to
those two rather than instead of them.

Both files are named by the house convention, differing only in extension:

```
qmsWrapper_<Module>_<YYYY-MM-DD>_<HHMM>_v<N>.<ext>
qmsWrapper_FormBuilder_2026-08-10_1432_v1.pdf
qmsWrapper_FormBuilder_2026-08-10_1432_v1.md
```

`<Module>` comes from `reference/modules.md` — the module registry shared with
the validation skill — so a how-to can be tied back to the test cases it came
from. The modules formerly called *Form Editor* and *Process Editor* are
**Form Builder** and **Process Builder**.

The PDF is produced by **printing HTML from Chrome**, which is how the existing
qmsWrapper documentation PDFs were made. It is deliberately *not* pandoc +
LaTeX: `pdflatex` cannot render `☐ → ▾ ⚠` without a per-character preamble,
needs `adjustbox` to stop oversized screenshots overflowing, and the result
looks like a LaTeX article rather than product documentation.

## Two rules the skill will not bend

1. **It always includes screenshots, and every one is annotated.** Each figure
   carries a pink box and label on the target with the rest of the page dimmed,
   then is cropped to it — so the reader is never left searching a full-window
   capture for the button. If screenshots cannot be captured the skill says so
   and asks, rather than quietly shipping a wall of text.
2. **It always delivers the PDF and the Markdown together.** The format is not a
   question; only where the document goes is, and it asks that before writing
   anything. A PDF with no source to regenerate it from is not a deliverable.

Image format is **PNG** and is not offered as a choice: lossless text so small
labels survive, flat UI colour is PNG's best case (it beats JPEG on size *and*
quality here), and it is supported everywhere.

## Scripts

| Script | Does |
|---|---|
| `highlight.py` | Draws the annotation on a **live** page you are driving with Playwright — pink box, label, rest of the page dimmed — before you capture. Position comes from the element's own bounding box. |
| `annotate_screenshot.py` | Draws the same annotation onto a PNG you **already have**. For captures that predate the document, targets with no element of their own (a pair of table columns), or one capture illustrating two steps. |
| `crop_highlights.py` | Finds the annotation highlight in a screenshot and crops around it with padding. A full-window capture renders the button ~8px wide once embedded; this fixes that. |
| `md_to_qmswrapper_html.py` | Markdown → self-contained styled HTML. Inlines every image as base64, so there are never missing-image boxes. |
| `html_to_pdf_chrome.py` | Drives Chrome's print engine to produce the PDF. Launches headless by default, or reuses an open browser with `--cdp`. |

### Typical run

```bash
# 1. annotate — either while driving the UI (highlight.py, inside your script)
#    or afterwards, on captures you already have:
python3 scripts/annotate_screenshot.py --spec figures.json

# 2. crop each annotation down to its target
python3 scripts/crop_highlights.py --map map.json --out doc/screenshots

# 3. markdown -> styled, self-contained HTML
python3 scripts/md_to_qmswrapper_html.py \
        doc/qmsWrapper_FormBuilder_2026-08-10_1432_v1.md /tmp/out.html \
        screenshots assets/qmswrapper-logo.png

# 4. HTML -> PDF via Chrome, same name as the Markdown
python3 scripts/html_to_pdf_chrome.py /tmp/out.html \
        doc/qmsWrapper_FormBuilder_2026-08-10_1432_v1.pdf
```

`figures.json` places the boxes; `map.json` maps capture path → output basename,
numbered in document order:

```json
{
  "/tmp/shots/step1.png": "01-create-button",
  "/tmp/shots/step2.png": "02-name-field"
}
```

```json
[
  {"src": "/tmp/shots/step1.png", "out": "/tmp/annotated/01-create-button.png",
   "regions": [{"box": [1315, 130, 128, 29], "label": "1. Click + Create Form"}]}
]
```

## Requirements

- **Pillow** — `pip install pillow` (annotating and cropping)
- **Playwright + Chromium** — `pip install playwright && playwright install chromium` (PDF)
- **pandoc** — optional, only for `.docx` output

## Notes

- Keep the Markdown and its `screenshots/` folder together — image links are
  relative, and moving one without the other breaks the pictures.
- Regenerate the PDF from the Markdown after edits rather than editing the PDF.
- Screenshots taken on a test environment show test data and a DEV banner. Fine
  internally; retake them on a clean tenant before anything customer-facing, and
  treat them as controlled content that needs recapturing whenever the UI changes.
- `assets/qmswrapper-logo.png` is the brand mark used in the PDF header.

## Licence

MIT
