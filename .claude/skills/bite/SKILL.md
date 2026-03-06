---
name: bite
description: Read bits from named chomps, formulate targeted research queries based on intent, run them against the full chomp files via RLM, and write structured research output.
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

# bite

Reads bits for named chomps, formulates research questions driven by intent, runs them against the full chomp files, and writes structured output.

## Invocation

```
/bite <chomp1,chomp2,...> <intent>
```

- `<chomp1,chomp2,...>` (required): Comma-separated chomp names (matching filenames in `chomp/`).
- `<intent>` (required): What the user wants to accomplish. A phrase or sentence.

If chomp names or intent are missing, ask for them before proceeding.

## Procedure

### 1. Parse arguments

Read `$ARGUMENTS`. Extract the comma-separated chomp names and the intent string (everything after the chomp names).

### 2. Verify chomps and read bits

For each chomp name:
- Verify `chomp/<name>.md` exists.
- Read all bit files from `chomp/bits/<name>/` (surface.md, patterns.md, deps.md).

If any chomp or its bits are missing, stop and tell the user. They may need to run `/chomp` first.

### 3. Formulate research questions

Using the bits content and the user's intent, generate 3-8 targeted research questions. These should be specific enough to answer from source code. Think about:

- Where in each codebase does the intent touch?
- What interfaces need to connect?
- What mismatches or conflicts might exist?
- What patterns in one codebase affect integration with another?

Print the research questions so the user can see them.

### 4. Load and chunk each chomp

For each chomp name, init the RLM and write chunks to a chomp-specific directory:

```bash
python3 ~/.claude/skills/chomp/scripts/rlm_repl.py init chomp/<name>.md
python3 ~/.claude/skills/chomp/scripts/rlm_repl.py exec <<'PY'
paths = write_chunks('chomp/.rlm_state/chunks/<name>', size=200000, overlap=0)
print(len(paths))
PY
```

Repeat for each chomp. Collect all chunk file paths.

### 5. Run research queries

For each research question, invoke the `rlm-subcall` subagent against every chunk file across all chomps. Pass the research question as the query.

Collect all subcall results. Keep them structured (JSON).

If the first round reveals gaps or unanswered questions, formulate follow-up queries and run another round. One pass is often not enough for cross-repo integration questions.

### 6. Synthesize and write output

Slugify the intent (e.g. "integrate streamcore into my working directory" -> `integrate-streamcore`).

From the collected subcall results, synthesize two files:

**`research.md`** — A narrative research document (1-3 pages). Should cover:
- Which files/modules in each codebase are relevant to the intent
- What each codebase does in the relevant areas (with specific file paths and function names)
- Mismatches, conflicts, or gaps between the codebases
- Recommended approach or adapter patterns

**`surface-map.md`** — A mapping table showing how APIs/modules in one codebase relate to the other. Format:

```
source_api  ->  target_location (action: replaces/augments/no equivalent)
```

Write both files to:

```
chomp/bites/<intent-slug>/
  research.md
  surface-map.md
```

### 7. Present results

Print a summary of what was found. Highlight the most important findings — especially mismatches or surprises. The user should be able to read the summary and decide whether to act on the bite, refine it, or shelve it.

## Guardrails

- Bites contain no implementation. No diffs, no generated code. They are research documents.
- Do not paste large raw chunks into the main chat context.
- Subagents cannot spawn other subagents.
- Keep scratch/state files under `chomp/.rlm_state/`.
