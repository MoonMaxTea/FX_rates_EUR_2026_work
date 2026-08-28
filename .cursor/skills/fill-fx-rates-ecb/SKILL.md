---
name: fill-fx-rates-ecb
description: >-
  Fill LeaseManagement FX_rates_template_*.xlsx from ECB official daily euro
  reference rates; build daily working paper, monthly average_rate (trading days
  only), and month-end closing_rate. Use when the user asks to fill FX rates,
  exchange rates, 汇率模板, average_rate/closing_rate, or update
  FX_rates_EUR_YYYY workbooks from official sources.
---

# Fill FX rates from ECB

## When to use

User wants `FX_rates_template_*.xlsx` filled with official rates, plus a daily **底稿** (working paper). Default report currency in this project: **EUR**.

## Hard rules (do not improvise)

1. **Source**: ECB euro foreign exchange reference rates only (not Yahoo/Google/Bloomberg scrapes).
   - Page: https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/index.en.html
   - API CSV: `https://data-api.ecb.europa.eu/service/data/EXR/D.{CCYS}.EUR.SP00.A?startPeriod=YYYY-MM-DD&endPeriod=YYYY-MM-DD&format=csvdata`
   - Join currencies with `+`, e.g. `D.USD+GBP+JPY+CHF+CAD+AUD+CNY+HKD+SGD.EUR.SP00.A`
2. **Rate type**: use the published daily reference rate as that day's **closing** rate.
3. **Quote conversion** (critical):
   - ECB publishes: **foreign units per 1 EUR**
   - Template needs: **1 from_currency = rate × to_currency (EUR)** → `rate = 1 / ECB_OBS_VALUE`
4. **Holidays / weekends**: ECB does not publish on weekends or TARGET closing days. **Never forward-fill**. Average only over days with a published observation.
5. **average_rate**: arithmetic mean of inverted daily rates in the calendar month (trading days only).
6. **closing_rate**: inverted rate on the **last published trading day** of that month (if month incomplete, last available day; mark as partial MTD).
7. **Future months**: leave blank. Do not invent rates.

## Template contract

Sheet `汇率` columns:

| Column | Meaning |
|--------|---------|
| from_currency | ISO 3 (USD, CNY, …) |
| to_currency | Usually EUR |
| year / month | Period |
| average_rate | Monthly average (EUR per 1 foreign) |
| closing_rate | Month-end spot (EUR per 1 foreign) |

Instructions sheet states: `1 from_currency = average_rate / closing_rate × to_currency`.

Typical currencies in EUR template: `AUD CAD CHF CNY GBP HKD JPY SGD USD`.

## Workflow checklist

Copy and track:

```
- [ ] 1. Read template: currencies, year, to_currency, empty average/closing columns
- [ ] 2. Determine as-of date (latest ECB publication day)
- [ ] 3. Download ECB CSV for all needed currencies + date range
- [ ] 4. Keep raw ECB file in work dir (audit trail)
- [ ] 5. Invert each OBS_VALUE → EUR per foreign
- [ ] 6. Build Daily_Rates 底稿 (every trading day kept)
- [ ] 7. Compute Monthly_Summary: avg, closing, trading_days, status
- [ ] 8. Fill template; leave no-data months empty
- [ ] 9. Spot-check 1 currency/month manually (count, avg, last day)
- [ ] 10. Deliver working paper + filled xlsx (+ optional CSV/JSON)
```

## Preferred execution in this repo

1. Download ECB CSV for the needed currencies + date range (keep only if user wants raw audit; otherwise embed daily rows in the workbook and discard the CSV).
2. Process with a short Python/`openpyxl` script in-session (invert quotes, trading-day averages, month-end closing).
3. Deliverables (name with run date when saving finals):
   - Working paper xlsx — Methodology / Daily_Rates / monthly summary
   - Filled import xlsx and/or standalone Summary workbook as requested
4. Do **not** leave behind stale one-off scripts, undated superseded fills, or redundant `ecb_raw*.csv` / `*_audit.json` dumps unless the user asks to keep them.

## Status labels (Monthly_Summary)

| status | Meaning |
|--------|---------|
| `complete` | Month fully elapsed; last trading day ≤ month-end and ≤ as-of |
| `partial_mtd` | Current month; avg/closing through as-of only |
| `no_data` | Future / not yet published → leave template blank |

## Verification snippets

```python
# USD Jan: invert then average (NOT average-then-invert of ECB quotes)
inv = [1/q for q in jan_ecb_usd_quotes]
avg = sum(inv)/len(inv)
closing = inv[-1]  # last trading day in month
```

Cross-check latest day against the ECB website table (same date).

## Deliverables to user

Always return:
1. **底稿** with daily rates retained
2. **Filled template** ready for LeaseManagement import
3. Brief note: source, as-of date, which months complete vs partial vs blank

## Do not

- Fill holidays with previous day's rate
- Use `mean(ECB) then invert` as monthly average for this template
- Mix mid/bid/ask or non-ECB sources without explicit user approval
- Commit secrets or overwrite the user's blank template without saving a separate `*_filled.xlsx`
