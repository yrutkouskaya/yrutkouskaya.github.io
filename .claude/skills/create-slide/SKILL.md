---
name: create-slide
description: Use when turning uploaded photos (of slides, book pages, tables, whiteboards, screens) into a slide page on this site. Cleans the photos, transcribes the slide text as real HTML, and builds the slide as markup with the pictures as img files.
---

# Create a slide from photos

A slide here is **HTML markup plus image files**, not one big picture and not an SVG:

- Slide text (title, body, table data, descriptions, captions, sources) is **real HTML text**, so the site search finds it.
- Photos and graphics are **PNG or JPEG files** referenced with `<img>`. Text that is part of a picture stays in the picture.
- Every slide has a **16:9 canvas**, whatever the ratio of the source photo. Fit the content into 16:9 and keep its order and proportions (see section 6).
- Slides are grouped in folders. A new slide goes into an **existing folder or a new folder, never into the repo root**, and every file that belongs to it lives inside that folder (see "Where slides live").
- Nothing is committed until the user approves (see `new-slide-flow`). Binary files are pushed with the API route in "Pushing", not the GitHub connector.

## Where slides live

```
<folder>/
  index.md                    (parent page of the folder, title = sidebar label)
  <slide>.md                  (one page per slide, parent: <folder title>)
  images/<slide>/<name>.png   (all picture files of that slide)
```

- Pick the folder with the user: use an existing folder, or create a new one (lowercase, hyphens). A new folder needs its own `index.md` with front matter (`title`, `nav_order`), as in `add-page`.
- Never create a slide `.md` in the repo root. Never put slide images in a shared top-level folder.
- Slides do not reuse files from other folders. The only shared file is `_includes/slide-style.html` (common styling).
- Moving a slide means moving its page and its images together, then fixing the `parent`, `nav_order` and `img src` paths.

## 1. Look before touching

1. View every uploaded photo. Decide for each: *content photo* (a picture that goes on the slide, e.g. a clinical photo or diagram) or *source photo* (a photo of a slide, page or table that you read the text from).
2. A source photo gives you text. It is **not** published as it is: no table, hands, desk or neighbouring objects in the result.
3. If the photo is too blurry, dark or glared to read reliably, ask for a re-shoot instead of guessing. Say what is wrong ("the dose line is cut off by glare").
4. If the photo cuts off part of the slide (edges of the frame, a hand, a reflection), **ask the user** whether to complete the missing parts. Plain decoration (a shape, a frame, a background) can be completed when the user says yes. Missing text or pictures cannot be completed: ask for a better photo.

## 2. Privacy check (this is a medical knowledge base)

Before anything else, check every photo for: a recognisable face, a name, date of birth, patient or case number, hospital or clinic labels, barcodes, or a visible screen with other patients.

- If found, **stop and ask** the user how to handle it (crop out, blur, or confirm consent). Do not decide alone.
- Never publish more of a face than the slide needs.
- The final image files must have **no EXIF** (GPS, device, timestamps). Re-save through Pillow or OpenCV without copying metadata, then check with `identify -verbose` or `PIL.Image.getexif()`.

## 3. Fix the photo (bad angle, garbage, bad light)

Work on a copy. Keep the original untouched in the sandbox. **Do not commit originals** unless the user asks.

**Order matters:**

1. **Orientation.** Apply EXIF rotation first (`PIL.ImageOps.exif_transpose`). OpenCV ignores it.
2. **Find the content area.** The page, slide, screen or table you want, not the whole frame. Estimate the 4 corners by looking at the image. Show the user the corners if you are unsure.
3. **Perspective correction.** Warp the 4 corners to a rectangle (`cv2.getPerspectiveTransform` plus `warpPerspective`, `INTER_CUBIC`).
   - Force the real aspect ratio when you know it: 16:9 for a presentation slide, about 1:1.414 for an A4 page. Otherwise take it from the longer of each pair of edges.
   - Put the corners 2 to 4 px **inside** the edge, or crop 1 to 2 % afterwards, so the border line and background do not leak in.
   - Curved book pages cannot be flattened perfectly. Crop tighter to the flatter part and accept a little curve instead of stretching the text.
4. **Crop the garbage.** Remove table, hands, fingers, bezels, other slides, UI chrome, reflections at the edges. Cropping is the default tool.
5. **Light.** For a photo of a screen, take the white balance from a known light area of the slide (for example the background): measure its mean colour and scale each channel to a neutral light grey. Take any colours you reuse in CSS from the corrected photo, not the raw one. Then fix exposure, lift shadows, even out a gradient (divide by a heavily blurred copy for paper or whiteboards), then a gentle contrast curve. For screens with moiré, downscale first, then blur very slightly.
6. **Sharpen last**, lightly (unsharp mask, radius about 1 px, amount about 0.5). Never sharpen before resizing.

**Fidelity rules (medical content):**

- **Never retouch the content.** No inpainting, cloning, "beautifying" or generative fill inside the picture. Do not change colours of skin, lesions, markings, injection points, scales or labels beyond neutral white balance.
- Removing garbage means **cropping or masking the background**, not drawing over content. If garbage overlaps the content (a finger over a dose), ask for a better photo.
- Do not crop away anything that carries meaning: scale bars, orientation marks (L/R), legends, source credits, figure numbers.
- If the user wants a photo anonymised, blur or crop and say exactly what you did.

**Tooling in the sandbox:** Pillow, OpenCV, numpy, scikit-image, tesseract and Playwright (headless Chromium) are installed. Tested snippet for step 3:

```python
import cv2, numpy as np

def order(pts):                      # TL, TR, BR, BL
    pts = np.array(pts, "float32"); s = pts.sum(1); d = np.diff(pts, axis=1).ravel()
    return np.array([pts[s.argmin()], pts[d.argmin()], pts[s.argmax()], pts[d.argmax()]], "float32")

def rectify(src, dst, corners, aspect=None, long_side=2000):
    img = cv2.imread(src); q = order(corners)
    w = max(np.linalg.norm(q[1]-q[0]), np.linalg.norm(q[2]-q[3]))
    h = max(np.linalg.norm(q[3]-q[0]), np.linalg.norm(q[2]-q[1]))
    if aspect: h = w / aspect
    k = long_side / max(w, h); W, H = int(w*k), int(h*k)
    M = cv2.getPerspectiveTransform(q, np.array([[0,0],[W-1,0],[W-1,H-1],[0,H-1]], "float32"))
    cv2.imwrite(dst, cv2.warpPerspective(img, M, (W, H), flags=cv2.INTER_CUBIC))
```

## 4. Export the images

| Kind | Format | Notes |
|---|---|---|
| Photo (skin, anatomy, real-world) | **JPEG**, quality 82 to 88, progressive, sRGB | no alpha needed |
| Graphic, diagram, screenshot, flat colours, sharp lines | **PNG** (optimised) | use 8-bit palette if it stays clean |

- **Size:** width = 2x the on-slide width, at most 1600 px wide for a half-slide picture and 2000 px for a full-width one. Never upscale a small photo to look bigger. Say so if the source is small.
- **Weight:** aim for under 300 KB per file, hard limit 500 KB. Lower quality or size before accepting more.
- **Location:** `<folder>/images/<slide>/<short-name>.jpg` (or `.png`), inside the slide's own folder (see "Where slides live"). It must be outside `.claude/`, which `_config.yml` excludes from the site.
- **Names:** lowercase, hyphens, descriptive (`injection-points.jpg`, not `IMG_4471.jpg`).
- Always set `width` and `height` attributes on `<img>` (the real pixel size) and `loading="lazy"`.

## 5. Text: what becomes markup, what stays in the picture

**Test for each piece of text:** *Would someone search for this to find the slide?* or *Is it the slide's own wording?* Then it is real text. *Is it a label that only makes sense on top of the picture?* Then it stays in the picture.

| Becomes HTML text | Stays in the image |
|---|---|
| Slide title and subtitles | Title or caption burned into a photo or figure |
| Body text, bullets, steps | Arrow labels, anatomy labels, legends |
| **Table data** (as a real `<table>`) | Axis labels, tick values and series names of a chart |
| Descriptions, captions, footnotes | Logos, watermarks, handwriting on the photo |
| Doses, values, units, thresholds | Scale bars, L/R marks |
| Sources and references | Text on packaging or devices in a photo |

- If a chart's **numbers** are the point of the slide, also give them as a table or a one-line statement in the text. If the picture contains a key message in a long label, put the gist in the caption.
- Alt text is not searchable here, so do not use it to hide slide text. Alt text says what the picture shows, in one sentence, plus the key in-picture labels if they matter.

**Transcription rules:**

- Follow the slide text exactly as it is on the source: wording, spelling, capitals, hyphenation, abbreviations, units and the user's language (`U`, `mm`, `%`, ranges with an en dash). Do not correct, normalise or rewrite it, and do not ask about it. Only ask when something is unclear or missing (covered, cut off, glare).
- Read **every number, dose and unit twice** against the photo. Never guess.
- Anything unclear or missing: stop and ask the user before building. Do not guess and do not fill gaps from memory or medical knowledge.
- Do not add content the source does not have. No extra bullets, no "helpful" context.
- Tables: keep the same rows, columns and order. Merged cells: repeat or use `colspan`.
- Order by reading order of the source, not by where text sits on the page.

## 6. Build the page

File: `<folder>/<slide>.md`, never in the repo root. Front matter as in `add-page`: `title`, `parent` (the folder's `index.md` title) and `nav_order` among the slides of that folder.

**Shared styles.** If `_includes/slide-style.html` does not exist, create it first (CSS below). It is kept in one place so all slides look the same. Sizes use `cqw` (1% of slide width) so the whole slide scales like the old SVG slides. Convert from the 1920 px design size: `cqw = px / 19.2`.

```html
<style>
.slide{container-type:inline-size;width:100%;margin:0 0 1.5rem}
.slide *{box-sizing:border-box}
.slide__canvas{aspect-ratio:16/9;overflow:hidden;background:#fff;color:#111;border:1px solid #e6e6ec;padding:5.6cqw 6.25cqw;display:grid;grid-template-columns:minmax(0,1fr) 35.4cqw;column-gap:4cqw;align-content:start;font-family:'Helvetica Neue',Helvetica,Arial,'Liberation Sans',sans-serif;line-height:1.3}
.slide__head{grid-column:1 / -1;margin-bottom:3cqw}
.slide__eyebrow{margin:0 0 1.2cqw;font-size:1.15cqw;font-weight:700;letter-spacing:.23em;text-transform:uppercase;color:#2f4f4f}
.slide h1.slide__title{margin:0;padding:0;font-size:3.96cqw;font-weight:700;line-height:1.1;color:#111}
.slide__label{margin:0 0 1.2cqw;font-size:1.15cqw;font-weight:700;letter-spacing:.18em;text-transform:uppercase;color:#333}
.slide ul.slide__list{margin:0;padding:0;list-style:none}
.slide ul.slide__list li{position:relative;margin:0 0 1.4cqw;padding:0 0 0 2.6cqw;font-size:1.77cqw;line-height:1.35;color:#1a1a1a}
.slide ul.slide__list li::before{content:"";position:absolute;left:.4cqw;top:.75cqw;width:.45cqw;height:.45cqw;border-radius:50%;background:#1a1a1a}
.slide figure.slide__figure{margin:0}
.slide figure.slide__figure img{display:block;width:100%;height:auto;border-radius:0}
.slide figcaption{margin-top:.8cqw;font-size:1.1cqw;color:#555}
.slide table.slide__table{display:table;width:100%;border-collapse:collapse;font-size:1.5cqw}
.slide .slide__table th,.slide .slide__table td{padding:.7cqw 1cqw;border:0;border-bottom:1px solid #e6e6ec;text-align:left;vertical-align:top}
.slide .slide__table th{background:#f4f4f7;font-weight:700}
</style>
```

The rules that start with `.slide h1`, `.slide ul`, `.slide table` and so on deliberately reset the Just the Docs defaults (heading size, list padding, block tables). Keep them.

**Page body:**

```html
{% include slide-style.html %}
<div markdown="0">
<section class="slide" aria-label="Slide title">
<div class="slide__canvas">
<header class="slide__head">
<p class="slide__eyebrow">Optional eyebrow</p>
<h1 class="slide__title">Slide title</h1>
</header>
<div class="slide__text">
<p class="slide__label">Section label</p>
<ul class="slide__list">
<li>First point, as written on the source</li>
</ul>
</div>
<figure class="slide__figure">
<img src="/<folder>/images/<slide>/<name>.jpg" width="680" height="564" alt="One sentence on what the picture shows" loading="lazy">
</figure>
</div>
</section>
</div>
```

- `markdown="0"` keeps kramdown from touching the HTML. No blank lines inside the block.
- If the source slide has no title (or eyebrow), leave out the `<header>` and the `<h1>`. Do not invent one. The `aria-label` carries the name, and the page title is the sidebar label.
- For a table slide, replace the list with `<table class="slide__table">` (`<thead>` and `<tbody>`, `<th scope="col">`). For a picture-only slide, drop the text column and set the figure to span both columns.
- Slide text containing `{{` or `{%` must be wrapped in `{% raw %}...{% endraw %}`, or Liquid will eat it.
- The image CSS uses `width:100%; height:auto`, so the picture keeps its own proportions and the layout never crops it. Do not add `object-fit: cover`.
- The slide must be a **100 % copy** of the source slide: every piece of slide text, every table row and cell, every picture, in the same order and structure. Never drop, shorten, merge, or split content to make it fit. If it does not fit the layout, change the layout, not the content.
- Only the slide itself goes in this skill. The description that follows the slide (step 3 of `new-slide-flow`) is plain Markdown below the `</div>`.

### Diagram and infographic slides

When the slide is a drawing (pyramid, flow, boxes, arrows, speech bubbles), rebuild it instead of publishing the photo:

- Plain shapes are CSS: `clip-path: polygon(...)` for triangles and arrows, `border-radius` for bubbles and frames, absolutely positioned inside `.slide__canvas`. Use `cqw` for every position and size, measured on the 1920 x 1080 design (`cqw = px / 19.2`).
- Text inside the drawing is slide text, so it is real HTML (section 5), placed over the shape. DOM order follows the reading order of the source. Use `z-index` for layering, not DOM order.
- Small pictures and icons are cut out of the photo as transparent PNGs (masking the background is allowed, see the fidelity rules). Do not upscale them.
- Background removal: for light icons on a light background, flood-fill from the edges with a small fixed tolerance (about 9 to 12 per channel) works best. `cv2.grabCut` works for dark icons but eats light parts. Then clear leftover border fragments by hand and check the result on a contrasting colour (for example pink) before using it.

```python
def cut(wb, box, tol, dst, clear=()):          # wb = white-balanced BGR image
    x0, y0, x1, y1 = box; c = wb[y0:y1, x0:x1].copy(); h, w = c.shape[:2]
    ff = np.zeros((h+2, w+2), np.uint8); f = c.copy()
    for sx, sy in [(0,0),(w-1,0),(0,h-1),(w-1,h-1),(w//2,h-1),(0,h//2),(w-1,h//2),(w//2,0)]:
        cv2.floodFill(f, ff, (sx, sy), (255,0,255), (tol,)*3, (tol,)*3, flags=4|cv2.FLOODFILL_FIXED_RANGE|(255<<8))
    a = 255 - (ff[1:-1,1:-1] > 0).astype(np.uint8)*255
    for cx0, cy0, cx1, cy1 in clear: a[cy0:cy1, cx0:cx1] = 0
    a = cv2.GaussianBlur(cv2.morphologyEx(a, cv2.MORPH_OPEN, np.ones((2,2), np.uint8)), (0,0), 0.7)
    Image.fromarray(np.dstack([cv2.cvtColor(c, cv2.COLOR_BGR2RGB), a]), "RGBA").save(dst, optimize=True)
```

## 7. Check before showing the user

1. Render the page body in a standalone HTML file with Playwright at about 1280 px wide and view the screenshot. Check: nothing cut off, nothing overlapping, text readable, picture not distorted, canvas is 16:9.
2. Compare the transcription with the source photo line by line, numbers last.
3. Show the user the render plus what you asked about, and what you changed by cropping, masking or completing.
4. The sandbox cannot build Jekyll (no access to rubygems), so theme interaction is only checked after deploy. After the push, open the live page and look at it. A needed fix is a separate small commit, and you should tell the user so.

## 8. Pushing (images included)

Only after the user approves in step 4 of `new-slide-flow`.

- The GitHub connector cannot push binary files. Use the GitHub REST API from the sandbox (`api.github.com` is allowed by the network settings). If the request is blocked, tell the user they can allow the domain in their network settings.
- Ask the user for a **fine-grained token limited to this repo with Contents: read and write**. Use it only as an environment variable for the one command. Never write it to a file, a commit, or memory, and suggest revoking it afterwards.
- Keep **one commit** for the whole slide: create a blob per file (`POST /git/blobs`, base64), read `main` (`GET /git/ref/heads/main`, then its commit and tree), create a tree with `base_tree` (`POST /git/trees`), create the commit (`POST /git/commits`), then fast-forward `main` (`PATCH /git/refs/heads/main`, never `force`). Include the page `.md`, the images, and `_includes/slide-style.html` if it is new.
- Run the secret scan on the text files as `commit-and-push` says. For images, check there is no EXIF and no text with patient data.
- Verify every pushed file: the blob SHA that GitHub returns must equal the local `git hash-object <file>`. Then follow `commit-and-push` "After pushing".
- Commit message: short and imperative, for example "Add lip flip slide".

## Do not

- Do not turn the whole slide back into one image or SVG with the text baked in.
- Do not extract every word from a photo. Only slide-level text goes to markup (section 5).
- Do not edit, clone, or generate anything inside a clinical picture.
- Do not publish photos with identifiable patients, names, or numbers without the user's explicit decision.
- Do not commit original phone photos, or images over 500 KB.
- Do not guess unreadable text, numbers, or doses.
