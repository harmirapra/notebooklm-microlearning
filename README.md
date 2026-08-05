# NotebookLM Microlearning

A Claude Code plugin that turns any topic into a NotebookLM audio
or slide podcast for microlearning — a structured source document plus,
when connected, automatic creation in your own NotebookLM notebook.

> **Prerequisite:** you need an **existing Google account with access to
> NotebookLM** (notebooklm.google.com). This plugin does not create that
> account for you — it only signs in to it, on your machine, in your own
> browser.

## What it does

1. You name a topic (e.g. "git and github, basic use cases").
2. The skill writes a source document: each concept explained, demonstrated
   with an example, given as an exact command/step, and re-explained in
   plain language — as one end-to-end walkthrough.
3. If the bundled MCP server is connected and signed in, it adds that
   document to a single, persistent NotebookLM notebook you name once
   (never creates a new one per topic) and triggers an audio or slide
   podcast, scoped to just that new source.
4. If the MCP server isn't set up yet, you still get the source document
   and a ready-to-paste NotebookLM prompt — nothing is blocked on setup.
5. **Want it in a different format afterwards?** Just ask — "make that a
   video instead", "also give me the audio version". The skill reuses the
   existing source rather than rewriting it, so one topic can have
   several formats without re-authoring anything.

## Install

```bash
claude plugin marketplace add harmirapra/notebooklm-microlearning
claude plugin install notebooklm-microlearning
```

> **What "marketplace" means here:** nothing is submitted to, listed in,
> or approved by any public catalog. Claude Code has no central plugin
> registry. `marketplace add` simply tells your Claude Code to read
> plugins from this GitHub repo — the repo *is* the marketplace, by
> virtue of containing `.claude-plugin/marketplace.json`. The first
> command registers the source; the second installs the plugin from it.

On first enable, you'll be prompted for five settings (default language,
explanation style, target notebook name, how technical explanations
should be, and whether to wait for generation to finish — see
`.claude-plugin/plugin.json` under `userConfig` for exact wording).
Change them any time via `/plugin configure`.

## MCP setup (required for automatic notebook creation)

The plugin bundles the `gemini-notebook-mcp` server (via
[`notebooklm-mcp-cli`](https://github.com/jacob-bd/gemini-notebook-mcp-cli),
run through `uvx` — no separate server install). You still need:

1. **Install `uv`:**
   ```bash
   brew install uv        # macOS
   # or see https://docs.astral.sh/uv/getting-started/installation/
   ```
2. **Sign in once** (opens a real browser window — you log in to your own
   Google account there; no password is ever shared with this plugin or
   with Claude):
   ```bash
   uvx --from notebooklm-mcp-cli nlm login
   ```
3. **Restart Claude Code** (or run `/mcp`) so it picks up the
   connection.

Without this, the skill still works — it just returns a document and a
prompt for you to paste into NotebookLM manually instead of doing it for
you.

## Known limitations

- **Unofficial integration.** Google does not publish a public NotebookLM
  API. `notebooklm-mcp-cli` works by driving NotebookLM's internal, web
  APIs — undocumented and subject to change without notice. If NotebookLM
  changes its interface, this plugin may need an update to
  `notebooklm-mcp-cli` before it works again. This is an inconvenience,
  not a security issue: your credentials are never exposed to this
  plugin, only a browser session cookie cached locally by the MCP server.
- **Whether NotebookLM's own completion notification fires for
  MCP-triggered generation is unconfirmed.** Manually starting a podcast
  in the NotebookLM web UI does notify you when it's ready. Generation
  triggered through this plugin's MCP server goes through an
  undocumented internal API, and it's not yet verified whether that
  triggers the same notification. Until confirmed, don't rely on it —
  check the notebook directly, or enable `wait_for_completion` if you
  want Claude to confirm completion in-session instead.
- **Source-scoping is not a documented guarantee.** Generating a podcast
  scoped to only the newly-added source (so unrelated past topics in the
  same notebook don't get blended together) depends on an
  undocumented parameter of the underlying tool. The skill checks the
  live tool schema before relying on it and tells you explicitly if it
  can't confirm the behavior — it will not silently generate from every
  source in your notebook at once.

**Note:** the install commands above were verified against Claude Code's
own CLI on 2026-08-06.

## Editor support

**Claude Code only.** Codex is not supported. This plugin's five settings
rely on the `userConfig` manifest field, which Claude Code implements but
Codex's plugin format does not — the skill would have nothing to fill its
configuration placeholders with. If that changes, Codex support can come
back.

## License

MIT — see [LICENSE](LICENSE).

## Acknowledgments

Built on [`notebooklm-mcp-cli`](https://github.com/jacob-bd/gemini-notebook-mcp-cli)
by Jacob Ben-David (MIT license), an unofficial, community-maintained
integration with Google NotebookLM. This project is not affiliated with
or endorsed by Google or Anthropic.
