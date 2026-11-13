# Portfolio Challenge — self-updating dashboard (demo)

**All data in this repo is fake. The 20 students, usernames and portfolios are
fabricated for demonstration.**

This repo replaces the manual workflow behind the Portföljtävlingen /
Investment Challenge leaderboard (a static HTML file with hand-pasted numbers)
with a pipeline that updates itself. The dashboard design is unchanged —
only the numbers are computed instead of typed.

## Data flow

| Step | Who | What happens | Tool | Output |
|---|---|---|---|---|
| 1 | Students | Submit one Excel file with up to 5 picks (ticker, buy date, amount € or shares) | itslearning assignment | `.xlsx` files named by username |
| 2 | Teacher | Bulk-download the assignment zip, drag the files into `submissions-xlsx/` via GitHub's web upload | GitHub web UI (no git needed) | Raw submissions in repo |
| 3 | Automation | Parse every Excel file, extract username, group and positions; flag files that need a human look | `ingest.py` | `submissions/<user>.json` |
| 4 | Automation | Fetch closing prices (buy date + latest) and FX rates | Yahoo Finance chart API (free, no key) via `update.py` | Portfolio values in EUR |
| 5 | Automation | Rank within SV / INT groups, render the dashboard | `update.py` + `template.html` | `dashboard.html` (static, self-contained) |
| 6 | Automation | Publish | GitHub Actions (weekly schedule + "Run workflow" button) → GitHub Pages | Public URL that is always current |
| 7 | Students | Open the link, type their username | Any browser | Their rank, return, value, group standings |

The only manual step left is **step 2** — about two minutes per week.
Steps 3–6 run automatically every Friday after market close
(`.github/workflows/update.yml`), or on demand from the Actions tab.

## Rules implemented (same as the course)

- Start capital €10,000, up to 5 positions, unspent money stays as cash
- Buy price = closing price on the buy date (or the next trading day)
- Foreign shares converted to EUR at the buy date's exchange rate
- Price return only, no dividends
- Ranking within each group; students without a submission stay unranked

## Run it locally

```bash
pip install openpyxl                      # only dependency
python3 ingest.py submissions-xlsx --dry-run   # preview extraction
python3 ingest.py submissions-xlsx             # xlsx -> submissions/*.json
python3 update.py                              # live prices -> dashboard.html
```

Open `dashboard.html` in a browser. Same-day reruns use a price cache and
take seconds.

## Repo layout

```
submissions-xlsx/   fake student Excel files (the itslearning download, step 2)
submissions/        generated per-student JSON (step 3)
ingest.py           Excel -> JSON converter
update.py           price fetching, valuation, ranking, dashboard generation
template.html       dashboard design; DATA block and date are regenerated
dashboard.html      the published page (output)
.github/workflows/  weekly automation
```

## Notes

- Ticker symbols use Yahoo Finance conventions: `NOKIA.HE` (Helsinki),
  `VOLV-B.ST` (Stockholm), `MC.PA` (Paris), `SAP.DE` (Frankfurt), plain
  symbols for US stocks.
- If a file is flagged during ingestion ("no positions found", budget over
  €10,000), open it and fix the submission by hand — the pipeline never
  guesses.
- The Pages site is public; the dashboard contains usernames only, no real
  names.
- For a zero-code alternative, the same idea works in Google Sheets with
  `GOOGLEFINANCE()` — this repo is the more faithful option for the exact
  competition rules above.
