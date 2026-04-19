# GSTR-1 Workings Generator

A browser-based tool to generate GSTR-1 working Excel files from multi-marketplace e-commerce sales data. Upload a single master Excel sheet and download one professionally formatted GSTR-1 workings file per Seller GSTIN — no server required.

---

## Features

- **Zero installation** — runs entirely in the browser as a single HTML file
- **Automatic B2B / B2C classification** — based on whether the Buyer GSTIN is a valid 15-character alphanumeric code
- **Credit Note detection** — any line item with a negative Taxable Value is treated as a Credit Note
- **Excel formula-based totals** — all summary/subtotal/grand-total cells use `SUM()` formulas, not hardcoded values
- **Professional formatting** — styled headers, alternating row colours, borders, bold totals, and proper column widths via ExcelJS
- **Multi-GSTIN support** — generates one `.xlsx` per Seller GSTIN found in the data
- **ZIP download** — download all generated files at once
- **Full GST State mapping** — all 37 states/UTs mapped to their GST Place of Supply codes

---

## How to Use

1. Open `gstr1_workings.html` in any modern browser (Chrome, Edge, Firefox, Safari)
2. Drag and drop your master Excel file (or click to browse)
3. Review the data preview — row counts, B2B/B2C/CN breakdown, detected GSTINs
4. Click **Generate GSTR-1 Workings**
5. Download individual files or all at once as a ZIP

---

## Input File Format

The input must be an Excel file (`.xlsx` or `.xls`) with a sheet named **"Master Sheet"** (falls back to the first sheet if not found).

### Required Columns (case-sensitive)

| Column | Description |
|---|---|
| `SELLER GSTIN` | 15-character GSTIN of the seller |
| `SELLER` | Seller / marketplace name |
| `TYPE` | Original type label (ignored for classification — see below) |
| `BUYER GSTIN` | Buyer's GSTIN — **used for B2B/B2C classification** |
| `INVOICE NO` | Invoice or credit note number |
| `INVOICE DATE` | Date (Excel date object or `DD-Mon-YYYY` string) |
| `PRODUCT DESCRIPTION` | Product name |
| `HSN` | HSN/SAC code |
| `SKU CODE` | SKU identifier |
| `QUANTITY` | Quantity sold (negative for credit notes) |
| `TAXABLE VALUE` | Taxable amount — **negative = credit note** |
| `TAX RATE` | GST rate as integer (0, 5, 12, 18, 28) |
| `IGST` | IGST amount |
| `CGST` | CGST amount |
| `SGST` | SGST amount |
| `BILLING STATE` | Customer's billing state name |

### Classification Rules

| Condition | Classification |
|---|---|
| `BUYER GSTIN` is exactly 15 alphanumeric characters | **B2B** |
| `BUYER GSTIN` is blank or not 15 characters | **B2C** |
| `TAXABLE VALUE` is negative | **Credit Note** (shown in Docs Issued) |

> Credit notes with negative taxable value are netted into the B2C sheet automatically. Quantity for credit notes is always made negative.

---

## Output Format

One `.xlsx` file per Seller GSTIN, named `{GSTIN}_GSTR1_Workings.xlsx`.

Each file contains **5 sheets**:

### Sheet 1 — B2B
- Summary row with formula-based totals for Invoice Value, Taxable Value, IGST, CGST, SGST
- Summary values are positioned directly above their corresponding data column headers
- One row per B2B invoice
- Tax rates shown as integers (5, 12, 18…)

### Sheet 2 — B2C
- B2C sales + credit notes netted together
- Aggregated by Place of Supply and Tax Rate
- Grand Total row uses `SUM()` formulas

### Sheet 3 — HSN
- Two blocks: B2B HSN summary and B2C HSN summary
- Grouped by HSN code and Tax Rate
- Subtotal/Grand Total rows use `SUM()` formulas

### Sheet 4 — Docs Issued
- Invoice and Credit Note number ranges with series detection
- Groups consecutive invoice numbers into ranges automatically
- Total row uses `SUM()` formula

### Sheet 5 — Summary
- GSTIN identification and period (auto-detected from invoice dates)
- B2B and B2C totals
- Total row uses formula-based addition of B2B + B2C values

---

## Tally Verification

The tool ensures internal consistency:

- B2B sheet totals = HSN sheet B2B subtotals = Summary B2B row
- B2C sheet Grand Total = HSN sheet B2C Grand Total = Summary B2C row
- All values rounded to 2 decimal places using `Math.round(value * 100) / 100`

---

## Dependencies (loaded via CDN)

| Library | Purpose | CDN |
|---|---|---|
| [SheetJS](https://sheetjs.com/) | Reading input Excel files | cdnjs.cloudflare.com |
| [ExcelJS](https://github.com/exceljs/exceljs) | Writing formatted output Excel files | cdn.jsdelivr.net |
| [JSZip](https://stuk.github.io/jszip/) | ZIP archive for bulk download | cdnjs.cloudflare.com |

No build tools, frameworks, or server-side code required.

---

## Browser Compatibility

Works in all modern browsers that support ES6+:
- Chrome 60+
- Firefox 55+
- Edge 79+
- Safari 12+

---

## License

MIT

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-change`)
3. Commit your changes (`git commit -am 'Add my change'`)
4. Push to the branch (`git push origin feature/my-change`)
5. Open a Pull Request
