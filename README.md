# African Gig-Economy & Digital Wallet Risk — Audit

A DataDNA dataset challenge: 50,000 wallet transactions across 4 African markets (Nigeria,
Kenya, Ghana, South Africa), 2023–2024. This repo runs a Spark join over the raw data, then
audits the challenge brief's headline fraud/risk claims against the actual numbers — and
finds none of them hold up. See [`skills/AUDIT_PROCESS.md`](skills/AUDIT_PROCESS.md) for the
full finding and every command used to get there.

**Live result:** [Gig Wallet Risk Audit](https://claude.ai/code/artifact/e6148614-afdd-4559-8ed2-61e5ff4f5cab) (one-page visualization, private — ask the owner for access)

---

## 0. Prerequisites

Install these before cloning:

| Tool | Why | Check with |
|---|---|---|
| [Claude Code CLI](https://claude.com/claude-code) | Runs the whole workflow conversationally | `claude --version` |
| Python 3.11+ with pip | Data analysis (pandas/numpy), the Spark join (PySpark) | `python3 --version` |
| Java (OpenJDK 17+) | PySpark needs a JVM | `java -version` |
| Node.js 18+ | Color-palette validation when building visualizations | `node --version` |
| `jq` | Validates hook/settings JSON | `jq --version` |
| Google Chrome | Headless-rendered for visualization QA screenshots and PNG/JPEG export | any recent version |

macOS ships `say` (terminal voice) and `sips` (image conversion) already — nothing to install there.

## 1. Clone and install Python dependencies

```bash
git clone <this-repo-url>
cd "DataDNA Dataset Challenge - 2026-08 - African Gig-Economy and Digital Wallet Risk"
pip install -r requirements.txt
```

`requirements.txt` also documents the non-pip tools above (Java, Node, jq, Chrome) — nothing
in this project was pip-installed beyond what's pinned there.

## 2. Open Claude Code in this folder

```bash
claude
```

A `SessionStart` hook (`.claude/hooks/session_greeting.sh`) fires automatically: it speaks a
greeting out loud (macOS `say`) and prints a 9-option menu of available visualizations. Wait
for it, then reply with a number or describe what you want — Claude won't build anything
before you choose.

`CLAUDE.md` at the repo root (a copy of [`skills/CLAUDE.md`](skills/CLAUDE.md)) is what makes
this automatic — Claude Code only auto-loads a **root-level** `CLAUDE.md`, so it must stay
there (not just in `skills/`) for the wait-for-your-pick behavior to apply on a fresh clone.
If you ever edit the instructions, edit both copies, or delete one and point to the other.

## 3. The 9 available visualizations

1. KPI summary
2. Claims vs. data
3. Dictionary vs. observed
4. Temporal trends
5. Entity analysis
6. Transaction patterns
7. Channel × market heatmap
8. Velocity vs. fraud
9. Recommended next steps

Ask for any one of these, or say "build the full one-page report" for all 9 combined. Anything
outside this list gets redirected to gauravkumarjha1195@gmail.com rather than built.

## 4. What happens when you ask for a visualization

1. **Data-freshness check** — Claude compares `data/*.csv` timestamps against
   `output/fact_transactions_with_channel*`. If source data is newer, it re-runs
   `Code/Data model.ipynb` (the PySpark join across `fact_transactions` + the 4 `dim_*`
   tables) and writes fresh output to `output/`. If not, it tells you "There is no any new
   Data Append" and skips straight to analysis.
2. **Analysis** — all figures are computed live from `output/fact_transactions_with_channel*`,
   never assumed.
3. **Build** — the chart is authored as a self-contained HTML page (inline SVG, no external
   libraries) and published as a Claude Artifact — a shareable link.
4. **QA** — before publishing, Claude renders the page headlessly and inspects full-resolution
   screenshots for layout bugs, not just a quick look.

## 5. Viewing and downloading the result

- **Shareable link**: the published Artifact URL (see top of this file, or ask Claude for the
  current one). Private by default — share it yourself from the page's share menu if needed.
- **Local copy with working downloads**: `output/visualizations/gig_wallet_risk_audit.html`
  opens directly in any browser (`open output/visualizations/gig_wallet_risk_audit.html`) and
  has real "Download PNG" / "Download JPEG" buttons — these only work in this local copy, not
  in the hosted Artifact, because claude.ai's viewer sandbox blocks script-driven downloads.
  The rendered `.png` / `.jpg` files sit right next to it in the same folder.

## 6. Optional: email the link / Notion summary

If asked, Claude can email the visualization link to a provided address, or write a Notion
page summarizing the findings. These require the Gmail / Notion connectors to be authorized
for your Claude account first (`claude mcp` or the claude.ai connector settings) — Claude will
always confirm with you before actually sending or posting anything.

## Repo structure

```
Code/Data model.ipynb          Spark join: fact_transactions + 4 dim tables -> output/
data/                          Source CSVs (fact + dimensions)
docs/                          Data dictionary, schema, validation report (read-only)
output/
  fact_transactions_with_channel(.parquet|/)   Joined dataset, csv + parquet
  visualizations/               Local HTML + PNG/JPEG exports with working downloads
skills/
  Flow.md                       The master workflow Claude follows, in order
  Data_instruction.md           Data-freshness / pipeline-rerun rule
  CHALLENGE_BRIEF.md            The original analysis brief and 6 headline claims
  instruction.md                Visualization spec (9 views, voice, branding, delivery)
  AUDIT_PROCESS.md              Full log of every step and command used to build the audit
  CLAUDE.md                     Standing instructions for how Claude should run this project
.claude/
  settings.json                 SessionStart hook wiring (voice + menu)
  hooks/session_greeting.sh     The hook script itself
requirements.txt                Pinned Python deps + non-pip tool notes
EDA_REPORT.html                 Auto-generated profiling export (plots only, no narrative)
CLAUDE.md                       Root copy of skills/CLAUDE.md — must live here to auto-load
```

## Headline finding (short version)

`is_fraud_flagged`, `is_disputed`, and `is_reversed` all sample at **~50%** against a spec of
**7% / 5% / 8%** — the event generator looks like it's flipping a coin rather than sampling
the documented weights. None of the 6 claims in `CHALLENGE_BRIEF.md` reproduce on this
extract. Full detail in [`skills/AUDIT_PROCESS.md`](skills/AUDIT_PROCESS.md) and the
visualization itself.

---
© Gaurav Kumar. All rights reserved. For a visualization outside the 9 listed above, contact
gauravkumarjha1195@gmail.com.
