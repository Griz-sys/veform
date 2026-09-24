# Delivery Receipt Filler

A static, client-side web app that turns an American Iron Works **customer report**
(`.xls` / `.xlsx` / `.csv`) into a filled **AIW Delivery Receipt** PDF.

Everything runs in the browser — the uploaded file never leaves the visitor's machine.
**No backend, no build step, no dependencies to install.**

## Files

| File | Purpose |
|------|---------|
| `index.html` | The whole app (UI + parsing + PDF generation). |
| `template.pdf` | Blank AIW Delivery Receipt form (AcroForm layer stripped) used as the background. |
| `favicon.svg` | Site icon. |
| `vercel.json` | Static hosting config (headers, clean URLs). |

The app loads two libraries from cdnjs at runtime: **SheetJS** (reads Excel) and
**pdf-lib** (stamps the PDF). An internet connection is required for those to load.

## How it maps data

- Header block → `PROJECT NAME`, `CONTRACTOR`, `ADDRESS`, `CITY / STATE`, `JOB NUMBER`
- Each line item (`units · description · mark#`) → one grid row, **23 rows per page**,
  adding pages automatically.
- The `%`, `D/E`, and `Units Shipped` columns are intentionally left blank for hand entry.

If the source column layout changes, the placement constants live near the top of the
`<script>` in `index.html` (`HDR`, `ROW_LINES`, `COL`).

## Deploy to Vercel

### Option A — Git (recommended)
1. Put these files in their own folder and initialize a repo:
   ```bash
   git init && git add . && git commit -m "Delivery Receipt Filler"
   ```
2. Push to GitHub/GitLab/Bitbucket.
3. On [vercel.com](https://vercel.com) → **Add New → Project** → import the repo.
   Framework preset: **Other**. Build command: **none**. Output dir: **/** (root).
4. Deploy. You get a `*.vercel.app` URL; add a custom domain under **Settings → Domains**.

### Option B — Vercel CLI (no git needed)
```bash
npm i -g vercel
vercel        # preview deploy, follow the prompts
vercel --prod # production deploy
```

## Local preview

```bash
# any static server works; python is fine
python -m http.server 8000
# then open http://localhost:8000
```

> Open via a server, not `file://` — the app `fetch()`es `template.pdf`, which
> browsers block on the `file://` origin.

## Swapping the blank template

Replace `template.pdf` with another single-page US-Letter (612×792 pt) AIW form.
If it is a fillable PDF, strip the form layer first so leftover field values don't
show through:

```python
from pypdf import PdfReader, PdfWriter
r = PdfReader("original.pdf"); w = PdfWriter(); w.append(r)
for p in w.pages:
    if "/Annots" in p: del p["/Annots"]
if "/AcroForm" in w._root_object: del w._root_object["/AcroForm"]
w.write("template.pdf")
```

Then re-check the placement constants in `index.html` against the new grid.
