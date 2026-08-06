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

## Ideas beyond "explain topic X to me"

The obvious use is learning something you're already half-familiar with.
These are less obvious, and all of them are just *things you ask for* —
none of it is special built-in automation.

- **Explain your own project to other people.** Point it at a repo's
  README and architecture notes and ask for a slide deck. You get an
  onboarding walkthrough for colleagues without writing slides.
  This README's own plugin was documented exactly this way.
- **Turn a release into a "what changed" episode.** After cutting a
  version, ask for a podcast covering the diff or changelog. Reviewers who
  won't read a diff will listen to five minutes on the way home. Works
  well as a step in a release checklist: tag → changelog → podcast.
- **Re-record when the source goes stale.** Generated artifacts are
  snapshots — nothing updates them when the underlying thing changes.
  This plugin's own first podcast claimed Codex support one day after
  Codex support was removed. The fix is to replace the source, regenerate,
  and **delete the outdated artifacts** so nobody learns the wrong version
  from a file that still looks current.
- **Give one topic several formats.** Ask for a video after the slide
  deck; the skill reuses the same source instead of re-authoring it.

**One important caveat if you do this with real content:** the skill's
default behavior is to *write* a source document about a topic. For
release notes, or anything where accuracy against a specific artifact
matters, give it the actual material — the changelog, the diff, the
commit messages — and say to work from that. Otherwise it writes from
general knowledge about your topic, which is exactly what you don't want
for a factual "here's what changed" summary.

## Known limitations

- **Unofficial integration.** Google does not publish a public NotebookLM
  API. `notebooklm-mcp-cli` works by driving NotebookLM's internal, web
  APIs — undocumented and subject to change without notice. If NotebookLM
  changes its interface, this plugin may need an update to
  `notebooklm-mcp-cli` before it works again. This is an inconvenience,
  not a security issue: your credentials are never exposed to this
  plugin, only a browser session cookie cached locally by the MCP server.
- ~~Whether NotebookLM's own completion notification fires for
  MCP-triggered generation is unconfirmed.~~ **Confirmed working
  (2026-08-06).** Generation started through this plugin's MCP server
  does trigger NotebookLM's normal completion notification — a push
  notification on the NotebookLM mobile app, same as when you start a
  podcast by hand in the web UI. So `wait_for_completion` is genuinely
  optional: you can start a generation, close the session, and let your
  phone tell you when it's done. Caveat: this is one confirmed
  observation on one account, and it rides on the same undocumented
  internal API as everything else here — if notifications ever stop
  arriving, that is a plausible thing to have broken.
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
