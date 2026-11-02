# Python Development: Data Processing Project

A command-line application that reads structured order data (CSV or JSON), validates every record against business rules, summarizes the valid data with meaningful statistics, and exports clear reports. Invalid records are never silently dropped. They are logged and written to a dedicated file with the exact reasons they failed.

Built with the **Python standard library only**, so there is nothing to install beyond Python itself.

## Features

- **Secure, defensive input reading**: CSV and JSON, UTF-8 (including Excel's BOM), header checks, and clear errors for missing, empty, or malformed files.
- **Rule-based validation**: required fields, type checks (integer quantity, numeric price, ISO date), range bounds, allowed values, ID format, and duplicate detection. *All* problems in a row are reported at once.
- **Graceful error handling**: one bad record never stops the run; file-level problems exit with a clean message and a non-zero status code.
- **Meaningful statistics**: validity rate, total / average / median revenue, largest and smallest order, and breakdowns by category, region, and month, plus top products and rejection counts per field.
- **Multiple outputs**: human-readable text report, machine-readable JSON, cleaned CSV, and a rejected-records CSV.
- **Logging**: progress to the terminal and a full log in `logs/processing.log`.
- **Unit tests** for validation and summarizing logic.

## Project Structure

```
data_processing_project/
├── main.py                  # CLI entry point
├── data_processor/
│   ├── __init__.py
│   ├── reader.py            # CSV / JSON loading
│   ├── validator.py         # business rules + record validation
│   ├── summarizer.py        # statistics and grouping
│   └── exporter.py          # report writers
├── data/
│   └── input.csv            # sample data (valid + invalid rows)
├── tests/
│   └── test_validator.py
├── output/                  # generated reports (created on run)
├── logs/                    # generated log file (created on run)
├── requirements.txt
├── .gitignore
└── README.md
```

## Installation

Requires **Python 3.9 or newer**.

```bash
cd data_processing_project
python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt  # no third-party packages needed
```

## Usage

```bash
# Run with defaults (data/input.csv -> output/)
python main.py

# Custom input / output
python main.py --input data/input.csv --output output

# JSON input also works
python main.py --input my_orders.json

# Verbose (debug) logging
python main.py -v
```

| Option | Description | Default |
|---|---|---|
| `-i`, `--input` | Input `.csv` or `.json` file | `data/input.csv` |
| `-o`, `--output` | Output directory | `output` |
| `-l`, `--log-dir` | Log directory | `logs` |
| `-v`, `--verbose` | Debug logging | off |

### Expected input columns

`order_id, customer_name, product, category, quantity, unit_price, order_date, region`

### Validation rules

| Field | Rule |
|---|---|
| `order_id` | Required, format `ORD-0000`, unique |
| `customer_name` | Required, at most 60 characters |
| `product` | Required |
| `category` | One of Electronics, Books, Clothing, Home, Sports |
| `region` | One of North, South, East, West |
| `quantity` | Whole number from 1 to 1000 |
| `unit_price` | Number from 0.01 to 100,000 |
| `order_date` | `YYYY-MM-DD`, between 2020-01-01 and today |

Rules live at the top of `data_processor/validator.py` and are easy to change.

## Outputs

| File | Contents |
|---|---|
| `output/report.txt` | Formatted summary report |
| `output/summary.json` | Same statistics, machine-readable |
| `output/valid_records.csv` | Clean records plus a computed `order_total` |
| `output/rejected_records.csv` | Invalid rows with line number and reasons |
| `logs/processing.log` | Full run log |

With the bundled sample data: 32 records, 20 valid, 12 rejected (62.5% valid).

## Running the Tests

```bash
python -m unittest discover -s tests -v
```

## Exit Codes

`0` success, `1` input/output problem (e.g. file not found), `2` unexpected error.
