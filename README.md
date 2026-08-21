# NSE Volume Tracker

Automatically fetch the latest available NSE Equity Bhavcopy, identify the top 250 stocks by traded volume, and update a Google Sheet.

## Features

- Downloads NSE cash-market Bhavcopy ZIP files
- Supports current UDiFF and older Bhavcopy column naming formats
- Includes only `EQ` (equity) series stocks
- Excludes ETF and commodity-related symbols such as BEES, GOLD, SILVER, LIQUID, and ETF
- Ranks stocks by total traded volume
- Writes the top 250 stocks to Google Sheets
- Adds the source data date and last update time in IST
- Searches up to the previous 5 calendar days, helping handle weekends and non-trading days

## Data Output

The script updates the `Top 250 Stocks` worksheet with:

| Column | Data |
|---|---|
| A | Stock symbol |
| B | Total traded volume |
| C | Closing price |

It also writes an update status in cell `K2`.

## Requirements

- Python 3.9 or later
- A Google Cloud service account
- Google Sheets API enabled
- Google Drive API enabled
- A Google Sheet shared with the service-account email address

Install dependencies:

```bash
pip install gspread oauth2client pandas requests
```

## Google Cloud Setup

1. Create a project in Google Cloud Console.
2. Enable:
   - Google Sheets API
   - Google Drive API
3. Create a service account.
4. Create and download its JSON key.
5. Share your Google Sheet with the service account email address, giving it **Editor** access.
6. Store the complete JSON credentials as an environment variable named `GCP_CREDENTIALS`.

Example:

```bash
export GCP_CREDENTIALS='{"type":"service_account","project_id":"your-project-id", ... }'
```

> Never upload your service-account JSON key to GitHub.

## Configuration

In `update_sheet.py`, update these values if needed:

```python
spreadsheet_id = "YOUR_GOOGLE_SHEET_ID"
worksheet = client.open_by_key(spreadsheet_id).worksheet("Top 250 Stocks")
```

The spreadsheet ID is the part between `/d/` and `/edit` in a Google Sheets URL:

```text
https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit
```

## Run Locally

Set your Google credentials environment variable, then run:

```bash
python update_sheet.py
```

On success, the script prints:

```text
SUCCESS: Sheet Updated!
```

## Automation with GitHub Actions

To run this automatically, add the following GitHub Actions secrets:

- `GCP_CREDENTIALS` — full service-account JSON as a single-line string
- `SPREADSHEET_ID` — optional, if you move the sheet ID into an environment variable

Example schedule:

```yaml
name: Update NSE Volume Sheet

on:
  schedule:
    - cron: "30 11 * * 1-5" # Weekdays, 5:00 PM IST
  workflow_dispatch:

jobs:
  update-sheet:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: pip install gspread oauth2client pandas requests

      - name: Update Google Sheet
        env:
          GCP_CREDENTIALS: ${{ secrets.GCP_CREDENTIALS }}
        run: python update_sheet.py
```

## Notes

- Bhavcopy data is end-of-day data, not live market data.
- The script ranks stocks by traded quantity/volume, not traded value.
- NSE files may be published after market close, so schedule the workflow after the Bhavcopy becomes available.
- This project is for educational and informational use only; it is not investment advice.

## License

MIT License
