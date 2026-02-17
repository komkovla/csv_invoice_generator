# Invoice Generator

A command-line tool that reads a CSV file and generates individual PDF invoices for each row. Built with Python, Jinja2 templating, and WeasyPrint for HTML-to-PDF conversion.

## UI

UI is built using `customtkinter` 🎨

<img width="632" height="660" alt="Screenshot 2026-02-16 at 18 55 11" src="https://github.com/user-attachments/assets/cc7be8b4-4e69-4bc7-bdb2-6bf1b74904ab" />

---

## Stack

| Component        | Technology    | Why                                                        |
| ---------------- | ------------- | ---------------------------------------------------------- |
| Language          | Python 3.10+  | Built-in CSV, rich PDF ecosystem, clean scripting          |
| HTML templating   | Jinja2        | Industry-standard, powerful template inheritance           |
| HTML → PDF       | WeasyPrint    | Pure Python, no external binaries (unlike wkhtmltopdf)     |
| CLI interface     | argparse      | Standard library, no extra dependency for a simple CLI     |
| Data validation   | pydantic      | Strict schema enforcement for CSV rows and seller info     |
| Project tooling   | pip + venv    | Simple, universal Python packaging                         |

## Project Structure

```
talgat_invoice_generator/
├── README.md
├── requirements.txt
├── sample.csv                  # Example input file
├── templates/
│   └── invoice.html            # Jinja2 HTML template
├── output/                     # Generated PDFs (gitignored)
├── src/
│   ├── __init__.py
│   ├── main.py                 # CLI entry point
│   ├── models.py               # Pydantic data models
│   ├── csv_parser.py           # CSV reading and validation
│   ├── renderer.py             # Jinja2 HTML rendering
│   └── pdf_generator.py        # WeasyPrint PDF conversion
└── tests/
    ├── __init__.py
    ├── test_csv_parser.py
    ├── test_renderer.py
    └── test_pdf_generator.py
```

## Data Fields

### Per-Invoice (from CSV)

| Field            | CSV Column       | Description                          |
| ---------------- | ---------------- | ------------------------------------ |
| Transaction ID   | `transaction_id` | Unique identifier for the invoice    |
| Customer Name    | `customer_name`  | Name of the customer being invoiced  |
| Date             | `date`           | Invoice / transaction date           |
| Purchase Item    | `item`           | Description of the purchased item    |
| Amount           | `amount`         | Total amount (numeric)               |

### Seller Info (CLI parameters, same on every invoice)

| Field               | CLI Flag     | Description                       |
| ------------------- | ------------ | --------------------------------- |
| Full Name           | `--name`     | Seller's full name                |
| Address             | `--address`  | Seller's address                  |
| IČO (ID number)     | `--ico`      | Seller's identification number    |

## Features

- **CSV Validation**: Automatic validation of invoice data with clear error messages
- **Custom Templates**: Use your own HTML templates with `--template` option
- **Progress Indicators**: Real-time progress showing "Generated X/Y invoices"
- **Robust Error Handling**: Continues processing even if individual invoices fail
- **Professional Output**: Clean, print-ready PDF invoices with modern styling

## CLI Interface

**Basic Usage:**
```bash
python -m src.main \
  --csv sample.csv \
  --name "Talgat Doe" \
  --address "123 Main St, Prague" \
  --ico "12345678" \
  --output ./output
```

**With Custom Template:**
```bash
python -m src.main \
  --csv invoices.csv \
  --name "John Doe" \
  --address "123 St" \
  --ico "12345" \
  --template custom_template.html \
  --output ./output
```

### Arguments

| Argument        | Required | Default      | Description                        |
| --------------- | -------- | ------------ | ---------------------------------- |
| `--csv`         | Yes      | —            | Path to the input CSV file         |
| `--name`        | Yes      | —            | Seller full name                   |
| `--address`     | Yes      | —            | Seller address                     |
| `--ico`         | Yes      | —            | Seller IČO (identification number) |
| `--output`      | No       | `./output`   | Directory for generated PDFs       |
| `--template`    | No       | built-in     | Path to custom HTML template       |

### Output

Each invoice is saved as a separate PDF file named:

```
<transaction_id>.pdf
```

Example: `TXN-2026-001.pdf`

### Error Handling

The tool includes robust error handling:

- **Per-Invoice Errors**: If one invoice fails to generate, processing continues for remaining invoices
- **Error Summary**: At the end, a summary shows successful and failed invoices
- **Clear Messages**: Error messages include context (which invoice, what went wrong)
- **Exit Codes**: Returns 0 on success, 1 if any invoices failed

**Example Error Output:**
```
Generating PDF invoices... (3 total)
Generated TXN-2026-001.pdf (1/3)
Error generating TXN-2026-002.pdf: Template syntax error
Generated TXN-2026-003.pdf (3/3)

Generated 2 PDF invoices successfully
1 invoice(s) failed:
  TXN-2026-002.pdf: Template syntax error
```

## CSV Format

```csv
transaction_id,customer_name,date,item,amount
TXN-2026-001,Acme Corp,2026-01-15,Consulting Services,1500.00
TXN-2026-002,Globex Inc,2026-01-20,Software License,3200.00
TXN-2026-003,Initech Ltd,2026-02-01,Training Workshop,800.00
TXN-2026-004,John Smith,2026-02-10,Website Development,5500.00
TXN-2026-005,Tech Solutions s.r.o.,2026-02-28,Monthly Maintenance,1200.00
TXN-2026-006,StartupXYZ,2026-03-05,Product Design Consultation,2500.00
```

See `sample.csv` for a complete example with realistic invoice data.

## Installation

```bash
# Clone the repository
git clone <repo-url>
cd talgat_invoice_generator

# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

## Releases and code signing

macOS releases are built and published automatically when you push a version tag (e.g. `v1.0.0`). The [release workflow](.github/workflows/release.yml) runs on `macos-latest`, builds the `.app` with PyInstaller, **code-signs** the app and DMG, **notarizes** the DMG with Apple, then uploads it to the GitHub Release. Signing uses the **Prod** environment so that secrets are scoped to production releases.

### Required secrets (Prod environment)

All of the following must be set as **GitHub Actions secrets** for the **Prod** environment (Settings → Environments → Prod → Environment secrets). The workflow specifies `environment: Prod`, so only these environment secrets are used.

| Secret | Description | How to obtain |
|--------|-------------|----------------|
| `APPLE_CERTIFICATE_BASE64` | Developer ID Application certificate and private key, base64-encoded | Export the certificate + key from Keychain Access as a `.p12` file, then run: `base64 -i YourCertificate.p12 | pbcopy` (paste into the secret value). |
| `APPLE_CERTIFICATE_PASSWORD` | Password used when exporting the `.p12` file | The password you set in Keychain Access when exporting the certificate. |
| `APPLE_TEAM_ID` | Apple Developer Team ID (10 characters) | [Apple Developer](https://developer.apple.com/account) → Membership details, or from the certificate name in Keychain. |
| `APPLE_ID` | Apple ID email used for notarization | The Apple ID associated with your developer account. |
| `APPLE_APP_PASSWORD` | App-specific password for notarization | [appleid.apple.com](https://appleid.apple.com) → Sign-In and Security → App-Specific Passwords → Generate. Use this instead of your normal Apple ID password. |

Without these secrets, the workflow will fail at the “Import code signing certificate” or “Notarize .dmg” steps. Repository-level secrets are not used for this job; the job runs in the Prod environment only.

### Getting a Developer ID certificate and `.p12`

1. In [Certificates, Identifiers & Profiles](https://developer.apple.com/account/resources/certificates/list), create a **Developer ID Application** certificate (you may need to create a CSR from Keychain Access on your Mac first).
2. Download the `.cer` file and double-click it to install into Keychain. It will pair with the private key created when you made the CSR.
3. In Keychain Access, find “Developer ID Application: …”, right-click → **Export**, save as `.p12` with a password, and use that file (and password) for `APPLE_CERTIFICATE_BASE64` and `APPLE_CERTIFICATE_PASSWORD` as above.

### Release flow

1. Tag and push: `git tag v1.0.0 && git push origin v1.0.0`
2. The release workflow runs (and may wait for Prod environment approval if you have protection rules).
3. Build → sign app and DMG → notarize with Apple → staple ticket → upload DMG to the GitHub Release.

The published DMG is signed and notarized so users can open it without Gatekeeper warnings.

## License

MIT
