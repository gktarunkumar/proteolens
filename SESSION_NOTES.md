# ProteoLens — Hackathon Dev Session Notes
**Project:** HTH-SW-04 · PROTEOLENS  
**Date:** 2026-11-15  
**Stack:** Flask · Biopython · Chart.js 4.4.4 · Python 3  
**Working directory:** `C:\Users\rithe\OneDrive\Desktop\proteolens`

---

## 1. What We Built

ProteoLens is a Flask web app that takes one or more protein sequences in FASTA format, computes physicochemical properties using Biopython's `ProtParam`, renders an interactive comparison table and bar charts, detects sequence motifs, and exports a CSV report.

### Feature list (all implemented and tested)

| Feature | Status |
|---|---|
| Paste FASTA text | ✅ |
| Upload .fasta / .fa / .txt file | ✅ |
| Physicochemical property table | ✅ |
| Molecular weight comparison chart | ✅ |
| Amino acid composition charts (per protein) | ✅ |
| Gold highlight on highest AA% bar | ✅ |
| Stable/unstable tag + plain-English line | ✅ |
| Hydrophilic/hydrophobic tag + plain-English line | ✅ |
| Sequence motif detection (5 patterns) | ✅ |
| Secondary structure annotations (Helix/Sheet/Turn - Bonus) | ✅ |
| Apple Human Interface Guidelines (HIG) UI Revamp | ✅ |
| Interactive Motif Sequence Highlighter | ✅ |
| Drag-and-drop file upload area | ✅ |
| Download CSV report (client-side) | ✅ |
| Load example button | ✅ |
| Modern web / accessibility guidance applied | ✅ |
| Chart.js served locally (offline-safe) | ✅ |
| Demo .fasta file for presentation | ✅ |

---

## 2. Files Changed

### `app.py`
- Added `import re` and `flash` to imports
- Added `MOTIFS` list (5 motif patterns with regex + descriptions)
- Added `detect_motifs(sequence)` function
- Added file upload handling via `request.files.get("fasta_file")`
- Added `motifs` key to each result dict

**Key additions:**

```python
from flask import Flask, render_template, request, flash
from Bio.SeqUtils.ProtParam import ProteinAnalysis
import re

MOTIFS = [
    {"name": "N-glycosylation site",    "pattern": r"N[^P][ST]",  "description": "N-X-S/T (X ≠ Pro) — common sugar-attachment site on proteins"},
    {"name": "RGD motif",               "pattern": r"RGD",         "description": "Arg-Gly-Asp — cell-adhesion sequence recognised by integrins"},
    {"name": "Casein kinase II site",   "pattern": r"[ST].{2}E",   "description": "S/T-X-X-E — target site for casein kinase II phosphorylation"},
    {"name": "PKC phosphorylation site","pattern": r"[ST].[RK]",   "description": "S/T-X-R/K — target site for protein kinase C phosphorylation"},
    {"name": "KDEL retention signal",   "pattern": r"KDEL",        "description": "Lys-Asp-Glu-Leu — endoplasmic reticulum retention signal"},
]

def detect_motifs(sequence):
    hits = []
    for motif in MOTIFS:
        positions = [m.start() + 1 for m in re.finditer(motif["pattern"], sequence)]
        if positions:
            hits.append({
                "name": motif["name"],
                "description": motif["description"],
                "positions": positions,
                "count": len(positions)
            })
    return hits
```

```python
# In route — file upload
uploaded_file = request.files.get("fasta_file")
if uploaded_file and uploaded_file.filename:
    try:
        raw_input = uploaded_file.read().decode("utf-8")
    except UnicodeDecodeError:
        errors.append("Could not read the uploaded file — make sure it is a plain text (.fasta / .fa / .txt) file.")
        return render_template("index.html", results=results, errors=errors, raw_input="")
```

---

### `templates/index.html`
Complete rewrite with all features. Key structural additions:

- `<a href="#main-content" class="skip-link">` — keyboard/screen-reader skip link
- `<form enctype="multipart/form-data">` — required for file upload
- `aria-invalid="true"` on textarea when errors present
- `role="alert"` on error section
- `role="status" aria-live="polite"` for screen-reader result announcement
- `scope="col"` on all `<th>` elements
- `<figure>` / `<figcaption>` wrapping each chart canvas
- `role="img"` + `aria-label` on each `<canvas>`
- Motif detection section with per-protein tables
- Download CSV button (green, calls client-side JS)
- Gold bar highlight on AA composition charts
- Plain-English interpretation lines under instability index and GRAVY score

**CSV download (client-side, no extra Flask route):**
```js
const csvData = {{ results | tojson }};

function downloadCSV() {
    const rows = [["Name","Length","Mol Weight","pI","Instability","Aromaticity","GRAVY"]];
    csvData.forEach(r => {
        const s = r.stats;
        rows.push([r.name, s.length, s.molecular_weight, s.isoelectric_point,
                   s.instability_index, s.aromaticity, s.gravy]);
    });
    const csv = rows.map(r => r.join(",")).join("\n");
    const blob = new Blob([csv], {type: "text/csv"});
    const a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = "proteolens_report.csv";
    a.click();
}
```

**Gold bar highlight logic:**
```js
const maxVal = Math.max(...values);
const colors = values.map(v => v === maxVal ? '#f59e0b' : '#7fc8a9');
```

---

### `static/style.css`
New additions:
- `--focus-ring` CSS variable
- `.skip-link` (visually hidden until focused)
- `.sr-only` (screen-reader-only text)
- `:focus-visible` global outline using `--focus-ring`
- `font-size: max(1rem, 1em)` on textarea — prevents iOS Safari zoom
- `textarea[aria-invalid="true"]` — red border on error
- `min-block-size: 44px` on buttons (WCAG touch target)
- `@media (pointer: coarse)` — 48px touch targets on mobile
- `.file-upload-row`, `.file-label`, `.file-name-display` — file upload UI
- `.motif-card`, `.motif-count`, `.positions`, `.motif-desc`, `.no-motifs`
- `.download-bar`, `#download-csv { background: #059669 }`
- `.interp` — small italic interpretation lines
- `figure` reset (no default margin/border)
- `@media (prefers-reduced-motion: reduce)` stub

---

### `static/chart.min.js`
- Downloaded Chart.js 4.4.4 from `https://cdn.jsdelivr.net/npm/chart.js@4.4.4/dist/chart.umd.min.js`
- Served locally (205,749 bytes) — works offline at the demo venue
- Original CDN `cdnjs.cloudflare.com/…/4.4.4/chart.umd.min.js` returned 404 (not on cdnjs at that version)
- Script tag updated to `url_for('static', filename='chart.min.js')`

---

### `demo.fasta` *(new file — for presentation)*
4 proteins chosen for maximum visual and scientific contrast:

```fasta
>Ubiquitin
MQIFVKTLTGKTITLEVEPSDTIENVKAKIQDKEGIPPDQQRLIFAGKQLEDGRTLSDYNIQKESTLHLVLRLRGG

>Histone_H4
SGRGKGGKGLGKGGAKRHRKVLRDNIQGITKPAIRRLARRGGVKRISGLIYEETRGVLKVFLENVIRDAVTYTEHAKRKTVTAMDVVYALKRQGRTLYGFGG

>Lysozyme
KVFGRCELAAAMKRHGLDNYRGYSLGNWVCAAKFESNFNTQATNRNTDGSTDYGILQINSRWWCNDGRTPGSRNLCNIPCSALLSSDITASVNCAKKIVSDGNGMNAWVAWRNRCKGTDVQAWIRGCRL

>Calmodulin
MADQLTEEQIAEFKEAFSLFDKDGDGTITTKELGTVMRSLGQNPTEAELQDMINEVDADGNGTIDFPEFLTMMARKMKDTDSEEEIREAFRVFDKDGNGYISAAELRHVMTNLGEKLTDEEVDEMIREADIDGDGQVNYEEFVQMMTAK
```

**Why these 4:**

| Protein | MW (Da) | pI | Key trait |
|---|---|---|---|
| Ubiquitin | 8,564.74 | 6.56 | Near-neutral, Leu-rich |
| Histone_H4 | 11,236.01 | 11.36 | Most basic (Lys/Arg-rich) |
| Lysozyme | 14,313.00 | 9.32 | Cys-rich, Asn dominant |
| Calmodulin | 16,837.42 | 4.09 | Most acidic (Glu-rich), richest motif profile |

- MW range spans 8.5 → 16.8 kDa — 4 clearly different bar heights
- pI range spans nearly full scale (4.09 → 11.36)
- Calmodulin has the richest motif profile: 1 N-glycosylation + 5 CK2 + 2 PKC sites
- Each protein has a different dominant amino acid (L / G / N / E) → gold bars all differ

---

## 3. Bugs Fixed

| Bug | Cause | Fix |
|---|---|---|
| Chart.js CDN 404 | cdnjs doesn't host 4.4.4 | Downloaded from jsdelivr, serve locally |
| "Chart is not defined" console error | Stale error from before fix | Verified `typeof Chart === "function"`, 3 chart instances active |
| Long FASTA headers truncating chart labels | Full UniProt-style headers overflowed Chart.js | Switched to short names: `>Ubiquitin`, `>Histone_H4`, etc. |
| File upload silently ignored | Missing `enctype="multipart/form-data"` on `<form>` | Added enctype attribute |
| iOS Safari input zoom | Default `font-size` below 16px | `font-size: max(1rem, 1em)` on textarea |

---

## 4. Modern Web Guidance Applied

From Chrome developer accessibility docs:

- **Skip link** — `<a href="#main-content">` at top of page
- **aria-live** — `role="status" aria-live="polite"` announces results to screen readers
- **aria-invalid** — applied to textarea on validation errors
- **focus-visible** — global `:focus-visible` outline, suppressed on mouse click
- **Touch targets** — `min-block-size: 44px` buttons; `@media (pointer: coarse)` bumps to 48px
- **sr-only** — table caption and live region are visually hidden but readable
- **prefers-reduced-motion** — stub in place for future transitions
- **Semantic markup** — `scope="col"` on `<th>`, `<caption>`, `<figure>`/`<figcaption>` for charts, `role="img"` on `<canvas>`

---

## 5. How to Run

```bash
cd C:\Users\rithe\OneDrive\Desktop\proteolens
python app.py
```

Open `http://127.0.0.1:5000` in a browser.

**To use the demo file:**
1. Click **Choose file**
2. Select `demo.fasta` from the project folder
3. Click **Analyze**

**To export results:**
- Click **Download CSV Report** after analyzing — saves `proteolens_report.csv`

---

## 6. Project Structure

```
proteolens/
├── app.py                  # Flask backend + motif detection
├── demo.fasta              # 4-protein demo for presentation
├── SESSION_NOTES.md        # this file
├── templates/
│   └── index.html          # Jinja2 template (full UI)
└── static/
    ├── style.css           # All styles + accessibility
    └── chart.min.js        # Chart.js 4.4.4 (local copy)
```

---

## 7. What to Say to Judges

- **Apple Human Interface Guidelines (HIG) Design**: Professional macOS/iOS aesthetics with frosted glass materials (`backdrop-filter`), segmented controls, tactile button feedback, and high visual contrast.
- **Interactive Motif Map**: Solves the "Motif visualization" bonus requirement by allowing users to click motif badges and interactively light up residues directly on the physical sequence track.
- **Secondary Structure Bonus Feature**: Satisfies the explicit bonus requirement on the competition sheet (computes Alpha-Helix, Beta-Sheet, and Turn percentages).
- **CSV download**: Satisfies the problem statement's explicit output requirement.
- **Motif detection**: Goes beyond ProtParam — adds biological interpretation.
- **Drag-and-Drop File Upload**: Modernized upload zone for rapid FASTA ingestion.
- **Offline Chart.js**: Works without internet at the venue.
- **Accessibility**: Skip links, ARIA live regions, 44px/48px touch targets, and focus states.
- **Plain-English interpretations**: Makes it usable by biologists, not just developers.
