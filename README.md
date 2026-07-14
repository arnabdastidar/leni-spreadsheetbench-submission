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

---

## Addendum (2026-07-14): defects found in the official evaluation script, and why our evaluation pipeline was used

### Defects in the official evaluation pipeline

While cross-checking this submission against the official evaluation code
([RUCKBReasoning/SpreadsheetBench](https://github.com/RUCKBReasoning/SpreadsheetBench)
@ `49b73a9`, run verbatim under its pinned environment, `openpyxl==3.1.3`), we
found that the official script cannot score some Verified-400 tasks at all:
their `answer_position` metadata in the official `dataset.json` is malformed,
and the official parser crashes or fails on it.

The decisive test: running the official `evaluation.py`, unmodified, over a
golden-vs-golden dataset (input = answer = the task's own golden file, all 400
tasks) — a sanity check that must score 100% — scores **396/400**. Four tasks
score 0 against their own reference answers:

| Task | `answer_position` as shipped | Failure in the official pipeline |
|---|---|---|
| `130-9` | `'b2b, sez, de'!A5:V10` | The sheet is literally named `b2b, sez, de`; the script's `split(',')` shreds it → `ValueError` → scored 0 |
| `283-32` | `Sheet3'!A:G,'Sheet4'!A:G` | Row-less range `A:G` crashes `parse_cell_range` (`int('')`) → scored 0 |
| `49300` | `\xa0'Sheet1'!C2:C3` | Leading U+00A0 non-breaking space corrupts the sheet-name lookup → "worksheet not found" → scored 0 |
| `45944` | `G4:G6, G11:G13, G20:G22` | Spaces after commas make the parser read column `" G"` → garbage cell references → crash → scored 0 |

A fifth defect works in the opposite direction: `73-45`'s `'Sheet1'!BD2:308`
(end column missing) parses to an **empty cell list**, so every submission
receives a vacuous pass on it.

These failures occur while parsing *where to look*, before either
spreadsheet's contents are compared — independent of model outputs, OS, or
library versions. The dataset tarball is byte-identical between the GitHub
repo and the HuggingFace release and has never been updated since its first
commit (2025-12-03). Two of the defects are raised on the maintainers'
tracker and remain unaddressed as of 2026-07-14:

- `130-9` (comma in sheet name): https://github.com/RUCKBReasoning/SpreadsheetBench/issues/33
- `283-32` (malformed range): https://github.com/RUCKBReasoning/SpreadsheetBench/issues/36

### Why `results.json` was produced with our evaluation pipeline

Because of the defects above — and because the official `evaluation.py`
cannot score Verified-400 outputs as shipped (it hardcodes three test cases
named `{n}_{id}_input/answer.xlsx`, while Verified-400 ships one test case
named `1_{id}_init/golden.xlsx`, with the model-output path commented out) —
`results.json` in this repository was produced by our evaluation pipeline:
the official comparison logic with tolerant parsing of the malformed
metadata, labeled `mode: official-with-metadata-tolerance` in the file.

### Cross-check against the unmodified official script

| Run | Hard restriction |
|---|---|
| Official script, dataset as shipped | 363/400 = 90.75% |
| Official script, metadata-corrected dataset (5 documented string fixes) | 366/400 = 91.50% |
| This submission (tolerant pipeline) | **365/400 = 91.25%** |

The reconciliation is exact. The as-shipped official run loses `130-9`,
`283-32`, `49300` (−3): our outputs for these three are **content-identical
to the goldens** — verified cell-by-cell over the intended ranges using the
official comparison functions under the official pinned environment; they are
unscoreable only because of the metadata defects. It gains `45896` (+1),
whose formulas were correct but carried stale cached values; the official
formula-recalculation step (`open_spreadsheet.py`) refreshes them — a false
negative in our pipeline. `45944` fails in every pipeline (our output is
genuinely incorrect there), and `73-45` passes in every pipeline.

The full reproduction package (pristine official repo as a submodule, the
exact staged inputs for both official runs, both result files, and setup
steps to re-run the unmodified official script) is available on request:
`soulrooms/LeniQ-SpreadsheetBench-Results` (private).
