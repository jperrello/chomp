# Chomp — Design Fiction

## Scene: Tuesday afternoon, March 2026

Ava has been building a real-time dashboard in `~/dash`. She chomped the repo two weeks ago. The chomp file sits at `chomp/dash.md` — 400k characters of every source file, tree and all. She's queried it a few times since then through the RLM, asking about the charting library wrappers, the WebSocket reconnection logic. Normal stuff.

Today she finds `github.com/mira/streamcore` — a library that handles exactly the backpressure problem she's been hand-rolling. She opens Claude Code.

```
/chomp github.com/mira/streamcore
```

No clone. She doesn't want the code in her working directory. She just wants to understand it. The script clones into a temp dir, dumps every source file into `chomp/streamcore.md`, loads it into the RLM, chunks it, and asks what she wants to know.

```
chomp/streamcore.md (218k)
12 chunks queued

What do you want to know about this codebase?
```

She asks a couple quick questions — what's the public API, how does it handle backpressure. The RLM fires subcalls against the chunks, extracts what's relevant, synthesizes an answer. Fast and cheap. She gets what she needs without reading the source herself.

---

## The bite

Now she wants to integrate it. She types:

```
/bite dash,streamcore integrate streamcore into my working directory
```

The bite command formulates research questions based on her intent. Where does dash currently handle backpressure? What would streamcore replace? Are there type mismatches between streamcore's iterator protocol and dash's event-based WebSocket handler?

It presents the draft questions and asks Ava to refine them. She adds one — "does streamcore have any opinion about reconnection, or is that orthogonal?" — and drops one she doesn't care about.

The RLM runs these questions against the full chomp files. The actual 400k and 218k dumps. Every chunk gets a subcall. The answers come back. The system writes:

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

Ava finds another repo — a date-formatting library. She chomps it. She never creates a bite for it. She just queries the chomp directly through the RLM when she has a quick question about its locale handling. The chomp file sits there, inert, ready for a future bite or a quick ad-hoc question.

---

## The hierarchy

```
chomp/
  dash.md                         <- chomp (full dump, queryable via RLM)
  streamcore.md                   <- chomp
  bites/
    integrate-streamcore/         <- bite (task-specific research)
      research.md
      surface-map.md
  .rlm_state/                    <- chunks and scratch (gitignored)
```

## Commands

**`/chomp <url> [clone]`** — Clone repo (temp dir unless `clone` specified), dump all source into `chomp/<name>.md`, load into RLM, chunk it, and ask the user what they want to know. The chomp is the ground truth.

**`/chomp local`** — Same as above but dumps the current repo into `chomp/local.md`. Excludes `chomp/` and `.claude/` directories.

**`/bite <chomp1,chomp2,...> <intent>`** — Formulate targeted research queries collaboratively with the user, run them against the full chomp files via RLM, write structured output to `chomp/bites/`. If chomp names or intent are missing, ask for them before proceeding.


## The metaphor

The chomp is the meal. The bite is when you decide what to cook next.
