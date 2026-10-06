# 21st Codex plugin

Find and install React components, build project-aware UI, and make editable
product videos with the 21st MCP server. The video workflow uses the coding
agent's own reasoning and the same project operations as the web editor.

## Local testing

The monorepo marketplace is `.agents/plugins/marketplace.json` at the repo
root. It points to this directory, `distribution/codex-plugin`, so there is
one canonical package instead of a duplicate under `packages/`.

1. Run `pnpm skills:build`, `pnpm skills:check`, and `pnpm plugin:check` from
   the monorepo root.
2. Register and install a cached package with the CLI:

   ```bash
   codex plugin marketplace add ./distribution/codex-plugin
   codex plugin add 21st@21st
   ```

   The repo marketplace can also appear in the desktop Plugins Directory as
   **21st Repository Plugins**. Refresh the desktop app's plugin discovery
   when needed; a fresh CLI session reads the installed package independently.
3. Connect the 21st account when prompted. The entry requests authentication
   on install; the endpoint advertises OAuth discovery.
4. In a new chat, ask: "Make a launch video for this repo and open the
   editable project beside the chat."
5. Verify that the editor opens, reflects a tool edit, preserves a manual
   edit, and resolves the selected layer. Then check a variant, a snapshot,
   and an export.

Repo marketplace paths resolve from the repo root. Codex installs a cached
copy, so reinstall the package after source changes and test in a fresh chat
or CLI session. This does not establish that an already-running desktop chat
has reloaded its tools. See the [official plugin packaging documentation](https://developers.openai.com/plugins/build/plugins#install-a-local-plugin-manually).
The offline checks validate packaging; they do not establish successful
installation, OAuth, editor iframe behavior, or rendering in the actual host.

## Standalone marketplace

The dedicated public repository mirrors this directory when explicitly
released. Its `.agents/plugins/marketplace.json` points to `./`. Add the
marketplace through the CLI, then install and test in the desktop app:

```bash
codex plugin marketplace add 21st-dev/codex-plugin
codex plugin add 21st@21st
```

For a local package-only source, use
`codex plugin marketplace add ./distribution/codex-plugin` from the monorepo.
See [Add a marketplace from the CLI](https://developers.openai.com/plugins/build/plugins#add-a-marketplace-from-the-cli).
No public submission is performed by this package or its build/check commands.

## Auth and editor handoff

The portable `mcp.json` connects to `https://21st.dev/api/mcp` using
`streamable-http` without embedding credentials. The repo marketplace uses
`policy.authentication: ON_INSTALL`. Account authorization is enforced by
the server. A created project's editor URL contains a short-lived, single-use
handoff, allowing the separate editor browser to edit and export the same
project. The browser session is scoped to that project; account actions such
as sharing or creating another project still require Clerk sign-in.

The compatibility `.mcp.json` retains API-key support for older loaders. It
reads `API_KEY_21ST` and sends it as a bearer token. Current portable loaders
use `mcp.json` and OAuth. A manual API-key MCP server can use this Codex config:

```toml
[mcp_servers.21st]
url = "https://21st.dev/api/mcp"
bearer_token_env_var = "API_KEY_21ST"
```

OAuth discovery and a local install still require a hands-on host check.
The marketplace authentication policy alone does not prove the OAuth flow.

For a standalone CLI login, use the bundled server's name and endpoint:

```bash
codex -c 'mcp_servers.21st.url="https://21st.dev/api/mcp"' mcp login 21st --oauth-client-registration auto
```

This command-specific server override leaves the stored server configuration
unchanged; successful login stores OAuth credentials. `--no-browser` supports
manual authorization and callback entry. Pre-registered clients must use the
exact callback Codex displays, as described in [OAuth client registration and callbacks](https://learn.chatgpt.com/docs/extend/mcp#oauth-client-registration-and-callbacks).

## Bundled workflows

The shared source pack contains eight skills: `21st-cli-use`, `21st-ai`,
`21st-registry`, `21st-design-sync`, `21st-ui-build`, `21st-ui-explore`,
`21st-ui-review`, and `21st-video`. `21st-cli-use` retains the existing
component search/install flow. `21st-video` discovers video blocks, creates a
project, opens the editor, applies revision-aware ops, uploads real assets,
checks frames, proposes variants, and exports.

Video authoring uses `video_list_blocks`, `video_search_blocks`,
`video_create_project`, `video_get_doc`, `video_apply_ops`,
`video_propose_variants`, `video_upload_asset`, `video_finalize_asset`, `video_snapshot`,
`video_export`, and `video_export_status` when offered by the connected
server. Each tool works without a widget; the real editor opens in the
in-app browser. Tool availability is checked before starting a workflow.

## Layout

```text
plugin.json                       # portable identity + OpenAI presentation
mcp.json                          # portable streamable-http MCP server
.agents/plugins/marketplace.json  # standalone local marketplace
.codex-plugin/plugin.json         # legacy compatibility fallback
.mcp.json                         # legacy API-key connection
marketplace.json                  # compatibility copy of the marketplace
assets/icon.svg                   # current /logo-icon.svg artwork
skills/                           # generated from distribution/source
```

The root manifest uses the Agent Plugins schema and `extensions.com.openai`.
The compatibility overlay is retained for older loaders; its metadata stays
in sync, but hosts use the root extension when present. See
[Plugin structure and metadata](https://developers.openai.com/plugins/build/plugins#plugin-structure).
Screenshots will be added after the real host workflow has been captured.
Local rendering, fullscreen MCP Apps UI, and public submission remain future
work.
