# Book writing with Proseify (Cascade)

The `proseify` MCP server is connected via `~/.codeium/windsurf/mcp_config.json`. It is a corpus and
a genre-discipline layer: **you** write the prose, it supplies the literature, the genre recipes and
the scoring gate. It calls no model of its own.

## Order of operations — do not reorder

1. `list_genres`, then `get_genre_recipe` for the closest fit to the ask. State the recipe you chose
   and why, in one line, before drafting anything.
2. `search_corpus` for 3-5 model passages in that genre. Read them. Quote the line that sets the
   register you are aiming at.
3. `plan_book` **once** — premise, chapter count, word budget. Print the outline and hold it. Do not
   re-plan mid-draft; if the outline is wrong, say so and fix it deliberately.
4. Draft **one chapter per step**. After each chapter run the edit pass below and state what you cut.
   Do not begin the next chapter until the current one has been through it.
5. `evaluate_book` before declaring the book finished. Fix the chapters it flags and re-run the gate.

## Per-chapter edit pass

- `seemed to`, `began to`, `started to`, `could feel` — delete or recast
- filter words: *felt, noticed, watched, saw, heard*
- stacked adverbs on dialogue tags; plain `said` is fine
- repeated sentence openings, and the weather-opening default
- three-item lists used as rhythm filler
- dialogue that exists only to explain the plot to the reader
- the last line quietly turning into a metaphor for the chapter

## Files and rules

- One file per chapter: `chapters/NN-title.md`, written to disk before the next chapter starts.
- `outline.md` holds the `plan_book` output verbatim. Re-read it instead of re-planning.
- Never print, log or commit the Proseify key. It is read from the `PROSEIFY_API_KEY` environment
  variable through the config's `${env:...}` interpolation, never typed into a file.
- Ask before you cascade: if a chapter has drifted from the outline, say so and stop, rather than
  silently repairing the story to fit what you already wrote.
