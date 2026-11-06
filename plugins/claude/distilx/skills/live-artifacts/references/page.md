# Pages

Rules for a page with the `mcp` capability showing DistilX data, on top of `SKILL.md`.

## Manifest

- Declare only `distilx_call_endpoint`, plus `distilx_accept_invitation` when the page holds a `share_link` from `distilx_share_project`.
- On a private project, except on a private branch, put `share_link` and `expires_at` in the page with the join banner.
- When a share fails, drop any earlier banner and `distilx_accept_invitation` with the link.

## Connector problems

On `server_not_connected`, name the connector to add; on `selection_required`, say to choose which DistilX connector to use; on `needs_reauth`, say to reconnect it. This holds for the accept call too, and none of these offers the join button.

## Access denied

A denial is a `tool_error` whose `message`, or a text block in its `result`, contains `access_denied: `.

- **Without a `share_link`:** say to ask the author.
- **With a `share_link`:** one banner at the top of the page, and each denied section points to it. The banner names the project, says the project's owner will see who the viewer is (name or email) and that access lasts until the owner revokes it, and offers a join button. Never accept without that click.
- **Join:** the click disables the button and makes one `callTool` of `distilx_accept_invitation` with `project` and with `invitation` set to the `share_link`. On success call `invalidate` for the connector, ignoring a rejection, which re-runs watched sections, then call the other denied sections again, once.
- **Join outcome:**
  - still denied after a successful join, or a `tool_error` from the accept: say to ask the author, show the tool's message as text, and offer no button again until reload;
  - `blocked_by_policy`, `approval_required`, `not_in_manifest` or `consent_required`: show the share link with its expiry;
  - `server_unavailable`, `upstream_error` or another code: reload the denied sections once and keep the button for another click if they are still denied, never retrying by itself.
- **Expiry:** once `expires_at` has passed, no button; say to ask the author. A null `expires_at` shows the button without an expiry.
