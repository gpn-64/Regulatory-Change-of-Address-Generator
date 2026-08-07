# Regulatory Change of Address Generator

Bulk-generate "Change of Address" Word documents from a CSV of product/application
records. Built for the situation where a company has moved offices and needs to
notify a regulator (e.g. the FDA) separately for every application on file --
dozens or hundreds of near-identical letters that differ only by an application
number and a product code.

This is a cleaned-up, anonymized version of a small internal tool I originally
wrote to handle this for a set of US regulatory filings. All company names,
addresses, and product data in this repository are fictional.

## How it works

1. `data/sample_applications.csv` lists the records to generate a letter for
   (`Application Number`, `Product Code`).
2. `templates/change_of_address_template.docx` is a Word template containing
   the placeholders `<Application No>` and `<Product Code>`.
3. `src/generate_documents.py` reads the CSV and, for each row, fills the
   template and saves an individual `.docx` into `output/`.

```
data/sample_applications.csv  ---\
                                   >---  generate_documents.py  --->  output/*.docx
templates/*.docx  ----------------/
```

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
```

## Usage

```bash
python src/generate_documents.py \
    --data data/sample_applications.csv \
    --template templates/change_of_address_template.docx \
    --output-dir output
```

Each row produces a file named `<ProductCode>_<ApplicationNumber>_change_of_address.docx`.

Run `python src/generate_documents.py --help` for all options.

## Tests

```bash
pytest
```

## Rebuilding the sample template

`templates/change_of_address_template.docx` is checked in, but it was produced
by `scripts/build_template.py`, which is kept in the repo so the template's
origin is fully reproducible (no hidden metadata, no borrowed corporate
branding):

```bash
python scripts/build_template.py
```

## Notes on the original version

This started as a quick internal script. Cleaning it up for a public showcase
was also a chance to fix a few real issues:

- **Broken input path.** The original script pointed `pandas.read_csv()` at a
  *folder* instead of a CSV file, so it never actually ran end to end.
- **Formatting loss on replacement.** Word frequently splits a single visible
  word across multiple XML runs (e.g. `<`, `Application No`, `>` each get
  their own run). The original code read/wrote `paragraph.text` directly,
  which either missed matches split across runs or collapsed all of a
  paragraph's formatting into a single default run. `generate_documents.py`
  reassembles each paragraph's text first, replaces placeholders, and writes
  the result back into the first run only -- preserving that run's
  formatting instead of discarding it.
- **No validation or error handling.** Missing CSV columns, blank rows, a
  missing template file, or unsafe characters in a product code would all
  either crash uninformatively or silently produce bad output. These are now
  handled explicitly, with a CLI (`argparse`) and logging instead of `print`.
- **Unnecessary dependency.** The original used `pandas` just to read a
  two-column CSV. Swapped for the standard-library `csv` module.

## License

MIT -- see [LICENSE](LICENSE).
