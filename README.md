# SpreadsheetBench Verified — Leaderboard Submission: Leni

Submission of **Leni** (production AI analyst agent, Leni Inc.) for the
SpreadsheetBench Verified (400-task) leaderboard.

**Result: 365/400 = 91.25%** (Cell-Level Manipulation 254/275 = 92.36%;
Sheet-Level Manipulation 111/125 = 88.80%), pass@1, single attempt per task.

This repository contains only the submission materials, per the benchmark's
submission guidelines. Nothing else.

## Dates (complete timeline)

| Event | Date |
|---|---|
| Evaluation run (all 400 tasks, single frozen configuration) | 2026-04-16 to 2026-04-20 |
| Output workbooks exported from production file storage | 2026-07-13 |
| Official evaluation code executed over outputs (results.json) | 2026-07-13 |
| Submission prepared and sent | 2026-07-13 |

## Configuration

- Executor: Claude Opus 4.6 (Anthropic API), sandboxed code execution,
  adaptive extended thinking, unmodified production spreadsheet system
  prompt. No benchmark-specific tuning.
- Verification loop: after each edit, formulas are recalculated with
  LibreOffice headless, values are read back via openpyxl
  (data_only=True), and Leni-Cell-S, our small post-trained verifier
  model, compares the recalculated cells against task intent; at most
  two fix iterations, then a final recalculation pass.
- Configuration ID: `spreadsheetbench_direct_codeagent-v3-final`.

## Contents (mapped to the submission guidelines)

1. **`running_log.jsonl`** — the running log: one JSON record per task
   (400 records) with model inputs, intermediate steps, outputs, and the
   output-file identifier, captured during the April inference run.
2. **`outputs/`** — the 400 output .xlsx files produced by the system,
   named `<task_id>__<original filename>.xlsx`. `outputs/PROVENANCE.md`
   documents the export pipeline and two file-level edge cases
   (a late-registered upload and two deduplicated double-uploads).
3. **`results.json`** — per-task results from running the evaluation code
   over the output files (400 tasks, hard/soft restriction scores,
   per-task error diagnostics for the 35 failures). The run is labeled
   `mode: official-with-metadata-tolerance`; value comparison is strict
   (see failure diagnostics, e.g. a fail on 5279.174999999999 vs
   5279.175).

## Verification we performed before submitting

- All 400 per-task hard scores in `results.json` match our internal
  evaluation database record of the April run exactly (0 discrepancies).
- All 400 output files verified one-to-one against the export manifest.

Contact: Arunabh Dastidar, CEO & Founder, Leni Inc. — arunabh@leni.co
