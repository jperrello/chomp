# chomp

Turn any Github or personal repo into one big markdown file. Run RLM-style analysis with subagents, and produces structured research output. RLM is taken from https://github.com/brainqub3/claude_code_RLM and the MIT paper: https://arxiv.org/html/2512.24601v1.

There are four use cases for chomp:

- You want to clone a repo into a fresh directory and have a large md copy of its content.
- You want to reference a github repo and just want a md file to reference, not the individual code.
- You want to turn your local codebase filetree and content into a single md file.
- You have a file (PDF, DOCX, etc.) and want to convert it to markdown for analysis.

## Why would you want chomp?

So you can run rlm calls on your codebase which has been proving to retrieve data accurately in a context efficient way. Without this you would need an agent to crawl your codebase, waste context, waste your time, and probably be wrong!

## Use Cases

- `/chomp <git-url> clone` — clones a repo, dumps all source into `chomp/<repo>.md`, then lets you ask questions answered via chunk-by-chunk subagent analysis
- `/chomp local` — same as above but dumps the current repo you're working in into `chomp/local.md` (excludes `chomp/` and `.claude/` dirs). Re-running overwrites the previous `local.md`
- `/chomp report.pdf` — converts a local file to markdown at `chomp/report.md`, then runs the same RLM analysis. The agent figures out what tools to use for the conversion (pdftotext, pandoc, etc.) and installs dependencies if needed
- `/bite <chomp1,chomp2,...> <intent>` — cross-repo research: formulates targeted queries collaboratively with the user, runs them against all chunks, writes narrative research docs
- `/chomp-init` — sets up the current project directory for chomp workflows (installs `/chomp` and `/bite` as project-level skills) and runs an initial `chomp local` dump

## Install

Copy the `chomp-init` skill and its bundled scripts/templates into your global Claude Code config:

```bash
git clone https://github.com/jperrello/chomp.git /tmp/chomp-install
cp -r /tmp/chomp-install/.claude/skills/chomp-init ~/.claude/skills/chomp-init
rm -rf /tmp/chomp-install
```

Requires `git` and `python3`.

## Usage

In any project directory, run `/chomp-init` first. This creates the `chomp/` directories, installs `/chomp` and `/bite` as project-level skills, and generates an initial `chomp/local.md` dump. Exit and restart Claude Code after init so the new commands appear:

```
/chomp-init
# exit and restart Claude Code
/chomp https://github.com/someone/cool-lib clone
```

After chomping, the skill asks what you want to know. For cross-repo research:

```
/bite cool-lib,other-lib how do these two handle authentication
```

The `/bite` command will draft research questions and ask you to refine them before running the analysis. This collaborative step ensures the research targets what you actually need.

## How it works

1. The `chomp` shell script shallow-clones the repo into a temp dir, walks all source files (skipping binaries/artifacts), and writes a single markdown file with a filetree + all source in fenced code blocks. For file mode, the agent converts the file to markdown using whatever tools work (pdftotext, pandoc, etc.), installing dependencies as needed.

2. The `rlm_repl.py` Python script loads that file, chunks it into ~200k char pieces, and writes them to disk.

3. For each chunk, Claude Code spawns a `rlm-subcall` subagent (runs on Haiku) that extracts relevant information for a given query. Results are collected and synthesized in the main conversation.

## File structure after use

```
chomp/
  <repo>.md              # full source dump
  bites/<intent-slug>/   # research output from /bite
    research.md
    surface-map.md
  .rlm_state/            # chunks and pickle state (gitignored)
```

## Design

See [chomp-design-fiction.md](chomp-design-fiction.md) for the original design thinking.
