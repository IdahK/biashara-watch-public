# biashara-watch-public

# 📈 biasharaWatch — NSE Stock Tracker

[![NSE Stock Tracker](https://github.com/IdahK/biashara-watch-public/actions/workflows/stock_poller.yml/badge.svg)](https://github.com/IdahK/biashara-watch-public/actions/workflows/stock_poller.yml)
![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/Runs%20on-GitHub%20Actions-2088FF?logo=github-actions&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)
![Zero cost](https://img.shields.io/badge/Hosting%20cost-KES%200-006633)
![NSE](https://img.shields.io/badge/Exchange-NSE%20Kenya-006633)

> Automated tracker for all NSE-listed stocks. Scrapes live prices every 10 minutes during trading hours, updates a formatted Excel workbook, and emails you a portfolio summary every Friday — all running free on GitHub Actions with no server required.

---

## ✨ Features

- **Live market data** — scrapes all NSE-listed stocks from a live public NSE data source every 10 minutes during trading hours, no API key needed
- **Formatted Excel workbook** — two sheets: a live Market sheet and a personal Portfolio sheet with auto-calculated gain/loss
- **Weekly email report** — HTML email every Friday at 5:00 PM EAT showing your full portfolio performance
- **Zero infrastructure** — runs entirely on GitHub Actions free tier; no server, no database, no cloud bill
- **Configurable** — trading hours, email schedule, and data source all live in `config.json`, no code changes needed
- **CLI flags** — `--force`, `--start`, `--end`, `--send-email` for local testing and overrides

---

## 🏗️ How It Works

```
GitHub Actions (cron, every 10 min)
        │
        ▼
stock_poller.py
        │
        ├─► GET live NSE data source
        │
        ├─► Write Market sheet  ──► stock_prices.xlsx
        │     Ticker │ Company │ Sector │ Price │ Change % │ Volume │ Updated
        │
        ├─► Update Portfolio sheet (VLOOKUP formulas against Market sheet)
        │     Ticker │ Shares │ Buy Price │ Current Price │ Value │ Gain/Loss
        │
        └─► Friday 5 PM EAT → send HTML email via Gmail SMTP
```


---

## 📊 Output

### Market Sheet
| Ticker | Company | Sector | Price (KES) | Change (%) | Volume | Last Updated |
|--------|---------|--------|-------------|------------|--------|--------------|
| SCOM | Safaricom PLC | Telecommunication | 30 | +0.61% | 4,821,300 | 2026-02-27 14:00 EAT |
| EQTY | Equity Group Holdings | Banking | 75| +2.68% | 1,203,400 | 2026-02-27 14:00 EAT |
| KCB | KCB Group PLC | Banking | 50 | -0.97% | 983,100 | 2026-02-27 14:00 EAT |
| ... | | | | | | |

### Portfolio Sheet
Columns A–E are yours to fill in. Columns F–I are auto-calculated via VLOOKUP from the Market sheet.

| # | Ticker | Company | Shares Owned | Buy Price (KES) | Current Price | Current Value | Gain/Loss | Gain/Loss % |
|---|--------|---------|--------------|-----------------|---------------|---------------|-----------|-------------|
| 1 | SCOM | *(auto)* | 1,000 | 30.00 | *(auto)* | *(auto)* | *(auto)* | *(auto)* |

### Weekly Email
A styled HTML email delivered every Friday at 5:00 PM EAT showing total invested, current value, total gain/loss, and a per-stock breakdown.

---

## 🚀 Setup

### Prerequisites
- A GitHub account (free)
- A Gmail account with [2-Step Verification](https://myaccount.google.com/security) enabled

### 1 — Fork or clone this repo

```bash
git clone https://github.com/IdahK/biashara-watch-public.git
cd biashara-watch-public
```

Or click **Fork** at the top right of this page to create your own copy.

### 2 — Copy the config file

```bash
cp config.example.json config.json
```

Edit `config.json` if you want to change trading hours or the email schedule. The defaults match NSE hours (9 AM–5 PM EAT, Mon–Fri).

### 3 — Enable workflow write permissions

In your repo → **Settings → Actions → General → Workflow permissions** → select **Read and write permissions** → Save.

### 4 — Add Gmail secrets

Go to **Settings → Secrets and variables → Actions → New repository secret** and add:

| Secret | Value |
|--------|-------|
| `GMAIL_ADDRESS` | Your Gmail address |
| `GMAIL_APP_PASS` | A [Gmail App Password](https://myaccount.google.com/apppasswords) (16 chars, not your login password) |
| `NOTIFY_EMAIL` | Where to send the weekly report (can be same as above) |

### 5 — Trigger a test run

Go to **Actions → biasharaWatch — NSE Stock Tracker → Run workflow**. After ~60 seconds, `stock_prices.xlsx` will appear in your repo. Download it and open in Excel or Google Sheets.

### 6 — Add your portfolio

1. Download `stock_prices.xlsx`
2. Open the **My Portfolio** sheet
3. Fill in: **Column B** (Ticker), **Column D** (Shares Owned), **Column E** (Buy Price in KES)
4. Upload the file back to your repo

The script will read your entries and fill in live prices and gain/loss on every run.

---

## 🖥️ Running Locally

```bash
# Install dependencies
pip install -r requirements.txt

# Copy and edit config
cp config.example.json config.json

# Run anytime — bypasses the trading hours check
python stock_poller.py --force

# Override trading hours for this run
python stock_poller.py --start 8 --end 18

# Force-send the weekly email right now
python stock_poller.py --force --send-email
```

### CLI Reference

| Flag | Description |
|------|-------------|
| `--force` / `-f` | Run regardless of trading hours |
| `--start HOUR` | Override start hour for this run (e.g. `--start 8`) |
| `--end HOUR` | Override end hour for this run (e.g. `--end 18`) |
| `--send-email` | Send the weekly portfolio email immediately |

---

## ⚙️ Configuration

All settings live in `config.json` — edit this file, never the script:

```json
{
  "trading_hours": {
    "start": 9,
    "end": 17,
    "days": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"],
    "timezone": "EAT (UTC+3)"
  },
  "email": {
    "send_on": "Friday",
    "send_at_hour": 17
  },
  "output_file": "stock_prices.xlsx",
  "source_url": "https://example.com/nse-data"
}
```

If you change `start`/`end`, also update the cron lines in `.github/workflows/stock_poller.yml` to match (cron runs in UTC; EAT = UTC+3, so subtract 3 hours).

---

## 📁 Repo Structure

```
biashara-watch-public/
├── stock_poller.py                   # Main script
├── config.example.json               # Config template (copy to config.json)
├── requirements.txt                  # Python dependencies
├── .gitignore                        # Blocks config.json and stock_prices.xlsx
├── README.md
├── SETUP.md                          # Detailed step-by-step setup guide
└── .github/
    └── workflows/
        └── stock_poller.yml          # GitHub Actions schedule & job
```

> `config.json` and `stock_prices.xlsx` are in `.gitignore` — they stay local/private and are never committed to the public repo.

---

## 🔧 Troubleshooting

| Problem | Fix |
|---------|-----|
| Workflow not visible in Actions tab | Ensure the workflow file is at exactly `.github/workflows/stock_poller.yml` |
| Push permission error | Re-check Step 3 — workflow needs read/write permissions |
| `[ERROR] Data source structure changed` | The upstream data source updated its page — open an issue |
| Email not arriving | Double-check secrets; check spam folder; verify App Password has 2FA enabled |
| Script exits immediately locally | Add `--force` flag to bypass the trading hours check |
| Empty columns in Excel | Delete `stock_prices.xlsx` from the repo and re-run — the old file has a stale column layout |

---

## 🛠️ Tech Stack

| Tool | Role |
|------|------|
| Python 3.11 | Core scripting |
| `requests` + `BeautifulSoup4` | HTTP fetch + HTML parsing |
| `openpyxl` | Excel workbook generation and formatting |
| GitHub Actions | Free cron scheduling and execution |
| Gmail SMTP | Weekly email delivery |


---


## 🙏 Acknowledgements

NSE market data is fetched from a live public data source.
