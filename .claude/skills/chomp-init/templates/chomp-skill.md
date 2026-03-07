---
name: chomp
description: Clone a GitHub repo (or dump the current repo), generate a markdown file of its source, then run a full RLM analysis loop against it.
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

Clones a GitHub repo (or dumps the current repo with `local`), generates a single markdown file of its source, loads it as RLM context, and runs a full analysis loop.

## Invocation

```
/chomp <git-url|local> [clone]
```

- `<git-url|local>` (required): GitHub repository URL, or `local` to chomp the current repo.
- `clone` (optional): if present and using a git URL, clone the repo into the current working directory so edits can be made. Ignored when using `local`.

## Procedure

### 1. Parse arguments

Read `$ARGUMENTS`. Determine if the first argument is `local` or a git URL. If it's a git URL, check whether the word `clone` is present.

### 2. Clone the repo (if requested, git URL only)

If using a git URL and `clone` is in the arguments:
```bash
git clone <git-url>
```
This clones into the current working directory (wherever Claude Code is running).

Skip this step entirely when the argument is `local`.

### 3. Generate the markdown dump

Run the chomp shell script to produce the markdown file:
```bash
bash ~/.claude/skills/chomp-init/scripts/chomp <git-url|local>
```
- For a git URL, this creates `chomp/<repo-name>.md`.
- For `local`, this creates `chomp/local.md`. The `chomp/` and `.claude/` directories are automatically excluded. Running this again overwrites the previous `local.md`.

### 4. Load context into the RLM REPL

Use `local` as the repo name when the argument is `local`, otherwise use the repo name extracted from the URL.

```bash
python3 ~/.claude/skills/chomp-init/scripts/rlm_repl.py init chomp/<repo-name>.md
python3 ~/.claude/skills/chomp-init/scripts/rlm_repl.py status
```

### 5. Scout the context

```bash
python3 ~/.claude/skills/chomp-init/scripts/rlm_repl.py exec -c "print(peek(0, 3000))"
python3 ~/.claude/skills/chomp-init/scripts/rlm_repl.py exec -c "print(peek(len(content)-3000, len(content)))"
```

### 6. Chunk the context

```bash
python3 ~/.claude/skills/chomp-init/scripts/rlm_repl.py exec <<'PY'
paths = write_chunks('chomp/.rlm_state/chunks', size=200000, overlap=0)
print(len(paths))
print(paths[:5])
PY
```

### 7. Ask the user their question

Print a brief summary of what was loaded (repo name, file count, total size). Stop here and ask the user what they want to know or do with this codebase. Wait for their response before continuing.

### 8. Run the full RLM loop

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
