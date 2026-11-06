---
name: data-lineage
description: Traces where a DistilX project's or table's data comes from and where it goes: sources and connectors, imported datasets with their pinned and latest synced versions, sync history, input files, exports and pipelines. Use when the user asks about data lineage, the sources or datasets behind a project or table, stale imports or imports pinned below their latest sync, what a table exports to, or which pipeline refreshes it.
---

# Data lineage

Trace lineage for the project, and optionally the table, given in the user's request, with the distilx tools only. Do not guess; every cell of the result traces to a tool result.

## Upstream

1. Resolve the project as in `project-review`: an id starting with `files/`, else `distilx_list_projects` with `query`.
2. `distilx_list_sources`: connector per source, `importing_tables`, `unresolved` source names, `file_inputs` (each `INPUT` path with its `tables`, `id_tables` and `missing`), and `distilx_list_project_files` for the `updated_at` of each file read, which is when that content was uploaded.
3. `distilx_list_datasets` with `used_only: true` and no `source`: every imported dataset with `latest_succeeded_version` and `used_by` entries (sheet, table, pinned `version`, `stale`). When a table is named, keep the datasets whose `used_by` names that table. Use `stale_only: true` only when the user wants nothing but the stale ones; a question that asks for the list of imports needs `used_only`, whatever the stale count. Note `imports_checked` and `stale_count`.
4. For a stale import, or when asked for history, `distilx_get_dataset` with `max_columns: 0` for the sync `versions` with status, times and error.
5. When a table is named, `distilx_get_tables` for it and read which tables its formulas reference (`Other[column]` or `'Other table'[column]` form; see `distilx_docs` path `language`). Follow references at most three hops back to tables that import or read files.

## Downstream

6. When a table is named, `distilx_search_dsl` with a pattern for the table name and `code_only` to find the tables that reference it.
7. `distilx_list_exported_tables`, keeping the `exports` entries whose `table` matches when a table was given, with `destination_name`, `dataset` and `last_sync`; `distilx_get_exported_table` when the last sync failed or the written columns matter.
8. `distilx_list_pipelines`: pipelines whose `tables_to_calculate` or `exports` name the table, with `schedule` and `last_run`.

## Result

Two tables.

- Upstream: source, connector, dataset, importing sheet and table, pinned version, latest succeeded version, stale.
- Downstream: table, referencing tables, destination, dataset, last export status and time, pipeline, schedule, last run status.

Link each importing, referencing and exported table with the `link` of its tool entry: `used_by` entries, `distilx_search_dsl` matches, exports. A table whose entry carries no `link` is named without one; build no URL.

List each `missing` path and each `id_tables` entry as a finding naming its tables: a missing path fails with `No file found`, and a call still passing a file id no longer compiles. Then one line with `stale_count` over `imports_checked`, and a note of anything `truncated`, `skipped` or reported in `warnings`. End with `distilx_show` for the project, its `link` when set, the project id, and the `branch` name and `version`. Pass that `branch` to `distilx_show` when it is not the default.
