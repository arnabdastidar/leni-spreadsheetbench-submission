# SpreadsheetBench output files — `spreadsheetbench_direct_codeagent-v3-final`

Output spreadsheets produced by the LeniQ coding-agent v3 eval run, downloaded from S3
by `../download_spreadsheetbench_files.py`.

**Source:** prod DB `leniq_eval.eval_runs_spreadsheet` where
`benchmark_code = 'spreadsheetbench_direct_codeagent-v3-final'`
→ `output_file_ids` → `chat.created_attachments.s3_key` → bucket `leniq-attachements-dev`.

**Naming:** `<bench_task_id>__<original filename>.xlsx`

## File count: 400 files for 400 eval rows

The benchmark has **400 rows** in the DB, one output file each → **400 files**.

Two things about how we got here are worth recording:

- **Task `50631` initially had no file.** When first downloaded (2026-07-13), its
  `output_file_ids` array was empty, so the folder held 399 files. The output was
  registered in the DB later that day (file `edited_1_50631_init.xlsx`), and a
  re-run of the download script picked it up — bringing the count to 400.
- **Two rows (`52964`, `73-45`) once had duplicate uploads.** Each briefly recorded
  2 uploads (seconds apart, same filename, different S3 folders). For `52964` the two
  differed (first attempt used VLOOKUP formulas in column C; the revision used
  INDEX/MATCH) — we kept the later revision. For `73-45` the two were byte-identical.
  The download script now uses `DISTINCT ON (eval row) ... ORDER BY createdAt DESC`,
  so it only ever fetches the latest upload per row; the earlier duplicates were
  deleted from this folder.
