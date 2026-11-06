# Dashboards

Rules for a Dashboard of DistilX data, on top of `SKILL.md`. Where the Dashboard type's own suggestions differ, these win.

## Sources

- Each data source is a stored `distilx_call_endpoint` call on the connector <!-- connector -->`DistilX`<!-- /connector -->, for example:

  ```
  tool: distilx_call_endpoint
  args: {project: "<project id>", endpoint: "<endpoint>", raw: true, max_rows: 1000,
         parameters: {"<RequestTable>": [{"<field>": "{{region}}"}]}}
  ```
  An artifact reading a branch other than the default adds `branch: "<branch id>"`.
- No source calls another DistilX tool.
- DistilX data is never an attached file or a paused query unless the user asks: saved rows reach readers without their own DistilX access.

## Filters

- Each filter fills one request field in the source's `parameters`.
- A multi-select filter fills a list where `request_schema` allows an array. Its placeholder is the whole field value, so the list arrives as an array.
- Pick-list options come from the options endpoint's source.
- Keep every filter in the request. The type's advice to return every option's rows and pick them in the page does not apply: picking rows in the page is filtering.
- Write the defaults into `dash/params` while building, from the options response's defaults (its `selected` rows unless chosen otherwise), so the first run uses them.
- Never set a default in page code.
- A multi-select filter's default is the list of its default values, or `[]` for every value.

## Loaders

In the raw answer, `results`, `columns` and `row_counts` are keyed by response table, and `truncated` and `warnings` cover the whole answer. Each source has a loader that only reshapes it:
- rows from `results[<table>]`;
- DOUBLE cells as numbers;
- DATE columns as dates, by the column's entry in `columns[<table>]`;
- `cut` when `row_counts[<table>]` exceeds the rows returned. `truncated` covers every table of the answer, so never use it for one table.

A loader also throws, in DistilX's words, on a failure inside a successful answer:
- a column in `columns[<table>]` with an `error`, whose values are left out of the rows;
- a warning starting `computation outlasted`, the timeout. Other warnings are not failures.

A loader never uses `call`. The dashboard declares no `dash.calc`: a total or a share is an endpoint column.

## Docs and links

- Read docs and links for tracing and the provenance line with `distilx_list_endpoints`, `distilx_list_controls` and `distilx_get_tables`, with `branch` set to the branch the dashboard reads, while building, and write them into the page. Read them again on every republish.
- A DistilX `source` or `retrieved` value is shown with the values it backs, never as a marked value.

## Access banner

- On a private project, except on a private branch, the page holds the `share_link` and `expires_at` from `distilx_share_project`.
- When a DistilX source fails with a message containing `access_denied: `, the page shows one banner naming the project.
- With a `share_link` before its `expires_at`, the banner offers the link with its expiry, to open and accept in DistilX and then reload. Otherwise it says to ask the author.

## Hand-over

Tell the user that each reader's calls run with their own DistilX access, that readers without project access see the access banner, and that docs and links change only when you republish.
