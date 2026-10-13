# bradan HANDBOOK — adapter recipes

> Core repo is dependency-free by design. This file shows how the distilled
> corpus plugs into external systems: a knowledge base, keyword lanes, and a
> render pipeline. Adopt the recipes or ignore the file — nothing in core
> imports it.

## 1. Knowledge-base assimilation

After `distill.py` produces a corpus, file one knowledge-base page per course
under `sources/` (primary subject = the course), plus per-lesson pages under
`media/courses/<course-slug>/`:

```markdown
# {Course Title} — Course Page

**Source:** acquired via scrape.py — see acquisition.json in the media dir
**Format:** course (N lessons, M minutes audio)
**Distilled:** YYYY-MM-DD, bradan v0.1

## Summary

{2–4 sentences: what the course teaches, who it's for}

## Keywords (from distill keywords.json)

{top 20 terms with tf/documents — these feed the keyword lanes}

## Concepts

{top concepts from concepts.json}

## Next-best-sentence seeds

{top seeds — candidate openers for marketing surfaces}

## Lesson index

- [Lesson 1 — title](lesson-01) (4.2 min) — transcript: media/courses/{slug}/lesson-01.txt
  ...
```

Filing rule: file by primary subject (the course), not by format; entity
extraction + back-links per your knowledge base's ingest phases.

## 2. Keyword lanes (research components)

The `keywords.json` output is direct food for per-platform keyword lanes:

- Each lane (Google, YouTube, Meta, TikTok, Reddit) takes `keywords.json` as
  seed terms — run platform-specific expansion on top.
- The `next-best-seeds.json` sentences are candidate first-sentences: fuse
  with ICP language and score per your composition stage.
- Convergence rule: terms appearing in ≥ half the lesson sources (the
  `documents` field) get priority — cross-source convergence is the strongest
  ranking signal.

## 3. Render script generation

`next-best-seeds.json` + `keywords.json` → render `slides.py` config (see the
rendermill project for the render engine):

1. Pick top seed as the hook line (first sentence spoken in first 3s).
2. Build beats from concepts (one escalating beat per concept cluster).
3. Narration: plain-language doctrine (median ~9 words/sentence, zero
   filler, one passive max).
4. Voice: your explainer-voice preset; music bed obeying the 60–90 BPM
   no-lyrics rule.
5. Preflight gates: hook ≤3s timing check, storyboard gate.

## 4. Deployment notes

- **Daily driver** (Apple Silicon): interactive/one-off transcription. Light
  use only if memory is tight — transcription is heavy-class.
- **Batch node** (headless Apple Silicon or x86): course-scale runs
  (whole-course transcription) belong here. Setup: `bash install.sh` there
  (same venv path), then run consumer.py from the repo checkout.
- **x86 PC** (Windows/Linux): use the faster-whisper backend
  (`--backend faster`), CPU inference.
- The venv is per-box (isolated); the repo is one clone per box.

## 5. MCP + external agents

Run `python3 mcp-server.py` and register it with any MCP client (Claude,
Cursor, any agent's MCP config). Tools: `transcribe`, `scrape`, `distill`.
This is the "other agentic tools and MCP stuff" surface — other people's
agents drive the same pipeline yours does, with zero deployment-specific
internals.
