---
name: live-artifacts
description: Rules for any artifact showing DistilX data through the DistilX connector, a Dashboard of Claude's Dashboard type or a page with the `mcp` capability: planning it, uploading its data, writing its code, sharing it and checking the DistilX agent's branch for it. Use when the user asks for, plans, changes or shares an artifact, dashboard, report page or app backed by a DistilX project or by data they want uploaded to one, even before anything is built.
allowed-tools: [mcp__plugin_distilx_DistilX__distilx_ping, mcp__plugin_distilx_DistilX__distilx_docs, mcp__plugin_distilx_DistilX__distilx_list_projects, mcp__plugin_distilx_DistilX__distilx_search_projects, mcp__plugin_distilx_DistilX__distilx_get_project, mcp__plugin_distilx_DistilX__distilx_show, mcp__plugin_distilx_DistilX__distilx_get_tables, mcp__plugin_distilx_DistilX__distilx_get_sheet, mcp__plugin_distilx_DistilX__distilx_search_dsl, mcp__plugin_distilx_DistilX__distilx_list_endpoints, mcp__plugin_distilx_DistilX__distilx_call_endpoint, mcp__plugin_distilx_DistilX__distilx_list_sources, mcp__plugin_distilx_DistilX__distilx_list_datasets, mcp__plugin_distilx_DistilX__distilx_get_dataset, mcp__plugin_distilx_DistilX__distilx_list_project_files, mcp__plugin_distilx_DistilX__distilx_get_project_file, mcp__plugin_distilx_DistilX__distilx_list_export_destinations, mcp__plugin_distilx_DistilX__distilx_list_exported_tables, mcp__plugin_distilx_DistilX__distilx_get_exported_table, mcp__plugin_distilx_DistilX__distilx_list_pipelines, mcp__plugin_distilx_DistilX__distilx_list_pipeline_runs, mcp__plugin_distilx_DistilX__distilx_list_branches, mcp__plugin_distilx_DistilX__distilx_get_branch_history, mcp__plugin_distilx_DistilX__distilx_calculate, mcp__plugin_distilx_DistilX__distilx_list_controls, mcp__plugin_distilx_DistilX__distilx_check_project, mcp__plugin_distilx_DistilX__distilx_create_project, mcp__plugin_distilx_DistilX__distilx_upload_project_file, mcp__plugin_distilx_DistilX__distilx_share_project, mcp__plugin_distilx_DistilX__distilx_send_message, mcp__plugin_distilx_DistilX__distilx_wait_turn, mcp__plugin_distilx_DistilX__distilx_stop_turn, mcp__plugin_distilx_DistilX__distilx_get_changes, mcp__plugin_distilx_DistilX__distilx_merge_branch, mcp__plugin_distilx_DistilX__distilx_fork_branch, mcp__plugin_distilx_DistilX__distilx_rename_branch, mcp__plugin_distilx_DistilX__distilx_create_pipeline]
---

# Live artifacts on DistilX

These rules hold for any artifact showing DistilX data. Once you have chosen what to build, also read:
- `references/dashboard.md` for a Dashboard, an artifact of Claude's Dashboard type;
- `references/page.md` for a page, an artifact with the `mcp` capability.

They keep an artifact verifiable: every value comes from saved project logic, its sources are recorded, and the viewer can trace each value back to the project. Rules marked as conventions are defaults you may change with the user, saying why. Other rules are fixed, apart from the parts the user may skip.

## Data

- Every number, data point and control value in the artifact, pick-list options and defaults included, comes from a `distilx_call_endpoint` response.
- The artifact never calls `distilx_calculate`. Use it only in the conversation, to explore or check; an exploring call over the scratch limit is a check and may be split.
- Send logic found while exploring to the DistilX agent as intent and decisions, marked as decided by the user or assumed by you, never as code.
- A table over `max_rows` (at most 1000) is aggregated or paged by a request input.
- Convention: ask for the fewest endpoints. Data that takes the same request and is shown together comes from one endpoint, for example one options endpoint for every pick list. Optional request tables let one endpoint serve requests that differ only by optional inputs.
- Convention: the options response marks each pick list's default in a boolean `selected` column, which arrives as the text `"1"` or `"0"` (`1` or `0` once a loader turns it into a number), and returns a one-row defaults table for free inputs. One defaults table in the project sets these and the endpoint sheet's control defaults.
- Convention: a filter where several values can be picked reads an optional one-column request table of its own. Ask the agent to read it in the response as a set, never with `FIRST`, where no rows or an NA value means every value. The artifact passes the picked values as a list in that field where `request_schema` allows an array.
- The DistilX agent doesn't know `!api_optional`: when asking for an optional input or request table, name the decorator, and check `optional` in `distilx_list_endpoints` on the branch before the artifact relies on it.

## Artifact code

Artifact code is a page's scripts, or a Dashboard's page and loaders.

- Call with `raw: true` and set `max_rows`.
- A DOUBLE column whose `format` is `DATE` holds day numbers counted from 1899-12-30.
- Allowed: format numbers, dates from day numbers, currencies and percentages; map rows to axes, series and cells; look up a row by key; pick the default row, such as the one whose `selected` is `"1"` or `1`; choose which columns to show; show truncation. A chart library's own axis scaling is fine.
- Never: sort, filter, group, sum, count, take a minimum or maximum, compute with values, or set a default no response gave.
- A value the artifact should show that no response returns, such as a grand total, is asked of the agent as an endpoint column, never computed in the artifact.

## Sources

- Upload nothing while where the data came from is open.
- Data you upload with `distilx_upload_project_file` records where it came from and when it was retrieved: `source` and `retrieved` columns in the file, or a sources table the agent adds to the project, whichever fits. A file from one source records it once.
- An upload belongs to the branch it went into: upload on the branch you build on, or without `branch` and then build on the branch it returns. On a project you created empty in this chat, upload with its created `branch.id`.
- A replace on the branch an artifact reads refreshes the artifact at once, with no agent turn; when that branch is the default branch, the `distilx` skill's "Replacing files" rules apply.
- Never invent a source or a retrieval date: ask where data the user supplied came from and when it was retrieved.
- Endpoints return the sources with the data. The artifact shows each source and its retrieval date with the values it backs, in their tooltip unless the user asks for another place or the artifact fits one better, and never as a value it charts, counts or filters.
- A value combining several sources links the sheet that lists them. Imported data names its import and dataset, from `distilx_list_sources`.

## Endpoint sheet

- Each endpoint the artifact calls belongs to one sheet: the sheet of the control table that sets its request tables' sample rows, as `distilx_get_tables` and `distilx_get_project` show. The sheet also has tables or charts showing each response table's columns unchanged.
- A control table is never itself a request table, and its defaults equal the artifact's. No pick list's options depend on its own selection.
- Convention: endpoints built together share the sheet, named for the artifact that first needed them. Later artifacts reuse it.

## Tracing

- Every control, chart, number and table says what it shows: column docs from `distilx_list_endpoints`, table docs from `distilx_get_tables`.
- It links what it shows: a control its `link` from `distilx_list_controls`; a value its response column, made from the response table's `link` from `distilx_list_endpoints` plus `&column=` and `encodeURIComponent` of the column name; a visual of several columns its response table. Build no other URL.
- With the endpoint sheet skipped there is no control: a pick list links its options response column, a free input its request column, made the same way from the request table's `link`.
- Something without a `link` is named, not linked.
- Only when the user asks, also link the calculation tables behind it, with their `link` from `distilx_get_tables`, as the agent named them or the project shows them.
- Read docs and links with `branch` set to the branch the artifact reads.
- Tooltips carry this text and these links by default; another way that gives the same text and links, such as a details panel, is fine when the user asks or the artifact fits it better.

## Building

1. **Open choices.** Create, upload or message nothing while a material choice is open. Decide what to ask and when, and never ask what the user already said; naming the project the data comes from does not say where to build. Open choices include:
   - where to build: the project, a private branch of it (its endpoints and links work only for the user, and its imports refresh only when a pipeline names it), a clone (a private copy whose changes never reach the original) or a new project for data you collect;
   - whether to have the agent add a missing endpoint sheet, docs or sources, or skip them;
   - where and when data the user supplied came from.

   When `sharing.permissions` from `distilx_get_project` lacks `WRITE`, the user can't change the project's shared branches: a private branch serves an artifact only they use, a clone one other viewers use. When existing endpoints, sheet, docs and sources cover the artifact, nothing in the project changes, so where to build is not open.
2. Ask the DistilX agent, on a branch, for the endpoints, their docs, the sources and the endpoint sheet, and, when the user wants calculation links, for the tables behind each endpoint.
3. When the turn is `done`, check it on that branch with `branch` on `distilx_list_endpoints`, `distilx_get_tables` and `distilx_call_endpoint` against these rules, then give the branch `link` for the user to review. On a project you created empty in this chat, build on its default branch, as the `distilx` skill's step 3 says: the artifact reads the default branch, fixes before publishing go there, and the agent's work there has nothing to merge. Otherwise merge with `distilx_merge_branch` when the user says to, or let them merge in the DistilX UI.
4. Before publishing, settle which branch the artifact reads: the default branch unless the user chooses another, shared or private, whose id every artifact call and the `distilx_show` call then pass as `branch`. An artifact reading the default branch waits for the merge, and so does a fix to it, except on a project you created empty in this chat, as step 3 says. Before reading another branch, tell the user what it means:
   - a shared branch changes whenever a turn or a merge lands on it, and a private branch works only for its owner;
   - neither gets later changes to its source, and its imports refresh only when a pipeline names it;
   - deleting it breaks the artifact.

   Point to the project with `distilx_show`, passing the endpoint sheet; links in the artifact stay plain links.
5. Right before publishing a page or writing a Dashboard, run on the branch the artifact reads each distinct artifact call once with the defaults, compare each endpoint-sheet control's default with the options response's defaults, and run `distilx_check_project` with the `controls` check, `max_controls` covering the endpoint sheet's controls. A control's default is its `value` from `distilx_list_controls`, or, when `value` is null, its `value_expression` evaluated with `distilx_calculate` on that branch. Fix a failure through the agent on a branch before publishing. On a private project, then share as Sharing says: the artifact holds `share_link` and `expires_at` in its join or access banner.

An artifact that still calls `distilx_calculate` is rebuilt on endpoints once the open choices are settled, never republished as is.

## Skipping

- The user may explicitly skip the endpoint sheet, the docs, the tracing or the sources of uploaded data; build the rest. Never skip a part on your own: raise missing docs, sheet or sources as an open choice.
- With the docs skipped, the tracing gives only links and sources, never text you wrote.
- Endpoints only and the artifact-code rules are never skipped.

## Connector

- Use the claude.ai DistilX connector of this session, never the plugin's server or a `host:` server. Every artifact call names it <!-- connector -->`DistilX`<!-- /connector -->, unless a page's publish result states another display name; then that page uses that one. Never derive the name from the tool names.
- Without a claude.ai DistilX connector in the session, stop and tell the user.
- No artifact calls a tool that writes or drives the DistilX agent, except a page's `distilx_accept_invitation` with a `share_link` from `distilx_share_project`.

## Disclaimer

Every artifact shows a provenance line with the project `link`, the endpoint sheets' `link`, both from the branch the artifact reads, that branch's name as `distilx_list_branches` shows it at publish when it isn't the default, and each skipped part; never claim the logic was reviewed.

## Sharing

A project in the public bucket needs no share: the artifact shows its `link` and holds no `share_link`.

An artifact on a private branch calls no share tool and holds no `share_link`: only its owner can open the branch.

On a private project, call `distilx_share_project` in the conversation before every publish, a republish included, unless the user said not to share or the artifact reads a private branch. Take `share_link` and `expires_at` from its result, never from the artifact's source, and put them in the artifact: a page shows them in its join banner and declares `distilx_accept_invitation`, a Dashboard in its access banner offering the link with its expiry. The reference has the details. Relay to the user:
- that the link reaches only the people they share the artifact with;
- what the tool description says joining grants, agent conversations included;
- the expiry (`reused` links keep their own date);
- the inputs `warnings` say were left out: viewers cannot compute without them;
- that each viewer who joins appears in the user's shared-by-me list and keeps access after the link expires until the user revokes it in the DistilX UI;
- for a project you created empty in this chat, that it is now shared, so later changes go to a new branch and reach the artifact after a merge.

When the share fails, for example without `SHARE` on the project, publish without a `share_link`, dropping any earlier link, and tell the user why. Viewers without access are then told to ask the author.
