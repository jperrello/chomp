# chomp

This repo is the distribution package for the chomp/bite Claude Code skills. See README.md.

### Moons (Knowledge Base)
This project uses a structured knowledge graph called `moons/` to organize project context.

**Structure:**
- `moons/graph.json` — the map. Lightweight nodes with typed edges. Read this FIRST.
- `moons/note/` — documentation and explanatory files
- `moons/concept/` — design thinking and conceptual models
- `moons/code/` — executable scripts and implementations
- `moons/config/` — skill definitions, agent templates, and project configuration

**How to use:**
1. Read `moons/graph.json` first when you need project context. It gives complete visibility of all nodes, edges, and types.
2. Follow edges to content files when you need depth on a specific node.
3. When you create or significantly modify a file, add or update its node in graph.json and the corresponding content file in moons/.
4. Keep graph.json lean — short descriptions, not full content.
5. After editing graph.json, run `python moons/sync.py` to update the visualization.
