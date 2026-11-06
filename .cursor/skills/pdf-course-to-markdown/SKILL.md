---
name: pdf-course-to-markdown
description: Converts course PDFs into a polished docs/ lesson and a faithful origins/ source document, with selected images. Use when importing, rewriting, or updating learning documents from PDFs into docs/, origins/, and images/.
---

# PDF Course to Markdown

Turn a source PDF into two Markdown outputs while preserving teaching sequence, useful examples, and visual evidence:

| Output | Path | Purpose |
|--------|------|---------|
| Polished lesson | `docs/lessonN.md` | Structured rewrite for study and navigation |
| Original document | `origins/lessonN.md` | Faithful transcription of the PDF body for reference and hand-copy practice |

Images are shared under `images/lessonN/` and referenced by both files.

## Workflow

### 1. Inspect the target

1. Read the source PDF in full.
2. Read the existing `docs/` lesson, `origins/` document, README/index entry, and image directory.
3. Determine whether to replace an existing lesson or create a new one.
4. If the requested outcome is ambiguous, ask whether the user wants:
   - a polished structured rewrite for `docs/` (default, always produce `origins/` too);
   - a faithful transcription only;
   - a new document alongside the existing lesson.

Do not overwrite unrelated lessons or navigation without confirmation.

### 2. Build a content outline

Extract the article's teaching structure before writing. Typical sections include:

- learning goals and expected outcomes;
- prerequisites and foundational concepts;
- framework or technology progression;
- backend and infrastructure requirements;
- observability, evaluation, and deployment;
- project practice;
- learning method;
- career preparation, résumé examples, and interview topics;
- summary and recommended sequence.

Preserve the author's causal logic and learning order. Remove only page chrome and publishing noise such as payment labels, timestamps, navigation, comments, avatars, advertisements, and private contact details.

### 3. Inventory and classify images

Extract all embedded images to a temporary directory before selecting assets. Prefer PyMuPDF without adding it to the project dependencies:

```bash
python3 -m pip install --quiet --target /tmp/pdf-course-tools pymupdf
PYTHONPATH=/tmp/pdf-course-tools python3 - <<'PY'
import pymupdf
from pathlib import Path

source = Path("pdfs/source.pdf")
output = Path("/tmp/pdf-course-images")
output.mkdir(parents=True, exist_ok=True)

pdf = pymupdf.open(source)
seen = set()
for page_number, page in enumerate(pdf, 1):
    for image_number, image in enumerate(page.get_images(full=True), 1):
        xref = image[0]
        if xref in seen:
            continue
        seen.add(xref)
        data = pdf.extract_image(xref)
        path = output / (
            f"page-{page_number:02d}-image-{image_number:02d}.{data['ext']}"
        )
        path.write_bytes(data["image"])
        print(path, data["width"], data["height"])
PY
```

Adapt `source` to the actual PDF path. Inspect the extracted images visually.

Keep images that materially support the lesson:

- course and architecture diagrams;
- framework comparisons and learning roadmaps;
- project architecture and technology-stack diagrams;
- workflow, monitoring, and evaluation diagrams;
- concrete examples discussed by the text, including résumé screenshots.

Exclude:

- avatars, icons, QR codes, and decorative fragments;
- comments, chat reactions, and promotional screenshots;
- duplicated images;
- images containing private information that is not needed for the lesson.

Do not discard a screenshot merely because it is not a diagram. If the text discusses a résumé, project case, output, or interface, the corresponding screenshot is instructional evidence and should normally remain.

### 4. Select stable asset names

Copy selected images into the lesson's image directory. Use ordered numeric names for the main teaching sequence and descriptive names for supporting examples:

```text
images/lesson0/
├── 0.png
├── 1.png
├── 2.png
└── resume-example.png
```

Do not renumber existing assets unnecessarily. Both `docs/` and `origins/` reference the same assets with relative paths:

```markdown
![Agent 学习路线](../images/lesson0/2.png)
```

Write meaningful alt text describing the image's role.

### 5. Write the polished lesson (`docs/`)

Use the source language unless the user requests translation. For a polished rewrite in `docs/lessonN.md`:

1. Start with the lesson title and a short orientation.
2. Use `##` and `###` headings to expose the learning structure.
3. Convert dense prose into short paragraphs and focused lists.
4. Define each framework or component by its role in the overall system.
5. Explain transitions, such as why a linear chain leads to graph orchestration.
6. Place each image immediately after the paragraph that introduces it.
7. Introduce concrete examples before displaying them.
8. End with a concise learning sequence and expected outcome.

For an example image, include a brief interpretation rather than inserting it without context:

```markdown
下面是一份工业知识库项目简历示例。它通过项目背景说明业务痛点，
列出核心技术栈，并从数据处理、检索和问答生成等方面描述个人工作。

![工业知识库项目简历示例](../images/lesson0/resume-example.png)
```

Do not invent details absent from the PDF. Correct obvious extraction artifacts and typography, but preserve technical meaning.

### 6. Write the original document (`origins/`)

Always also produce `origins/lessonN.md` as a faithful source document of the PDF body.

Rules for `origins/`:

1. Keep the author's original wording, order, and conversational tone as closely as possible.
2. Use light Markdown only: title, blank-line paragraphs, and the same teaching images used by `docs/`.
3. Do not restructure into a polished teaching outline; that belongs in `docs/`.
4. Fix PDF extraction artifacts (broken CJK compatibility characters, accidental line breaks mid-sentence) so the text is readable.
5. Remove the same publishing noise as in step 2 (paywalls, comments, private contacts, navigation chrome).
6. Keep instructional screenshots and diagrams; omit video-only placeholders that have no recoverable content.
7. Mark missing video content briefly when the PDF only has a player stub, for example: `（原课程视频：……）`.
8. Use the same `../images/lessonN/...` paths as `docs/` so assets stay shared.

Naming: keep `docs/lessonN.md` and `origins/lessonN.md` in sync for the same lesson number.

### 7. Check surrounding navigation

If the lesson title or purpose changed, inspect README files and lesson indexes for stale labels. Update them only when they are part of the requested scope or necessary to prevent misleading navigation. Prefer linking the polished `docs/` lesson from the index; `origins/` is the source archive.

### 8. Verify

Before reporting completion:

1. Confirm every Markdown image reference in `docs/` and `origins/` resolves to a non-empty file.
2. Confirm every copied image is referenced by at least one of the two documents, unless intentionally retained for later use.
3. Check heading order, list rendering, alt text, and relative paths.
4. Run available Markdown linting.
5. Review against the PDF outline:
   - no major teaching stage is missing from `docs/`;
   - `origins/` preserves original wording without polished rewrite;
   - no useful project or résumé example was dropped;
   - publishing noise and private details were removed;
   - the `docs/` summary matches the body.
6. Inspect the final diff and avoid modifying unrelated files.

Report the updated `docs/` path, `origins/` path, number of retained images, notable omissions, and verification performed.
