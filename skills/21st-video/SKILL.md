---
name: 21st-video
description: >-
  Create and edit a product, launch, demo, or social video from the current
  project with 21st video MCP tools. Open the editable project alongside the
  chat, apply small revision-aware edits, check rendered frames, offer visual
  variants, and export the result. Use when the user asks for a video, says
  "make a launch video for this repo", "edit this video", or points to a
  selected layer in the 21st video editor. Requires the 21st MCP server.
---

# 21st Video

Create the video with the user's coding agent and the 21st video tools. Keep
the real editor open so the user can see changes and edit the same project by
hand. Use the available tool input schemas as the source of truth for fields,
operation names, limits, and supported formats.

## Start from the product

Read the project's README, relevant screens/components, brand assets, and
release notes. Use real product copy and assets. Establish one message, an
appropriate aspect ratio, and a short scene sequence from the user's request.
Keep claims grounded in the product. Respect project instructions and any
restriction on browser or computer use. Reuse the user's running app; start a
server only when authorized.

Inspect the available `video_*` tools before starting. If they are absent,
report that this connection does not offer video authoring. Use the existing
`21st-cli-use` skill for finding and installing React components; video blocks
use the separate video catalog.

## Create and open the project

1. Call `video_list_blocks` or `video_search_blocks` for relevant blocks. Read
   each chosen block's supported sizes and props schema before composing it.
2. Call `video_create_project` with a title and `size` (`16:9`, `9:16`, or
   `1:1`). Choose at most one of `template` (a slug), `blockRef` (one block
   with optional `props`), or `blocks` (initial kit ops); omit all for empty.
   Retain the returned `projectId`, `rev`, and `editorUrl`.
3. Immediately open the returned `editorUrl` in the host's in-app browser. In
   Codex, use the native browser-opening tool, such as `open_in_codex` with a
   browser URL, when available. The URL already selects Codex host mode and
   includes a short-lived, single-use handoff. Open it intact and promptly;
   never print the handoff token or persist that URL to a repo or log. For
   final links, remove the handoff fragment from the project's editor URL. If the
   host cannot open the editor, give the stable link and continue with tools.
4. Call `video_get_doc` for the actual document, current revision, and
   `editorContext` containing the user's current selection/frame/view size.

## Edit in small batches

Apply a few related operations at a time with `video_apply_ops`, supplying
`projectId`, the latest `baseRev`, `ops`, and a short `label` that explains the
user-visible change. Use the shared operation format returned by the tool.
Use block schemas for props and identifiers returned by the document/catalog;
do not invent layer IDs, block refs, or unsupported props.

Inspect `applied`, `kept`, and `errors` after every edit. A batch can keep
hand-edited fields or reject some ops; success does not mean every requested
change applied. Preserve the user's `mine` fields. Use `force` only for the
specific `id.key` fields the user explicitly asked to overwrite.

After a successful edit, retain the new revision and changed IDs. If the tool
reports a revision conflict, inspect its fresh document or call
`video_get_doc`, reconcile with the user's edits, and construct a new batch.
Never replay a stale batch blindly or replace the whole document to bypass a
conflict. Re-read before each later edit, since the user may be editing too.

For "this one", "the selected layer", or a page comment, call `video_get_doc`
and resolve the target from the current selection/frame or the comment's
`data-layer-id`. Use that stable ID for the edit. If no unambiguous target is
available, ask for the target while continuing independent work.

## Add real assets

Use `video_upload_asset` with `projectId`, `kind`, `mime`, and the file's
`bytes` for local logos, screenshots, or footage. PUT the exact file bytes to
the returned `uploadUrl` with its `contentType`. Call `video_finalize_asset`
with `projectId`, `baseRev`, and `assetId`, plus known dimensions or duration
when applicable. It verifies and attaches the uploaded asset, returning a new
revision. On conflict, retry finalization with the fresh revision; no second
upload is needed. Compose with that finalized asset through
`video_apply_ops`. Keep presigned URLs out of chat and logs. Capture a running
app only when browser/computer use is allowed by the user and project
instructions.

## Check the video and offer variants

Call `video_snapshot` with `projectId`, the current `rev`, and a `frame` for
representative frames: the settled frame of each scene, the main product
proof, and any changed or potentially clipped layer.
Inspect the returned image when an image viewing tool is available. Check
readability, contrast, crop, spacing, and timing. Fix issues through
`video_apply_ops`; do not claim a visual check from a tool's success alone.
Snapshots cover individual frames. Review playback separately when available
and do not treat snapshots as full motion or audio verification.

When a visual choice would help, call `video_propose_variants` with
`projectId`, `baseRev`, a `label`, and 2 to 4 `variants` containing distinct
`title`, optional `note`, and `ops` sets. Inspect returned validation details;
invalid options are dropped. The editor shows a variant tray where the user
can choose. Proposing does not apply a variant.
Wait for the user's pick, then read the current doc/revision before making
further edits; do not silently pick a proposed variant for them.

## Export and deliver

Once the requested video is ready, call `video_export` with `projectId`, the
current `rev`, target `sizes`, and `quality` (`720p` or `1080p`). If the
service requires sign-in or the export quota is exhausted,
report the returned requirement and preserve the editable project. Never
invoke hosted AI generation to work around a video tool or quota failure.

Poll `video_export_status` with `projectId` and reasonable spacing, tracking
the jobs returned by `video_export`, until they complete or report failure.
Use the returned URL only when the export is done. Report the editable
project link, export link, and any
remaining visual/playback limitation briefly. Export renders the video; it
does not publish it to social media.

Local CLI rendering, a fullscreen MCP Apps editor, and public plugin
submission are future options. Do not promise or invoke them as available.
