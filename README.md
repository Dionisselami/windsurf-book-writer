# windsurf-book-writer

**Write a book in Windsurf.** Cascade plus the [Proseify](https://proseify.xyz) MCP server: genre recipe first,
corpus-grounded register, a real outline, chapter-by-chapter drafting with an edit pass, and a
scoring gate before you call the book done.

[![Listed on mcpservers.org](https://mcpservers.org/badge.svg)](https://mcpservers.org/servers/dionisselami/proseify-mcp) [![MCP Registry](https://img.shields.io/badge/MCP_registry-io.github.Dionisselami%2Fproseify--mcp-4a3f35)](https://registry.modelcontextprotocol.io/v0/servers?search=proseify)

---

## Why this exists

Cascade's strength is autonomous multi-step work, and a book is the most multi-step writing task
there is. Left alone with a premise, an autonomous agent will happily produce thirty chapters of
the same chapter — plausible, well-punctuated, and utterly without a genre.

Proseify gives the agent something external to be wrong against: pacing beats and dialogue ratios
per genre, model passages to calibrate against, an outline it must hold, and `evaluate_book`, which
returns numbers, so "I think it's good" stops being the criterion.

```text
premise  →  genre recipe  →  corpus passages  →  outline  →  chapter drafts  →  evaluate_book  →  revision
```

Never let the model start at "chapter drafts". The ordering is the whole product: a genre
recipe read *before* prose beats any amount of "write like Stephen King" prompting, because it
hands the model concrete numbers — pacing beats, dialogue ratio, sentence-length profile — that
its defaults would otherwise flatten to its own house style.

---

## Quick start

### 1. Get a Proseify key

https://proseify.xyz — sign in, pick a plan (from $19/mo, or the one-time Founding Lifetime tier),
key issued on payment.

### 2. Put the key in your environment

```bash
export PROSEIFY_API_KEY="sk-..."     # macOS / Linux, in your shell profile
[Environment]::SetEnvironmentVariable("PROSEIFY_API_KEY","sk-...","User")   # Windows PowerShell
```

Restart Windsurf afterwards: the app reads its environment at launch, and the config interpolation
below resolves at that moment.

### 3. Connect the MCP server

Windsurf reads `~/.codeium/windsurf/mcp_config.json` (Windows:
`%USERPROFILE%\.codeium\windsurf\mcp_config.json`). Remote HTTP servers use **`serverUrl`**, and
Windsurf interpolates `${env:VAR}` inside `serverUrl` and `headers`:

```json
{
  "mcpServers": {
    "proseify": {
      "serverUrl": "https://mcp.proseify.xyz/mcp",
      "headers": { "Authorization": "Bearer ${env:PROSEIFY_API_KEY}" }
    }
  }
}
```

`mcp_config.json` in this repo is that block. You can also open it from Cascade's MCP panel
(Cmd/Ctrl+Shift+P → "MCP"), which shows the connection state and the discovered tools.

### 3. Install the writing skill

```bash
npx skills add https://proseify.xyz --skill anti-prose-slop -y
```

The MCP server gives the agent the *tools*. The `anti-prose-slop` skill gives it the *method* —
genre recipe first, corpus grounding, chapter discipline, and the per-chapter edit pass. Install
both or you get tool access and the same generic prose you had before.

### 4. Drop the rules file in and give it a premise

`windsurf-rules.md` in this repo is the Cascade rules content — paste it into **Windsurf Settings →
Cascade → Rules** (global, for book work everywhere) or `.windsurfrules` in the book project.

> Write a 26-chapter gothic mystery set in a sanatorium in 1911: a nurse keeps finding patient
> records for someone who has not been admitted. Around 55,000 words.

---

## What's in here

| File | Purpose |
|------|---------|
| `mcp_config.json` | The `proseify` server block for `~/.codeium/windsurf/mcp_config.json` |
| `windsurf-rules.md` | Cascade rules content — the writing discipline |
| `LICENSE` | MIT |

### The five-step loop

1. **Pick the genre before any prose exists.** `list_genres`, then `get_genre_recipe` for the
   closest fit. Blends are allowed — name both and let the recipe argue for one.
2. **Ground the register.** `search_corpus` (or `get_style_references`) for 3–5 model passages
   in the target genre. Read them. This is the calibration step, not decoration.
3. **`plan_book` once.** Hold the outline. Do not re-plan mid-draft; if the outline is wrong,
   fix it deliberately and say so.
4. **Draft chapter by chapter**, pausing after each for the per-chapter edit pass below.
5. **`evaluate_book` before you call it done.** Fix the chapters it flags and re-run the gate.

### The per-chapter edit pass

The failure mode of AI prose is not grammar — it is sameness. Every chapter gets struck against
this list before it counts as drafted:

- `seemed to`, `began to`, `started to`, `could feel` — delete or recast
- filter words: *felt, noticed, watched, saw, heard* — put the reader in the perception instead
- stacked adverbs on dialogue tags; `said` is not a problem, `said loudly, angrily` is
- sentence openings repeated across the chapter (and the weather-opening default)
- three-item lists used as rhythm filler
- dialogue that exists to explain the plot to the reader
- simile endings: the last line becoming a metaphor for the chapter

### If you only read one paragraph

The agent's own model does all the writing — Proseify calls no LLM and stores no manuscript.
There is no token bill from us: it is a corpus, a set of genre recipes, and a pipeline.


---

## Windsurf notes and gotchas

- **Remote servers use `serverUrl`.** Windsurf accepts `url` as an alias, but `serverUrl` is the
  documented field and the one the MCP panel writes back. If the server never appears, check this
  first.
- **`${env:VAR}` interpolation is what keeps the key out of the file.** Without it you are putting
  a live credential in a config file in your home directory — works, but it is one backup or screen
  share away from being someone else's key.
- **Restart Windsurf after changing the environment.** The variable is read at launch; a config that
  looks perfect can send no `Authorization` header at all, and the tools simply will not be there.
- **Cascade will happily run past its context.** For anything past novella length, have it write each
  chapter to `chapters/NN-title.md` before starting the next one. The files are the memory; the
  conversation is not.
- **Rules go in Cascade's Rules, not just the repo.** A `.windsurfrules` file that Windsurf does not
  read is a README with delusions. Verify the rules are active before blaming the model.
- **Keep the tool count small.** Cascade's autonomy plus seven well-described tools is a good pair;
  a workspace with twenty MCP servers competing for the tool list is not.

| Tool | What it does |
|------|--------------|
| `list_genres` | The 11 genres the corpus covers (theatre, horror, romance, adventure, literary, mystery, gothic, sci-fi, fantasy, comedy, children's) |
| `get_genre_recipe` | Pacing beats, dialogue ratio, sentence-length profile and stylistic anchors derived from that tradition |
| `search_corpus` | FTS5 full-text search across 2,500+ chapters of public-domain classics (quoted phrases, AND/OR/NOT, wildcards) |
| `get_style_references` | Model passages from books in the target genre — the register you are aiming at |
| `plan_book` | A full chapter-by-chapter outline from a one-line premise, each beat carrying a corpus style reference |
| `write_book` | The one-shot flow: plan → draft → evaluate, in a single session |
| `evaluate_book` | Scores a draft against genre benchmarks (structure, pacing, chapter coverage, word budget) so weak chapters get revised |

## The corpus

80+ books and 2,500+ chapters of public-domain literature — Project Gutenberg and similar
sources — every chapter verified against its own file header at download time, so nothing in
the library has a copyright question hanging over it. Works from Austen, Stevenson, Hugo,
Conrad, Verne, the Brontës, Shelley and the gothic masters, grouped by genre and indexed for
full-text search.

Commercial use of what you write on top of it is safe. Full-text searchable chapter by chapter,
not a scrape of titles.

## Pricing

| Plan | Price | Rate limit |
|------|-------|-----------|
| Quill (Starter) | $19/mo | 120 req/min |
| Fable (Pro) | $49/mo | 400 req/min |
| Opus (Studio) | $99/mo | unlimited (fair use) |
| Founding Lifetime | $149 one-time | 400 req/min, all genres |

Sign in at https://proseify.xyz, pick a plan, and the key is issued the moment the purchase
clears. Cancel from https://proseify.xyz/account; 14-day refund window
(https://proseify.xyz/refunds).

## Other clients

Same Proseify server, same workflow, different config file. The cookbook is the method itself.

- [claude-book-writer — write a book with Claude Code](https://github.com/Dionisselami/claude-book-writer)
- [chatgpt-book-writer — write a book on your ChatGPT plan](https://github.com/Dionisselami/chatgpt-book-writer)
- [cursor-book-writer — write a book in Cursor](https://github.com/Dionisselami/cursor-book-writer)
- [gemini-cli-book-writer — write a book with Gemini CLI](https://github.com/Dionisselami/gemini-cli-book-writer)
- [copilot-book-writer — write a book in VS Code with Copilot](https://github.com/Dionisselami/copilot-book-writer)
- [claude-book-cookbook — recipes for writing a whole book with Claude](https://github.com/Dionisselami/claude-book-cookbook)

## License

MIT for everything in this repository. The corpus texts themselves are public domain.

## Disclaimer

Unofficial. This repository is not affiliated with, endorsed by, or sponsored by Cognition, Codeium or Windsurf. Client names appear only to describe which configuration file and transport the instructions are for. Proseify is an independent product — https://proseify.xyz.

Keep your Proseify key in local agent config or an environment variable. Never commit it, print it, log it, or paste it into a repository file.

## Links

- **Sign up / key issuance:** https://proseify.xyz
- **Filled-in config for your key:** https://proseify.xyz/agent
- **FAQ (ownership, KDP, what an MCP server is):** https://proseify.xyz/faq
- **Server repo & MCP registry entry:** https://github.com/Dionisselami/proseify-mcp
- **Support:** support@proseify.xyz
