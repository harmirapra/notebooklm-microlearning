---
name: microlearning-podcast
description: >
  Turns a topic into a NotebookLM audio or slide podcast for microlearning
  — a source document plus, if the bundled MCP server is connected, the
  actual notebook source and generated artifact. Use when the user says
  "turn this into a podcast", "make me a NotebookLM podcast on X",
  "microlearning on X", "ukotvi mi X" (Czech), or names a topic they want
  to reinforce/learn via audio or slides on the go. Do NOT use this as
  preparation before a first lesson on a topic — it's for reinforcing
  something already partly known, not introducing it.
---

# Microlearning Podcast

## Prerequisite

This skill assumes you already have a **Google account with access to
NotebookLM** (notebooklm.google.com). It does not create that account —
it only signs in to it. See "MCP setup" below for how the sign-in works.

## Role

Turn a topic into a structured learning source, then (if possible) feed
it directly into a persistent NotebookLM notebook and generate an audio
or slide podcast from it — no manual copy-pasting.

## Configuration

Read from plugin `userConfig` (set when this plugin was enabled — the
user can change these any time via `/plugin configure`):

- **Language:** `${user_config.default_language}`
- **Explanation style:** `${user_config.explanation_style}`
- **Target notebook:** `${user_config.notebook_name}`
- **Audience calibration:** `${user_config.expertise_level}`

If a file at `2_Context/identity/about-me.md` exists relative to the
current working directory (i.e. the user happens to also have a PACT
workspace), read it as **additional, optional** calibration context on
top of `expertise_level` — never as a requirement. Its absence is normal
and not an error.

## Workflow

1. **Clarify scope** — if the topic is vague (e.g. just "git"), narrow it
   to concrete use cases with one question. Don't guess.
2. **Verify facts and commands** — no guessed/uncertain syntax. If unsure
   of exact syntax, verify it (documentation, existing usage) rather than
   assuming.
3. **Write the source document** in `${user_config.default_language}`,
   following `${user_config.explanation_style}`, as an end-to-end
   walkthrough (concepts in the order they'd actually be used, not an
   alphabetical list).
4. **Check MCP availability.** The plugin bundles an MCP server
   (`gemini-notebook-mcp`, via the community `notebooklm-mcp-cli`
   project). If it isn't connected or isn't authenticated:
   - Tell the user explicitly (don't fail silently).
   - Point them to "MCP setup" below.
   - Return the finished source document plus a short prompt they can
     paste into NotebookLM manually, and stop there.
5. **If MCP is available and authenticated:**
   - **Never create a new notebook.** Find the notebook named
     `${user_config.notebook_name}` via `notebook_list`. If it doesn't
     exist, tell the user to create it once themselves at
     notebooklm.google.com, rather than creating a substitute.
   - Add the source document to that notebook (`source_add`).
   - Before generating, **check the live tool schema** for `studio_create`
     — do not assume from this file that a source-scoping parameter
     (e.g. `source_ids`) exists or behaves a certain way. This connects to
     an **unofficial, undocumented internal API** (Google has no public
     NotebookLM API) and can change without notice. If scoping to a
     single source isn't confirmed available, do not generate from all
     sources in the notebook at once — that would blend unrelated topics
     together. Tell the user and let them pick the source manually in the
     Studio UI instead.
   - Ask (or infer from context) whether the user wants an audio podcast
     or a slide deck, then trigger generation scoped to just the new
     source.
6. **Return everything together** — the source document, the notebook
   link (if step 5 ran), and a clear statement of which parts happened
   automatically vs. need a manual step.

## MCP setup

This skill needs the `gemini-notebook-mcp` MCP server (bundled via
`.mcp.json` in this plugin — it runs through `uvx`, no separate install
step for the server itself). You still need:

1. **`uv` installed** — e.g. `brew install uv` (macOS) or see
   [docs.astral.sh/uv](https://docs.astral.sh/uv/getting-started/installation/).
2. **Signed in once**, in a terminal: `uvx --from notebooklm-mcp-cli nlm login`
   — this opens a real browser window, you sign in to your own Google
   account there (no password is ever shared with this skill or with
   Claude), and the session is cached locally.
3. Restart Claude Code (or run `/mcp`) so it picks up the connection.

**Known limitation:** this integration uses `notebooklm-mcp-cli`
(MIT-licensed, community-maintained — not affiliated with Google or
Anthropic), which works by driving NotebookLM's internal, undocumented
web APIs, because Google does not publish a public NotebookLM API. It can
break if NotebookLM's interface changes; treat that as an inconvenience
to re-check, not a security concern — no credentials are ever exposed to
this skill.

## Example

**User:** "microlearning on git and github, basic use cases"

**Output (excerpt of the source document, assuming default style/English):**

```markdown
## Commit — saving a change to history

Explanation: a commit is a snapshot of the project at a point in time...
Example: you edited `extract.py` to handle a new file format...
Command: `git add extract.py` → `git commit -m "Add support for format X"`
What actually happens: `git add` marks the file for the next snapshot;
`git commit` writes that snapshot permanently into history...
```

Plus, if MCP is connected: the source added to the notebook named in
`notebook_name`, and (once format is confirmed) a generation triggered
scoped to that one source.

## Acknowledgments

Built on [`notebooklm-mcp-cli`](https://github.com/jacob-bd/gemini-notebook-mcp-cli)
by Jacob Ben-David (MIT license) — an unofficial, community-maintained
integration with Google NotebookLM. This project is not affiliated with
or endorsed by Google or Anthropic.
