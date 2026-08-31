# Bill Analyzer

Point a camera at a receipt and get back what it was for and what it cost — logged to a
spreadsheet you can actually use.

> Built during a hackathon. A desktop tool that turns a photograph of a bill into a
> categorised, tallied expense record.

---

<!--
SCREENSHOT: add one here. Run the app, scan the sample receipt, and screenshot the
window with the parsed category and total visible. Save it as docs/screenshot.png,
then delete this comment and uncomment the line below.

![Bill Analyzer scanning a receipt](docs/screenshot.png)
-->

## What it does

1. **Capture** — upload an image of a bill, or take one live from the webcam
2. **Read** — extract the text with Tesseract OCR
3. **Classify** — infer the vendor category from keywords in the extracted text
4. **Tally** — locate the total line and parse the amount, handling currency symbols and thousands separators
5. **Log** — append `category, total` to `bill_analysis.csv`

Categories recognised: utility bills, retail receipts, medical bills, dining and
hospitality, transportation and travel.

## How it works

**Text extraction.** OpenCV loads the image and converts BGR→RGB before handing it to
`pytesseract`, which returns the raw text of the receipt.

**Classification.** A keyword dictionary maps trigger words (`electricity`, `pharmacy`,
`uber`, `restaurant`, …) to categories; the first category with a keyword present in the
lowercased text wins. Deliberately simple — on real receipts the vendor name and line
items carry enough signal that a keyword pass gets most of the way there, and it needs
no training data or network call.

**Total parsing.** Only lines containing "total" are considered, which skips the
per-item prices. A regex then pulls numeric values out of those lines, tolerating `$`,
commas and decimals, and sums them.

**Interface.** Tkinter GUI with a thumbnailed preview of the scanned bill and the parsed
result underneath. The CSV is created with headers on first run.

## Running it

Tesseract must be installed on the system, not just the Python binding:

```bash
brew install tesseract          # macOS  (apt-get install tesseract-ocr on Debian/Ubuntu)
pip install opencv-python pytesseract pillow
python final.py
```

A webcam is required for the *Take Bill Photo* path; the upload path works without one.
`receipt-template-with-barcode.jpg` in the repo is a sample receipt to test against.

## Packaging as a standalone executable

```bash
pyinstaller --onefile -w final.py
```

A prebuilt Windows executable is
[available here](https://drive.google.com/file/d/1CKE8r2ekOh3r2L1CE04lnep2arKr1ODb/view?usp=sharing).

## Repository layout

| Path | Contents |
| --- | --- |
| `final.py` | The application — GUI, OCR, classification, CSV logging |
| `Hackathon_1.ipynb` | Notebook the OCR and parsing logic was prototyped in |
| `receipt-template-with-barcode.jpg` | Sample receipt for testing |

## Known limits

- The classifier is keyword-based, so an unrecognised vendor returns `Unknown category`
- Totals are read from any line containing "total", which double-counts on receipts printing both "Subtotal" and "Total"
- OCR accuracy depends on lighting and focus; no deskewing or thresholding is applied before recognition

## Built with

`Python` · `OpenCV` · `Tesseract OCR` · `Tkinter` · `Pillow`
