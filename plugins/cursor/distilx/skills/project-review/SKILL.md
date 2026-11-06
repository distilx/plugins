---
name: project-review
description: Reviews a DistilX project and writes a structured report from tool results. Use when the user asks to review, explain, summarise or health-check a DistilX project, for example "review project X", "what does this DistilX project do", "is project X healthy", "summarise the project".
---

# Project review

Review the project given in the user's request with the distilx tools only. Do not guess; every statement traces to a tool result.

## Steps

1. Resolve the project. A project given as an id starting with `files/` is used as is. Otherwise call `distilx_list_projects` with `query`; when several projects match, take the one in scope `own`, else list the matches and stop.
2. `distilx_get_project`: record sheets, tables with kind and column count, `endpoints`, `error_count`, `other_errors`, `other_files`, `size_bytes`, `truncated`.
3. Errors: when `error_count` is above zero, call `distilx_get_tables` for the tables whose outline shows errors, at most 50 per call, and record each error with its table and column.
4. Data in: `distilx_list_sources` for sources, `unresolved` source names and `file_inputs` with their `missing` and `id_tables`, and `distilx_list_project_files` when the project reads files, whose `updated_at` is when that content was uploaded; `distilx_get_project_file` for the outline's `other_files`, the notes, specs and reports kept in the branch; then `distilx_list_datasets` with `used_only: true` and no `source` for every imported dataset, its `used_by` formulas and `stale_count`.
5. Data out: `distilx_list_export_destinations`, `distilx_list_exported_tables` (note `last_sync` status and errors), `distilx_list_pipelines` with `last_run`, and `distilx_list_pipeline_runs` with `limit: 5` when a pipeline exists.
6. Endpoints: when `endpoints` is not empty, `distilx_list_endpoints` for request and response tables.
7. Intent: `distilx_get_branch_history` on the default branch for recent merges and turns, and a merge's `source_cursor` for the turns behind it; take intent from them, the files read in step 4, table docs and names.
8. Before explaining any formula, read the relevant `distilx_docs` node.

## Report

Sections, in this order: Purpose, Structure, Data in, Data out, Health, Open questions.

- Purpose: what the project computes, from the branch's notes, table docs and table names; mark inferences as such.
- Structure: sheets, table counts by kind, endpoints and their tables.
- Data in: one line per source with connector, importing tables and stale imports; unresolved sources and file inputs.
- Data out: exported tables with destination and last sync status; pipelines with schedule and last run.
- Health: compile errors by table and column, failed syncs, failed runs, stale imports, `missing` file paths and `id_tables` (calls still passing a file id, which no longer compile), each with its tables; "none found" when a check was made and clean.
- Open questions: what could not be verified, and what was cut by `truncated`, `skipped` or `warnings`.

Link each table, column and error in the Health, Data in and Data out rows, every repeat in Health included, and a table's first mention elsewhere, with the `link` of the tool entry it came from: errors from `distilx_get_tables` or `distilx_check_project`, endpoint tables from `distilx_list_endpoints`, imports from `distilx_list_datasets`, exports from `distilx_list_exported_tables`. A table whose entry carries no `link` is named without one; build no URL.

End with `distilx_show` for the project, its `link` when set, the project id, and the `branch` name and `version` the report was read from. Pass that `branch` to `distilx_show` when it is not the default.
