# University instructions

Personal repository for university coursework: course materials as PDFs and their Markdown conversions.

## Layout

- One folder per course, named by course code (`ECON315/`).
- Inside a course, one folder per material type (`lectures/`, `problem_sets/`), each with:
  - `pdf/`: original files, unchanged.
  - `md/<name>.md`: the conversion, named after its PDF.
  - `md/assets/<name>/`: that conversion's images.

## Python

Use `uv run` from the repository root; never the system interpreter. Tools available: `markitdown`, `pdfplumber`, `pypdfium2`, `pillow`, `polars`, `duckdb`, `ruff`, `pyrefly`, `rumdl`. Poppler CLIs (`pdfimages`, `pdftoppm`, `pdftotext`) are on the system.

## Converting PDFs to Markdown

Goal: a Markdown file that preserves all content and the document's shape, and reads better than the PDF. Extracted text is a draft; the rendered pages are the authority. Always view every page before writing.

### Content

- Keep the wording verbatim: no paraphrasing, summarising, or correcting the author's typos and wording.
- Normalise only extraction artefacts: wrong typographic quotes (`”game”` → `"game"`), words glued together, and hard line breaks inside sentences (one line per paragraph or list item).
- Rejoin words hyphenated only by a line break (`monopo-list` → `monopolist`); keep real hyphens that fall at a line end (`unit-elastic`, `two-part`).
- Drop page furniture: page numbers, running headers and footers, decorative title-slide elements.
- Add nothing except the markers, headings, and alt text these rules define.
- Keep a deck's or document's own title even when it disagrees with the file name.

### Header

```markdown
# <document title>

<course code> · <course name> · <term> · Instructor: <name>

<!-- source: ../pdf/<name>.pdf (<page count and kind>) -->
```

Use only the metadata fields the source shows.

### Slide decks

- `#` deck title, `##` section-divider slides, `###` slide titles.
- Merge "(cont.)" slides under the first slide's heading.
- Put `<!-- slide N/M -->` before each numbered slide's content; for merged slides, include the original title: `<!-- slide 2/10: Strategic situations (cont.) -->`.
- In-slide bold labels become a `**bold**` line, not a heading.

### Documents (problem sets, handouts)

- `## Problem N` per top-level problem; the word "Problem" is the only added text.
- Sub-parts are list items with their labels kept as text: `- (a) …`. Use the same for roman labels: `- (i) …`.
- Put `<!-- page N/M -->` before the first block that starts on that page; when a page break falls mid-paragraph, put it after that paragraph.
- Text between sub-parts stays a top-level paragraph and splits the list.

### Lists, emphasis, tables

- Preserve list nesting and numbering exactly.
- Italic or coloured key terms and titles become `*italic*`; bold stays `**bold**`.
- Tables become Markdown tables, kept where the source places them, with math cells in LaTeX.
- When a list contains block content (images, display math), separate all its items with blank lines.

### Math

- Use LaTeX: inline `$…$`, display `$$…$$` on separate lines.
- Rebuild math from the rendered page, not from extracted text: extraction drops sub/superscripts and turns commas into semicolons (`aA; aB` is `$a_A, a_B$`).
- Keep the source's layout: centred equations stacked in `gathered`, equations inside a list item indented under that item.

### Figures

- Place each figure where the source places it; a figure belonging to a sub-part is indented inside that list item.
- Raster images (`pdfimages -list` shows them): extract with `pdfimages -all`, composite any soft mask onto white, save as JPEG (quality 85) for photos, with a descriptive name (`john-nash.jpg`).
- Vector figures (TikZ plots, game trees; `pdfimages -list` is empty): render a crop with `pdftoppm -r 300 -png -singlefile -x -y -W -H`, named `figure-N.png` or after the source's label (`game-tree-1.png`).
  - Crop from the top of the figure to the bottom of its caption, so the caption is in the image; do not repeat the caption as a text line. Figures can extend beside their caption, so never cut between figure and caption.
  - Find the crop box with `pdfplumber`: the union of chars, lines, curves, and rects in a rough area, plus 4 pt padding. Then check that no char straddles the rough area's edge, and view every crop.
- Slide images with a separate caption keep the caption as a text line below the image (a raster image has no caption inside it).
- Alt text starts with the caption (`Figure 1: …`) and describes what the figure conveys, without solving anything:
  - Plots: exact line endpoints and labels.
  - Game trees: movers, actions, Nature's probabilities, terminal actions, information sets, and marks such as arrows.
- Read exact values from the PDF's drawing data (`pdfplumber` line, curve, and point coordinates), not by eye. Crossing dashed curves for information sets are easy to misread.

### Verification

- Compare word counts against `pdftotext` (or the markitdown draft): every difference must be explained (merged "(cont.)" titles, added "Problem" headings, labels inside figures).
- Run `uv run rumdl check <file>`. MD013 (line length) is expected from the one-line-per-paragraph rule, and MD036 (emphasis as heading) from bold slide labels; fix anything else.
- Check that the `source:` path resolves and every image link exists.
