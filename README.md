# 21st — Codex plugin

Wires up the 21st MCP server and the 21st.dev skills in Codex, so the agent
searches, installs and publishes on 21st.dev from the terminal or via MCP,
instead of hand-writing UI.

> The official Codex plugin directory is **not open yet**, so this plugin is
> **self-hosted only**: add the marketplace by its git repo / URL directly, then
> install through the in-CLI `/plugins` browser.

## What's inside

- **MCP server `21st`** — the remote endpoint `https://21st.dev/api/mcp`,
  exposing 21 tools: `search`, `get_component`, `get_theme`, bookmarks + lists,
  teams, `get_usage`, `generate`, and edit/delete for components/themes/
  templates. Metadata search is free; component code, generation and writes
  are metered.
- **Skills** — seven focused skills that teach the `21st` CLI, auto-activating
  when the project has a `components.json`: `21st-cli-use` (search/get/add),
  `21st-ai` (generate/iterate/grab-code with 21st AI), `21st-registry` (publish
  & manage), and `21st-design-sync` (publish the project's design tokens as a
  theme), plus project-aware Build, Explore, and Review workflows.

## Install

1. Set your 21st API key (get one at https://21st.dev/settings/api-keys):

   ```bash
   export API_KEY_21ST="sk_..."
   ```

2. Add this self-hosted marketplace:

   ```bash
   codex plugin marketplace add 21st-dev/codex-plugin
   ```

   (Replace `21st-dev/codex-plugin` with the actual `<org>/<repo>`, git URL, or
   local path that hosts this directory.)

   Refresh the Git-backed marketplace later with:

   ```bash
   codex plugin marketplace upgrade 21st
   ```

3. Open the in-CLI plugin browser and install `21st`:

   ```
   /plugins
   ```

   Find `21st` in the browser and install it. This registers the skill and the
   MCP server from this plugin.

## Auth

The MCP server is configured (`.mcp.json`) to read your key from the
`API_KEY_21ST` environment variable and send it as a bearer token. The
equivalent Codex `config.toml` block is:

```toml
[mcp_servers.21st]
url = "https://21st.dev/api/mcp"
bearer_token_env_var = "API_KEY_21ST"
```

If tool calls report "Not authenticated," confirm `API_KEY_21ST` is exported in
the shell that launched Codex.

## Layout

```
.codex-plugin/
  plugin.json     # skills + mcp references
.mcp.json         # remote 21st MCP (bearer via API_KEY_21ST env)
.agents/plugins/
  marketplace.json  # Codex marketplace manifest
marketplace.json    # legacy marketplace manifest
skills/
  21st-cli-use/
    SKILL.md       # bundled skills (shared with the Claude Code plugin
  21st-ai/         # and https://21st.dev/api/skills/<name>)
    SKILL.md
  21st-registry/
    SKILL.md
  21st-design-sync/
    SKILL.md
```
