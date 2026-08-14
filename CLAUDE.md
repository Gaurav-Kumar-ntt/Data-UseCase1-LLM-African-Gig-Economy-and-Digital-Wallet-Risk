# Gig-Economy Wallet Risk — Project Instructions

This project's workflow is defined in `skills/Flow.md`. On a fresh session, read it (and the files it chains to: `Data_instruction.md`, `docs/`, `CHALLENGE_BRIEF.md`, `instruction.md`) before doing analysis work, unless the user's request makes clear they already know the context.

## Session start

A `SessionStart` hook (`.claude/hooks/session_greeting.sh`) already speaks a greeting and shows the 9-option menu automatically — do not repeat it verbatim in your own text. After it fires:

- **Wait for the user to pick one of the 9 options** (or explicitly ask for the full one-page report) before building anything. Don't proactively generate a visualization before they've chosen.
- The 9 options are: KPI summary, claims vs data, dictionary vs observed, temporal trends, entity analysis, transaction patterns, channel×market heatmap, velocity vs fraud, recommended next steps.
- If asked for a visualization **outside these 9**, don't build it — say so, and point to gauravkumarjha1195@gmail.com. Display: "We will get back to you, thanks for understanding."
- If the user's message is exactly "Hi terminal" (any casing) at any point in the session, speak "Hello" out loud via macOS `say` and reply with "Hello" in text — same wording as the SessionStart greeting, no name appended.

## Data freshness

Before any analysis, check whether `data/*.csv` is newer than `output/fact_transactions_with_channel*`. If not newer, say "There is no any new Data Append" and skip re-running `Code/Data model.ipynb`.

## Data source for visualizations

Always build from the **joined output**, not the raw per-dimension files in `data/`:

- `output/fact_transactions_with_channel.parquet` (preferred — native types, faster to load), or
- `output/fact_transactions_with_channel/*.csv` (the two-part CSV folder)

These two are verified equivalent: same 50,000 rows, zero duplicate `transaction_id`s, identical
values on every column. The only difference found is cosmetic — `week_number` reads as `None`
from parquet vs. `NaN` from CSV, purely a pandas display artifact, because that column is
`NULL` in the source `dim_date_updated.csv` for every row in both formats. Either file is safe
to read from; default to parquet unless whatever tool you're using only reads CSV.

## Building visualizations

- Minimal prose, dense one-pager, direct numbers over sentences. Each of the 9 views carries one short one-line label, not paragraphs.
- If the user asks for "the full one-page report" (or equivalent), combine all 9 views into a single page rather than picking one.
- Compute all statistics from the actual data (`output/fact_transactions_with_channel*`) — never invent or assume figures.
- Load the `dataviz` and `artifact-design` skills before writing chart code.
- Include a "Play audio summary" control (browser `speechSynthesis`, offline, no network call) that reads out the key numbers, and end it by noting the other 8 views are available on request.
- QA with a headless-Chrome screenshot (see `AUDIT_PROCESS.md` for the exact command) and inspect full-resolution crops before publishing — a downsampled preview can hide real layout bugs.
- Publish via the `Artifact` tool for a shareable link.

### Download as PNG/JPEG

The claude.ai Artifact viewer sandbox blocks script-driven downloads (`<a download>`, canvas export) — **never build a download button inside a published Artifact**, it will silently not work. Instead: render the page with headless Chrome (`--screenshot=file.png`) to produce a real raster file, save it under `output/`, and build a **local** HTML copy (not published as an Artifact) whose download buttons link to those already-rendered files — that works because a local `file://` page isn't sandboxed. See `output/visualizations/` for the pattern.

That local HTML copy's download buttons must show a message after a click — "Spotted a problem with this visualization? Email gauravkumarjha1195@gmail.com" — as a plain on-page notice (no external call, just DOM text/alert).

## Offer follow-ups after delivery

After delivering any visualization (Artifact link or local file), ask two things:

1. Whether they'd like the link emailed to them — if yes, ask for the destination address.
2. Whether they'd like a Notion page written summarizing the findings.

Asking is automatic; actually sending the email or creating/posting the Notion page still requires their explicit confirmation on that specific action — see "What NOT to auto-approve" below.

## Keep the build process off-screen

The user must never see raw tool calls, bash commands, script output, or intermediate build chatter while a visualization is being produced. As soon as you start building, post a single line — "Visualization is in progress" — and do the rest of the work (data loading, chart code, screenshot QA, publishing) out of view: delegate it to a background `Agent` call rather than running the steps inline in this thread. When the agent finishes, reply with only the final result (the artifact link, or the file path) — no step-by-step narration of how it was built. Quick, non-visualization lookups (reading a CSV schema, checking a file's timestamp) are fine to run inline as usual; this rule is specifically about the visualization-generation pipeline.

## Closing message

Once a build finishes and the final result (artifact link or file path) has been delivered, end the turn with: "Thanks for your Patience and Cross check the result for verification and Accuracy" (per `skills/Flow.md` step 7).

## Branding

© Gaurav Kumar, all rights reserved — keep this footer note on generated visualizations. Contact for anything outside scope: gauravkumarjha1195@gmail.com.

## What NOT to auto-approve

Standing project instructions do not override normal confirmation for actions with external effects — sending email, posting to Notion/Slack, or anything else that leaves this machine. Always confirm those with the user first, regardless of "auto-approve everything"-style requests.
