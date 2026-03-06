---
name: chomp-init
description: Initialize the current project for chomp workflows. Creates directories and appends instructions to the project CLAUDE.md.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
---

# chomp-init

Sets up the current working directory for `/chomp` and `/bite` workflows.

## Invocation

```
/chomp-init
```

No arguments.

## Procedure

### 1. Create directories

```bash
mkdir -p chomp/bits chomp/bites chomp/.rlm_state
```

### 2. Install project-level skills and agent

Copy the chomp and bite skills into the project's `.claude/` directory, and the rlm-subcall agent:

```bash
mkdir -p .claude/skills/chomp .claude/skills/bite .claude/agents
cp ~/.claude/skills/chomp-init/templates/chomp-skill.md .claude/skills/chomp/SKILL.md
cp ~/.claude/skills/chomp-init/templates/bite-skill.md .claude/skills/bite/SKILL.md
cp ~/.claude/skills/chomp-init/templates/rlm-subcall.md .claude/agents/rlm-subcall.md
```

### 3. Update CLAUDE.md

Check if a `CLAUDE.md` exists in the current working directory.

**If it exists**, read it. Check if it already contains the string `## chomp`. If it does, skip this step. If not, append the chomp instructions block (below) to the end of the file.

**If it does not exist**, create `CLAUDE.md` with the chomp instructions block.

**Chomp instructions block:**

```markdown

## chomp - repo analysis

This project uses `/chomp` and `/bite` skills for analyzing GitHub repos.

- `/chomp <git-url> [clone]` — dump a repo's source into `chomp/<repo>.md`, generate bits to `chomp/bits/<repo>/`
- `/chomp local` — dump the current repo into `chomp/local.md` and analyze it in place
- `/bite <chomp1,chomp2,...> <intent>` — research across chomped repos, output to `chomp/bites/<slug>/`
- Chunks and RLM state live in `chomp/.rlm_state/`
```

### 4. Update .gitignore

Check if `.gitignore` exists. If it does, check whether it already contains `chomp/.rlm_state`. If not, append:

```
chomp/.rlm_state/
```

If `.gitignore` does not exist, create it with that line.

### 5. Confirm

Print: "chomp initialized. Run `/chomp <git-url>` to analyze a repo, or `/chomp local` to analyze the current repo."
