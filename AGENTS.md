# FX rates EUR — agent notes

## What this folder is

Working package for ECB-based EUR FX rates: LeaseManagement import fills and ad-hoc summary workbooks (with daily 底稿).

## How other agents should start

1. Read and follow the project skill (authoritative workflow):
   - [`.cursor/skills/fill-fx-rates-ecb/SKILL.md`](.cursor/skills/fill-fx-rates-ecb/SKILL.md)
2. Prefer regenerating deliverables from ECB API per the skill; do not keep stale one-off scripts or raw CSV dumps in this folder unless the user asks.

## Why Skill (not only this file)

| File | Role |
|------|------|
| **Skill** `fill-fx-rates-ecb` | Procedure + hard accounting rules; use when filling 汇率 / FX templates |
| **AGENTS.md** (this file) | Project map of current deliverables |

Do **not** duplicate the full procedure here. Update the skill when rules change.

## Keep in this folder

| File | Purpose |
|------|---------|
| `AGENTS.md` | This map |
| `.cursor/skills/fill-fx-rates-ecb/SKILL.md` | Workflow |
| `FX_rates_EUR_2026_filled_2026-07-17.xlsx` | LeaseManagement import: 2025 + 2026 (partial 2026 through Jul) |
| `FX_rates_EUR_2026_working_paper.xlsx` | 底稿 for the original 9-currency 2026 fill |
| `FX_rates_EUR_2026Q2_MXN_USD_GBP_PLN_CNY_2026-08-05.xlsx` | Standalone Q2 2026 table (MXN/USD/GBP/PLN/CNY): Summary + Daily_Rates |

## Do not keep (already cleaned)

- Undated `*_filled.xlsx` when a dated final exists
- Intermediate `ecb_raw*.csv`, `daily_rates_*.csv`, `monthly_audit.json` (regenerable; daily detail lives in working papers)
- One-off `build_fx_rates.py` with frozen dates (skill replaces it)

## One-line domain reminder

ECB quote = foreign per EUR → table rate = `1 / ECB`; average over **published trading days only**; month-end = **last published day** of the month.
