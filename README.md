# chomp

Analyze any GitHub repo from inside Claude Code. Dumps source into a single file, chunks it, runs RLM-style analysis with subagents, and produces structured research output.

## What it does

- `/chomp <git-url>` — clones a repo, dumps all source into `chomp/<repo>.md`, generates neutral "bits" (summaries of APIs, patterns, dependencies), then lets you ask questions answered via chunk-by-chunk subagent analysis
- `/bite <chomp1,chomp2,...> <intent>` — cross-repo research: reads bits from multiple chomped repos, formulates targeted queries, runs them against all chunks, writes narrative research docs
- `/chomp-init` — sets up the current project directory for chomp workflows

## Install

Copy the skills and agents into your global Claude Code config:

```bash
git clone https://github.com/jperrello/chomp.git /tmp/chomp-install

# skills
cp -r /tmp/chomp-install/.claude/skills/chomp ~/.claude/skills/chomp
cp -r /tmp/chomp-install/.claude/skills/bite ~/.claude/skills/bite
cp -r /tmp/chomp-install/.claude/skills/chomp-init ~/.claude/skills/chomp-init

# subagent
mkdir -p ~/.claude/agents
cp /tmp/chomp-install/.claude/agents/rlm-subcall.md ~/.claude/agents/rlm-subcall.md

# clean up
rm -rf /tmp/chomp-install
```

Requires `git` and `python3`.

## Usage

In any project directory:

```
/chomp-init
/chomp https://github.com/someone/cool-lib
```

After chomping, the skill generates bits and asks what you want to know. For cross-repo research:

```
/bite cool-lib,other-lib how do these two handle authentication
```

## How it works

1. The `chomp` shell script shallow-clones the repo into a temp dir, walks all source files (skipping binaries/artifacts), and writes a single markdown file with a filetree + all source in fenced code blocks.

2. The `rlm_repl.py` Python script loads that file, chunks it into ~200k char pieces, and writes them to disk.

3. For each chunk, Claude Code spawns a `rlm-subcall` subagent (runs on Haiku) that extracts relevant information for a given query. Results are collected and synthesized in the main conversation.

4. Bits (surface, patterns, deps) are neutral summaries cached in `chomp/bits/<repo>/` so future `/bite` calls don't re-analyze from scratch.

## File structure after use

```
chomp/
  <repo>.md              # full source dump
  bits/<repo>/           # neutral summaries
    surface.md
    patterns.md
    deps.md
  bites/<intent-slug>/   # research output from /bite
    research.md
    surface-map.md
  .rlm_state/            # chunks and pickle state (gitignored)
```

## Design

See [chomp-design-fiction.md](chomp-design-fiction.md) for the original design thinking.
