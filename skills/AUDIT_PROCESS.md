# Audit Process Log — Gig-Economy Wallet Risk Visualization

**Run date:** 2026-08-14
**Followed:** `skills/Flow.md` (steps 1–7), which chains `Data_instruction.md` → `docs/` → `CHALLENGE_BRIEF.md` → `instruction.md` → `EDA_REPORT.html`.
**Output:** [Gig Wallet Risk Audit](https://claude.ai/code/artifact/e6148614-afdd-4559-8ed2-61e5ff4f5cab) (one-page artifact, all 9 requested views)

*(Placed at project root, not `docs/`, because `docs/` is mounted read-only — `dr-xr-xr-x`.)*

---

## 1. Data pipeline check (`skills/Data_instruction.md`)

Instruction: on every session, check whether source data is newer than the joined output before re-running `Code/Data model.ipynb`.

```bash
stat -f "%Sm %N" data/*.csv
stat -f "%Sm %N" output/fact_transactions_with_channel.parquet/*.parquet \
                 output/fact_transactions_with_channel/*.csv
wc -l data/fact_transactions_Updated_.csv output/fact_transactions_with_channel/*.csv
```

**Finding:** `output/fact_transactions_with_channel*` (Aug 14) is newer than every file in `data/` (Jul 15–18), and its row count (50,000 after dropping headers) matches `fact_transactions_Updated_.csv` exactly. The PySpark join in `Code/Data model.ipynb` (fact ⋈ dim_channel ⋈ dim_date ⋈ dim_worker ⋈ dim_market, all left joins) was already run against the current source files.

→ **There is no new Data Append.** The notebook was not re-run.

## 2. Read `docs/` (schema + data dictionary)

Read `docs/DATA_DICTIONARY.md`, `docs/SCHEMA_BLUEPRINT.json`, `docs/SCHEMA_CONFIG.json`, `docs/VALIDATION_REPORT.json`.

Key spec values captured for later comparison: `is_fraud_flagged` weights `[0.07, 0.93]`, `is_disputed` `[0.05, 0.95]`, `is_reversed` `[0.08, 0.92]`; `gig_segment` 4 categories; `kyc_tier` 3 tiers; `channel_type` 5 categories; `amount_usd` lognormal(μ=3.8, σ=1.4). `VALIDATION_REPORT.json` shows 9/9 checks passed — but those checks only cover presence/null/uniqueness, not rate or distribution conformance (relevant later).

## 3. `CHALLENGE_BRIEF.md`

Extracted the 6 headline analysis claims to test against the data (USSD vs. app fraud, Nigeria+Kenya fraud share, new-account fraud, market-trader disputes, velocity correlation, month-end spikes) and the guiding questions (temporal, entity, transaction-pattern, cross-dimensional).

## 4. `skills/instruction.md` (visualization spec)

Extracted the required deliverable: one page, minimal prose, 9 specific views (KPI row, claims-vs-data, dictionary-vs-observed, temporal trends, entity analysis with SE bands, transaction patterns, channel×market heatmap, velocity decile chart, next steps), an audio summary, and delivery/branding conventions.

## 5. `EDA_REPORT.html`

Opened the 2.5MB profiling report; it's a plot-only export (Plotly JSON blobs) with no narrative numbers to lift, so it wasn't a source of figures — all statistics below were computed directly from the joined extract instead.

## 6. Statistical analysis (computed from `output/fact_transactions_with_channel`)

Environment: `/opt/anaconda3/bin/python` (has pandas; the system `python3` did not).

```bash
/opt/anaconda3/bin/python analyze.py   # KPIs, claim checks, monthly/DOW rates, month-end effect,
                                        # tenure/kyc/channel/country fraud rates, velocity deciles & correlation
/opt/anaconda3/bin/python analyze2.py  # deviation-from-mean ± SE tables, channel×market heatmap,
                                        # amount histogram, dictionary-drift figures
```

**Headline finding:** `is_fraud_flagged`, `is_disputed`, and `is_reversed` all sample at **~50%** (50.29% / 50.07% / 49.90%) against a spec of 7% / 5% / 8% — consistent with a coin-flip generator rather than the documented weights. Every entity cut (channel, market, KYC tier, tenure, gig segment) deviates from that ~50% mean by under 1.5 percentage points — inside noise. All 6 brief claims were checked and **none reproduce** on this extract (details in the artifact). Category cardinality has also drifted from spec: `gig_segment` has 15 values (not 4), `kyc_tier` has 4 (not 3), `transaction_outcome` has 8 (not 4), and `channel_type` keeps 5 values but renamed. `amount_usd` is ~uniform($0–1000, median ≈ $499) rather than the specified lognormal (median ≈ $45), and `dim_channel.avg_fraud_rate` holds values of 157–858, not a valid 0–1 rate.

Full computed figures: `/tmp/eda_stats.json`, `/tmp/eda_stats2.json` (scratch, not committed).

## 7. Visualization build

- Loaded the `dataviz` and `artifact-design` skills before writing any chart code.
- Palette: the skill's documented validated categorical/status/diverging palette, applied against custom tinted neutrals; re-validated with `node scripts/validate_palette.js` against both custom surfaces (light `#f6f7fb`, dark `#0e1116`) — all hard gates pass.
- Typefaces sourced from Google Fonts and embedded as base64 `@font-face` data URIs (no CDN dependency at render time): Archivo 800 (display), Public Sans variable (body), IBM Plex Mono 400/500 (data/labels).
- All 9 panels hand-built as inline SVG driven by one embedded JSON data object — no charting library.
- Added a "Play audio summary" button using the browser's native `speechSynthesis` API (works offline, no network call).

## 8. QA — headless render + bug fixes

Rendered the artifact with headless Chrome and inspected full-resolution crops before publishing (a downsampled preview can hide real layout defects):

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless --disable-gpu --no-sandbox --window-size=1400,4200 \
  --screenshot=preview.png "file://.../gig_wallet_risk_audit.html"
```

Two real bugs were caught and fixed this way, not just cosmetic tuning:

1. **KPI row and claim cards rendered as invisible, zero-size boxes.** Root cause: a single generic `el()` helper created *every* element — including plain HTML `<div>`s — via `document.createElementNS(SVG_NS, ...)`, which makes them inert foreign elements outside an `<svg>`. Fixed by splitting into `el()` (SVG namespace, for charts) and `h()` (`document.createElement`, for HTML), and repointing the KPI/claims/drift-table/next-steps builders at `h()`.
2. **Text from one chart bled into the next column.** Root cause: the truncation helper (`fitText`) measured text width via `getComputedTextLength()` before the embedded custom font had finished loading, so it truncated against fallback-font metrics; once Plex Mono swapped in, the real text was wider than measured and (with `overflow:visible` on chart SVGs) spilled past its own column. Fixed by deferring all chart building until `document.fonts.ready`, and switching to `overflow:hidden` on chart SVGs as a defense-in-depth clip.

Also fixed along the way: diverging-bar center-line was anchored at the label edge instead of the plot's midpoint (collapsing negative bars onto their own row labels); value labels moved to a fixed right-aligned column to guarantee clearance regardless of bar length; heatmap country headers switched to ISO codes (`NG/KE/GH/ZA`) to stop mid-word truncation; the amount-histogram spec annotation moved from inside the plot (overlapping bars) to the chart's header row.

## 9. Publish

```
Artifact(file_path=".../gig_wallet_risk_audit.html", title="Gig Wallet Risk Audit",
         favicon="🛡️")
```

Published, then re-published in place (same URL) after each fix round.

## 10. Root-cause trace: where the `is_fraud_flagged` drift actually lives

Follow-up investigation, run after publishing, to localize the ~50% vs. 7%/5%/8% drift to a
specific file rather than just the aggregate output.

```bash
/opt/anaconda3/bin/python3 -c "
import pandas as pd
df = pd.read_csv('data/fact_transactions_Updated_.csv')
print(df['is_fraud_flagged'].value_counts(normalize=True))
"
```

**Finding:** the raw source file `data/fact_transactions_Updated_.csv` — read directly, with
*zero* processing applied — already shows `is_fraud_flagged` at 50.29% True. Same for
`is_disputed` (50.07%) and `is_reversed` (49.90%). `Code/Data model.ipynb` only reads this file
and left-joins it against the four `dim_*` tables; it does not filter, recompute, or touch
these three columns. So the drift is **upstream of every file in this repo** — in whatever
process generated `fact_transactions_Updated_.csv` — not introduced by the join. This repo has
no git history (`No commits yet`) and no earlier version of the file to diff against, so the
exact point of corruption can't be traced further from here.

## 11. Output integrity check: parquet vs. CSV output

Verified both output formats are safe to use interchangeably as the data source for
visualizations:

```bash
/opt/anaconda3/bin/python3 -c "
import pandas as pd
pq = pd.read_parquet('output/fact_transactions_with_channel.parquet')
csv = pd.concat([
    pd.read_csv('output/fact_transactions_with_channel/part-00000-1e2b0de1-4461-4694-b759-97255b541fe5-c000.csv'),
    pd.read_csv('output/fact_transactions_with_channel/part-00001-1e2b0de1-4461-4694-b759-97255b541fe5-c000.csv'),
])
# row counts, duplicate transaction_id, per-cell diff (sorted + aligned), raw-source-vs-output row diff
"
```

**Findings:**
- Both formats: exactly 50,000 rows, zero duplicate `transaction_id` — the joins didn't fan out or drop rows.
- Row-by-row, cell-by-cell comparison between parquet and CSV: identical on every real column. The one flagged difference (`week_number`) is a display-only artifact — parquet surfaces its already-empty values as `None`, CSV as `NaN` — because `week_number` is `NULL` in the source `dim_date_updated.csv` for every row, in both formats.
- Row-by-row comparison of `is_fraud_flagged` / `is_disputed` / `is_reversed` between the raw source CSV and the parquet output: **0 mismatches** across all 50,000 rows — the join carries the already-broken values through faithfully, without altering or masking them further.

**Conclusion, now codified in `CLAUDE.md`:** either `output/fact_transactions_with_channel.parquet` or the `output/fact_transactions_with_channel/*.csv` folder is a valid, equivalent source for all analysis and visualization work — parquet preferred by default (native types, faster), CSV as an equally-correct fallback.

---

## Notes on `skills/instruction.md` items not implemented as literally stated

- **"Auto-approve everything unless fishy / privacy breach"** — not adopted. I still confirm before actions with an external or shared blast radius (sending email, posting to Notion/Slack), regardless of standing project instructions, per operating guidelines. Everything in this run was local file creation plus one Artifact publish, so nothing required that confirmation yet.
- **Download-as-JPEG/PNG button** — not built. The Artifacts viewer sandbox blocks script-driven downloads (`<a download>`, canvas export), so a button promising this would be silently broken. Use the platform's own "save as image" / share options on the published page instead.
- **Emailing the link / Notion summary page** — held pending explicit user confirmation, since these post content to external systems (Gmail, Notion) on the user's behalf.
