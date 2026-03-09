## rlm-repl

Python script (~400 lines) implementing a persistent mini-REPL for RLM workflows. Subcommands: init (load context file), exec (run Python code with injected helpers), status, reset, export-buffers. State persisted via pickle. Injected helpers include peek, grep, chunk_indices, write_chunks, and add_buffer for working with large text contexts.

**Source:** `.claude/skills/chomp-init/scripts/rlm_repl.py`
