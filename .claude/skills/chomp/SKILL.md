---
name: chomp
description: Clone a GitHub repo, dump its source into a markdown file, then run a full RLM analysis loop against it.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Agent
---

# chomp

Clones a GitHub repo, generates a single markdown dump of its source, loads it as RLM context, generates bits (neutral structured summaries), and runs a full analysis loop.

## Invocation

```
/chomp <git-url> [clone]
```

- `<git-url>` (required): GitHub repository URL.
- `clone` (optional): if present, clone the repo into the current working directory so edits can be made.

## Procedure

### 1. Parse arguments

Read `$ARGUMENTS`. Extract the git URL and check whether the word `clone` is present.

### 2. Clone the repo (if requested)

If `clone` is in the arguments:
```bash
git clone <git-url>
```
This clones into the current working directory (wherever Claude Code is running).

### 3. Generate the markdown dump

Run the chomp shell script to produce the markdown file:
```bash
bash ~/.claude/skills/chomp/scripts/chomp <git-url>
```
This creates a file at `chomp/<repo-name>.md`.

### 4. Load context into the RLM REPL

```bash
python3 ~/.claude/skills/chomp/scripts/rlm_repl.py init chomp/<repo-name>.md
python3 ~/.claude/skills/chomp/scripts/rlm_repl.py status
```

### 5. Scout the context

```bash
python3 ~/.claude/skills/chomp/scripts/rlm_repl.py exec -c "print(peek(0, 3000))"
python3 ~/.claude/skills/chomp/scripts/rlm_repl.py exec -c "print(peek(len(content)-3000, len(content)))"
```

### 6. Chunk the context

```bash
python3 ~/.claude/skills/chomp/scripts/rlm_repl.py exec <<'PY'
paths = write_chunks('chomp/.rlm_state/chunks', size=200000, overlap=0)
print(len(paths))
print(paths[:5])
PY
```

### 7. Generate bits

Run three fixed queries against every chunk using the `rlm-subcall` subagent. These are neutral queries — they capture what the codebase *is*, not what the user wants to do with it.

**Queries:**

1. **surface** — "What are the public APIs, exports, and entry points in this code? List each module's public interface with function signatures and types."
2. **patterns** — "What architectural patterns, conventions, and idioms does this codebase use? Note data flow, error handling, state management, and structural patterns."
3. **deps** — "What are the external dependencies and what does each one provide? Note any significant internal coupling between modules."

**For each query**, invoke the `rlm-subcall` subagent against each chunk file. Collect all results.

**Synthesize** — for each query, combine the subcall results into a single coherent markdown file. Keep each file under 500 words. Write to:

```
chomp/bits/<repo-name>/
  surface.md
  patterns.md
  deps.md
```

### 8. Ask the user their question

Print a summary of the bits that were generated. Stop here and ask the user what they want to know or do with this codebase. Wait for their response before continuing.

### 9. Run the full RLM loop

Once the user provides their question, execute the full RLM workflow:

1. **Subcall loop** — for each chunk file (already written in step 6), invoke the `rlm-subcall` subagent with:
   - The user's question
   - The chunk file path
   - Extraction instructions
   Keep subagent outputs compact and structured (JSON preferred).

2. **Synthesis** — once all chunks are processed, synthesise the final answer in the main conversation. Optionally use `rlm-subcall` once more to merge collected results.

3. **Apply changes** — if the user asked for code changes and `clone` was specified, apply the changes directly to the cloned repo in the current working directory.

## Guardrails

- Do not paste large raw chunks into the main chat context.
- Use the REPL to locate exact excerpts; quote only what you need.
- Subagents cannot spawn other subagents.
- Keep scratch/state files under `chomp/.rlm_state/`.
