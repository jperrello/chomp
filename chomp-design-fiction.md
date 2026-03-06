# Chomp — Design Fiction

## Scene: Tuesday afternoon, March 2026

Ava has been building a real-time dashboard in `~/dash`. She chomped the repo two weeks ago. The chomp file sits at `chomp/dash.md` — 400k characters of every source file, tree and all. She's queried it a few times since then through the RLM, asking about the charting library wrappers, the WebSocket reconnection logic. Normal stuff.

Today she finds `github.com/mira/streamcore` — a library that handles exactly the backpressure problem she's been hand-rolling. She opens Claude Code.

```
/chomp github.com/mira/streamcore
```

No clone. She doesn't want the code in her working directory. She just wants to understand it. The script clones into a temp dir, dumps every source file into `chomp/streamcore.md`, loads it into the RLM, and generates bits — small structured files that capture the first hump of understanding.

```
chomp/streamcore.md (218k)
12 chunks queued
generating bits...

wrote chomp/bits/streamcore/
  surface.md
  patterns.md
  deps.md
```

The system fires off fixed queries against each chunk. Not questions about what Ava wants to do — neutral questions. What does this module export? What patterns does this codebase follow? What are the external dependencies? Each sub-LLM call returns a small structured answer. These get collected and written out.

Each file is a few hundred words. `surface.md` lists the public API — `createStream()`, `BufferPolicy`, `tap()`, `merge()`. `patterns.md` notes that streamcore uses a pull-based iterator protocol, not push-based events. `deps.md` shows it depends on nothing — zero external packages.

Ava glances at the bits. She doesn't read them carefully. They're not for her — they're for the next LLM call.

---

## The bite

She types:

```
/bite dash,streamcore integrate streamcore into my working directory
```

The bite command reads the bits for both repos — `chomp/bits/dash/` and `chomp/bits/streamcore/` — and formulates research questions. Where does dash currently handle backpressure? What would streamcore's `BufferPolicy` replace? Are there type mismatches between streamcore's iterator protocol and dash's event-based WebSocket handler?

The RLM runs these questions against the full chomp files. Not the bits — the actual 400k and 218k dumps. The bits told it where to look. The chomps have the actual code.

The answers come back. The system writes:

```
chomp/bites/integrate-streamcore/
  research.md
  surface-map.md
```

`research.md` is two pages. It identifies three files in dash that handle backpressure (`src/ws/buffer.ts`, `src/ws/reconnect.ts`, `src/charts/throttle.ts`). It notes that `buffer.ts` is doing manually what `streamcore.createStream()` does natively. It flags a mismatch: dash uses `EventEmitter`, streamcore uses `AsyncIterator`. The adapter pattern from streamcore's own examples would bridge this.

`surface-map.md` maps streamcore's API to dash's codebase:

```
streamcore.createStream()  ->  replaces src/ws/buffer.ts (entire file)
streamcore.BufferPolicy    ->  replaces BUFFER_SIZE constants in src/ws/reconnect.ts
streamcore.tap()           ->  replaces src/charts/throttle.ts:42-67
streamcore.merge()         ->  no current equivalent, enables new fan-in pattern
```

The bite doesn't contain implementation. No diffs. No "here's the new version of buffer.ts." It's a research document — legible to Ava, useful to whatever agent does the work next.

If the first round of research reveals gaps, the bite process runs follow-up RLM queries automatically. One pass is often not enough for cross-repo integration questions.

---

## After the bite

Ava reads `research.md`. She sees the `EventEmitter` -> `AsyncIterator` mismatch flagged and thinks yeah, that's the hard part. She could hand this bite to Claude and say "do it." She could refine the bite first — "ignore throttle.ts, I want to keep that manual." She could shelve it and come back Thursday.

---

## Three days later

Ava finds another repo — a date-formatting library. She chomps it. Bits get generated. She never creates a bite for it. She just queries the chomp directly through the RLM when she has a quick question about its locale handling. The bits sit there, tiny and inert, waiting in case a future bite needs to understand the library without reading 150k of source.

---

## The hierarchy

```
chomp/
  dash.md                         <- chomp (full dump, queryable via RLM)
  streamcore.md                   <- chomp
  bits/
    dash/                         <- bits (neutral, durable, small)
      surface.md
      patterns.md
      deps.md
    streamcore/
      surface.md
      patterns.md
      deps.md
  bites/
    integrate-streamcore/         <- bite (task-specific, timestamped)
      research.md
      surface-map.md
```

## Commands

**`/chomp <url> [clone]`** — Clone repo (temp dir unless `clone` specified), dump all source into `chomp/<name>.md`, load into RLM, generate bits. The chomp is the ground truth.

**`/bite <chomp1,chomp2,...> <intent>`** — Read bits for the named chomps, formulate targeted research queries, run them against the full chomp files via RLM, write structured output to `chomp/bites/`. If chomp names or intent are missing, ask for them before proceeding.


## The metaphor

The chomp is the meal. The bits are what you remember about the taste. The bite is when you decide what to cook next.
