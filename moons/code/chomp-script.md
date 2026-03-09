## chomp-script

Bash script (~210 lines). Accepts a git URL or `local` as input. For URLs, shallow-clones into a temp dir. Walks all source files, filtering out binary extensions, ignored directories, and optionally limiting to specified extensions. Outputs a single markdown file with a `<filetree>` section and a `<source_code>` section containing every file in fenced code blocks.

**Source:** `.claude/skills/chomp-init/scripts/chomp`
