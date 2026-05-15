---
name: research_and_draft_chapter
description: End-to-end recipe for producing a Manning-ready first-draft chapter of "Build a Large Robot Model (From Scratch)". Drives the full pipeline — seed-source research, cross-book learning synthesis, structure planning against the proposal, manuscript drafting, section-level review by subagents, figure generation, dedicated chapter-reviewer pass, and final DOCX assembly with Manning template styles and embedded figures. Use this when starting a new chapter.
model: opus
argument-hint: "<chapter_number> [optional: path/to/seed-source]"
---

# Research and Draft a Chapter

This skill encodes the end-to-end process for producing a first-draft chapter (markdown + Manning-styled DOCX with figures embedded). Run the stages in order; do not skip ahead. Each stage has a clear input artifact, a clear output artifact, and a quality bar that gates progression to the next stage.

The skill assumes you are operating from the book's project root — the directory that contains `proposal/`, `book_learnings.md`, `chapter_template/`, `writing_instructions/`, `mqr/`, and the existing per-chapter folders. All paths below are relative to that root.

## Inputs you should always have on hand

| Source | Where it lives | Role |
|--------|----------------|------|
| Book proposal | `proposal/proposal.md` | Authoritative TOC, learning goals, scope contract. Every chapter must trace back to here. |
| Cross-book learnings | `book_learnings.md` | Patterns harvested from Sebastian Raschka's *Build a Large Language Model*, *Build a Reasoning Model*, Nathan Lambert's *RLHF Book*, *Designing Deep Learning Systems*, and others. The Manning rhythm playbook. |
| Manning house style | `writing_instructions/writing_instructions.md` | Hard formatting rules (Heading 4 for listings, Heading 5 for tables, Heading 6 for figures, blockquote callouts, no em dashes, etc.). |
| Minimally Qualified Reader | `mqr/mqr.md` | The reader profile. Every word must be readable by an MQR-class engineer. |
| Voice reference | The most recently completed chapter's `manuscript/` | The most-recently-shipped chapter sets the voice, density, and Manning rhythm. Treat it as the calibration target. Default to `chapter_1/manuscript/` until later chapters have shipped. |
| DOCX template | `chapter_template/chapter_template.docx` | Source of Title, Subtitle, Heading 1-6, and table/listing/figure caption styles. |
| In-house Manning linter | `agents/book/SKILL.md` | Run via `/book lint`, `/book structure`, `/book mqr-check` on the manuscript at any stage. |
| Chapter reviewer agent | `.claude/agents/lrm-chapter-reviewer.md` | Final-pass review agent — invoke with `Agent(subagent_type="lrm-chapter-reviewer", ...)`. |

## Output skeleton (create up front)

```text
chapter_<N>/
├── resources/                              # Stage 1 deposit
│   ├── claude_research_summary.md
│   ├── gemini_research_synthesis.md
│   └── (any user-provided seed PDFs/MDs)
├── chapter_<N>_structure_and_plan.md       # Stage 3 plan
├── structure.md                            # Stage 3 mirror (lighter index)
├── manuscript/
│   ├── chapter_<N>.md                      # Stage 4 draft, iteratively edited
│   ├── _build_docx.py                      # Stage 10 converter
│   └── chapter_<N>.docx                    # Stage 10 final deliverable
└── figures/
    ├── graphic_generation.md               # Stage 7 prompt set
    └── diagrams/
        ├── _build_figures.py               # Stage 7 PIL renderer (fallback)
        ├── figure_<N>_1.{drawio,png}
        ├── figure_<N>_2.{drawio,png}
        └── …
```

Make this skeleton with `mkdir -p` before Stage 1.

---

## Stage 1 — Research

**Goal.** Collect a complete, factually-grounded picture of the topic the chapter covers, before any structural decisions.

**Inputs.** The chapter heading from `proposal/proposal.md`. Optional seed sources from the user (URLs, PDFs, paragraphs of prior art).

**Action.**
1. If the user provided seed sources, save them to `chapter_<N>/resources/` verbatim.
2. Spawn **two parallel research agents** (`general-purpose`, run in background) — one tasked with "first-principles" coverage of the topic (mathematical foundations, why the problem exists, the conceptual move the chapter teaches), one tasked with surveying the most recent published systems and named techniques in the topic area. Each writes one consolidated dossier:
   - `chapter_<N>/resources/claude_research_summary.md`
   - `chapter_<N>/resources/gemini_research_synthesis.md`
3. Each dossier should answer: *what is the mathematical core?*, *what are the design choices the field has converged on?*, *what are the named pitfalls?*, *what are the canonical citations?*

**Quality bar.** Both dossiers should overlap meaningfully (catches hallucinations) and disagree on at least one specific detail (catches over-summarization). When they disagree, **read the cited paper** before writing the chapter — do not pick by majority vote.

**Anti-patterns.**
- Letting one dossier act as the sole source. Always cross-reference.
- Citing arXiv IDs that don't resolve. Verify every arXiv ID with `WebFetch`.

---

## Stage 2 — Synthesize learnings from other books

**Goal.** Pull *style and structure* patterns from the existing curated learnings before designing the chapter's outline.

**Input.** `book_learnings.md` (root). Optional: read directly from the PDFs under `other books/` if learnings.md lacks coverage for a pattern you need (mathematical-derivation rhythm from Raschka, RL-vocabulary discipline from Lambert, ML-systems framing from *Designing Deep Learning Systems*).

**Action.** Re-read `book_learnings.md` and flag — by section number — the items that apply to this chapter. Things to look for:
- Three-beat opener (concrete problem → why the obvious fix fails → the chapter's fix).
- "Let's run it" micro-loop after every conceptual block.
- Worked numeric example per algorithm.
- Honest scoping disclaimer at the end of every survey table.
- Tight equation-to-code coupling (within 5 lines).

**Output.** No new file. A mental (or scratch) list of the patterns you will deliberately apply during Stage 3 planning.

---

## Stage 3 — Structure and plan

**Goal.** A section-by-section outline detailed enough that the manuscript writer can produce prose without re-doing any research.

**Action.**
1. Use the `superpowers:brainstorming` skill (or its equivalent for non-Claude-Code platforms) to draft the structure. Brainstorming should ask the user *up front* about any flex from the proposal — deviating is fine, but only if it improves the chapter. Common flexes: adding a "why this matters" section in front of the proposed §X.1 to motivate the topic before the technical work begins, or splitting a long final section into "limits and bridge" plus "summary".
2. Write `chapter_<N>/chapter_<N>_structure_and_plan.md`. Required contents:
   - One section per `X.Y` heading the chapter will have.
   - For each section: subsections, bullet list of items to cover, where figures help and what they roughly look like, where listings sit and what they contain (titles + 1-line behavior), where tables sit and what columns/rows they have.
   - A consolidated *listing summary table* and *figure summary table* and *table summary table* near the bottom of the plan so listing/figure/table numbering can be sanity-checked in one place.
3. Copy the same content to `chapter_<N>/structure.md` (lighter or identical — they serve as a checkpoint pair).
4. **Cross-reference the proposal.** Any flex from `proposal/proposal.md` must be a *deliberate* decision recorded in the plan, with a one-sentence justification.

**Quality bar.** Hand the plan to the user for explicit approval before drafting. The plan is cheap to revise; the manuscript is not.

**Anti-patterns.**
- Planning so loosely that the manuscript writer has to make sequencing decisions. Pre-decide.
- Listing-number off-by-ones between plan and final manuscript. Use the listing summary table as the canonical source.

---

## Stage 4 — Draft manuscript

**Goal.** A complete `chapter_<N>.md` in Markdown that reads with the book's established voice and obeys Manning house style.

**Action.**
1. Read the most-recently-shipped chapter's `manuscript/` for voice calibration. The opening must follow the three-beat rhythm (concrete problem → why the obvious fix fails → the chapter's fix announced as the same trick that powers a system the reader has heard of).
2. Write the chapter section by section. Conventions to follow:
   - Each listing is preceded by `**Listing X.Y** <one-line caption>.` and a fenced code block. **Code bodies can be placeholders** during drafting (the user will fill them in a later pass) but the captions, annotations (`#A`, `#B`, …), and surrounding prose must be final.
   - Each figure is referenced as `**Figure X.Y** <one-paragraph caption that describes what the reader takes away, not just what it labels>`. The actual PNG is generated in Stage 7.
   - Each table is preceded by `**Table X.Y** <caption>` then a markdown table.
   - Callouts are blockquote-style: `> **PITFALL — title.**` / `> **TIP — title.**` / `> **DEEP DIVE — title.**` / `> **NOTATION — title.**` / `> **DEFINITION — title.**` / `> **NOTE — title.**`. *Do not* use heading-pair callouts (`#### Pitfall` / `#### Title`) — the DOCX converter and the Manning template assume blockquote format.
3. Apply learnings from Stage 2 in real time as you write. If you reach the end of a conceptual section without a "let's run it" micro-loop, go back and add one.
4. Adhere to hard Manning rules: no em dashes anywhere in body prose (use commas or sentence breaks), no marketing words, no meta-language ("in this chapter, we will…"), bullet lists with `-` (the converter promotes them to `•`).
5. After the section's prose is written, run `/book lint` and `/book structure` on the chapter file and fix anything the in-house linter flags. This catches the easy stuff before reviewers do.

**Output.** `chapter_<N>/manuscript/chapter_<N>.md` ~ 10-15 k words, plus 1 file-level write at the end.

---

## Stage 5 — Section reviews

**Goal.** Defend the chapter against MQR-comprehension failures before the chapter-reviewer agent runs.

**Action.** Spawn **N parallel `general-purpose` subagents** (Sonnet model, run in background), one per section or section-pair. Aim for ~4-6 subagents per chapter; group adjacent short sections into a single subagent and give long, dense sections their own subagent. Each gets:
- The full manuscript file.
- A direct quote from `mqr/mqr.md` describing what the reader does and does not have.
- A quote from `writing_instructions/writing_instructions.md` for the formatting bar.
- An explicit instruction to flag issues as BLOCKER / MAJOR / MINOR / NIT.

Each review returns a single Markdown report. Read all of them inline.

**Apply** every BLOCKER and MAJOR before Stage 6, even if it means rewriting paragraphs. MINORs and NITs can be batched.

---

## Stage 6 — Iteration on review fixes

**Goal.** A manuscript where every reviewer-flagged issue is either fixed or has a written reason to defer.

**Action.** Apply fixes section by section. Re-read changed prose to make sure transitions still flow — fixes can shatter the surrounding paragraph if applied carelessly. Re-run `/book lint` after a batch of edits.

---

## Stage 7 — Figure generation

**Goal.** All 6–8 figures rendered as crisp publication-grade PNGs in `chapter_<N>/figures/diagrams/`, with editable `.drawio` companions where feasible.

**Action.**
1. Write `chapter_<N>/figures/graphic_generation.md` — one section per figure, with a long-form prompt (palette, layout, labels, accent color, aspect ratio). Mirror the format of the most-recently-shipped chapter's `graphic_generation.md`.
2. Attempt to spawn **parallel `general-purpose` subagents** (roughly one per two figures), each tasked with rendering its figures via the `draw-io` skill. **Known issue:** subagents in this environment frequently get denied access to the `draw-io` skill and to most shell commands. When that happens:
3. **Fallback:** write a single Python file `chapter_<N>/figures/diagrams/_build_figures.py` that uses Pillow (`pip3 install Pillow`) to render every figure deterministically. The most-recently-shipped chapter's `_build_figures.py` is the canonical reference — copy it, change the palette per figure, and adjust the layout. Output both a PNG (the deliverable) and a stub `.drawio` (a tiny mxfile that references the PNG, so the file is editable in draw.io if the author wants to overlay text later).
4. Whichever path produced the PNGs, **visually inspect each PNG via the Read tool** before declaring the stage done. Common defects: label overflow, label collisions (text from one element overlapping another), labels rendered at the wrong z-order, axis tick labels truncated.

**Quality bar.** Each figure should reproduce cleanly at Manning's text-block width (~6 inches at 300 DPI). Use a neutral publication palette: dark navy (`#2a2e3a`), charcoal (`#3a3e4a`), white background, plus *exactly one* accent color per figure.

---

## Stage 8 — Run `lrm-chapter-reviewer` agent

**Goal.** A senior-reviewer-level pass over the whole chapter.

**Action.** Spawn the dedicated reviewer:

```text
Agent(
  subagent_type="lrm-chapter-reviewer",
  description="Review chapter <N> manuscript",
  run_in_background=true,
  prompt=<self-contained brief that includes paths to the manuscript, the structure plan,
          the research dossiers, the Manning style file, the MQR file, the proposal, and
          the most-recently-shipped chapter's manuscript folder as a voice reference>,
)
```

The reviewer returns a single comprehensive report. Expect: a verdict ("publish-ready" / "minor" / "major"), an issue list per section tagged BLOCKER/MAJOR/MINOR/NIT, a "what this chapter does particularly well" section, and a final go/no-go.

---

## Stage 9 — Incorporate reviewer feedback

**Goal.** Resolve every BLOCKER and every MAJOR. Defer MINORs and NITs only with written reason.

**Action.** Apply the reviewer's fixes the same way as Stage 6. Pay special attention to:
- Factual claims about specific named systems, papers, action-space numbers, and hardware-precision claims (e.g., bf16 / fp16 / Tensor-Core support on specific GPU generations). These are the easiest things to get wrong and the easiest things for an alert reader to spot.
- Listing/figure/table number consistency between the manuscript and the structure plan.
- Callout formatting — every callout must be a single blockquote starting with `> **LABEL — title.**`. The DOCX converter depends on this exact shape.

If the user said code listings would be filled in a later pass, do not block on the listing-placeholder BLOCKERs that the reviewer flags — note them and move on. Everything else is in scope.

---

## Stage 10 — Convert to Manning-styled DOCX

**Goal.** A single `chapter_<N>.docx` that adopts the template's Title/Subtitle/Heading 1-6 styles, embeds all figures inline, renders tables with borders, and is ready to drop into Manning's Google Docs pipeline.

**Action.**
1. Write `chapter_<N>/manuscript/_build_docx.py` — a python-docx converter. The most-recently-shipped chapter's `_build_docx.py` is the canonical reference; copy it and only change paths.
2. The mapping the converter must implement:

| Markdown construct | DOCX style |
|--------------------|------------|
| `# <N> <Chapter Title>` (first H1) | Title = `"<N>"`, Subtitle = `"<Chapter Title>"` |
| `## <N>.1 …` | Heading 1 |
| `### <N>.1.1 …` | Heading 2 |
| `#### …` | Heading 3 |
| `**Listing X.Y** …` | Heading 4 (caption above code block) |
| `**Figure X.Y** …` | Heading 6 (caption above embedded image) |
| `**Table X.Y** …` | Heading 5 (caption above the table) |
| ``` fenced code | normal paragraph, Courier New 9pt, light grey shading |
| `> **LABEL — title.** body` | normal paragraph with bold lead + parchment shading |
| Bullet `- item` | normal paragraph, indented, `•` lead |
| Markdown table | docx table with manual single-line `888888` borders |

3. After writing the converter, **always inspect the resulting DOCX** with python-docx: print the counts of each Heading style and the number of inline images. Sanity-check magnitudes: 1 Title, 1 Subtitle, one Heading 1 per `X.Y` section, one Heading 4 per listing, one Heading 5 per table, one Heading 6 per figure, and one inline image per figure caption.

**Known template constraints:**
- The template **does not** define `"List Bullet"` or `"Table Grid"` styles. Don't try to apply them by name — render bullets as plain paragraphs with a `•` lead, and apply table borders manually via `OxmlElement("w:tblBorders")`.
- The Custom Menu macros that Manning's template provides (Insert Callout, Insert Sidebar) are Google Docs script — python-docx cannot invoke them. Use blockquote-shaded paragraphs as the printable equivalent.

---

## Pipeline diagram

```text
proposal/proposal.md ──┐
book_learnings.md     ─┤                          ┌─ chapter_<N>/resources/*.md       (Stage 1)
                       ├── Stage 1: Research ─────┤
seed sources from user ┘                          └─ verified arXiv citations
                                                                                       │
                                                                                       ▼
                       Stage 2: Synthesize learnings → patterns to apply ─────────────┐│
                                                                                       │
                                                                                       ▼
proposal/proposal.md + plan ⇄ Stage 3: Structure & plan ─→ chapter_<N>_structure_and_plan.md
                                                                                       │
                                                                                       ▼
most recent chapter (voice) + writing_instructions/ ──→ Stage 4: Draft ───→ chapter_<N>.md
                                                                                       │
                                                                                       ▼
                              Stage 5: Section subagents ─→ N review reports
                                                                                       │
                                                                                       ▼
                              Stage 6: Apply fixes ────────→ chapter_<N>.md (revised)
                                                                                       │
                                                                                       ▼
                              Stage 7: Figures (drawio agents → PIL fallback) ─→ figures/diagrams/*.png
                                                                                       │
                                                                                       ▼
                              Stage 8: lrm-chapter-reviewer ─→ comprehensive report
                                                                                       │
                                                                                       ▼
                              Stage 9: Apply reviewer feedback ─→ chapter_<N>.md (final)
                                                                                       │
                                                                                       ▼
chapter_template/chapter_template.docx ─→ Stage 10: _build_docx.py ─→ chapter_<N>.docx
```

---

## Failure modes to expect (and what to do)

1. **Subagent permission denials on skills.** Spawned `general-purpose` subagents frequently have the `Skill` tool, `Bash`, or specific skills denied. When this happens, the agent will halt and ask. Re-prompt them with the Write-only fallback (have them produce the artifact in plain markdown or have them write an XML/Python file you can render from the main session), or do the step in the main session yourself.
2. **Listing/figure number drift between the plan and the manuscript.** Reconcile via the listing summary table in the plan. If the manuscript adds a small listing that the plan didn't enumerate, update the plan — don't silently renumber the manuscript.
3. **Heading-pair callouts.** `#### Pitfall\n\n#### Title\n\nbody` is the wrong format and the DOCX converter will not shade it. Convert to blockquote.
4. **Hardware-precision claims.** Cross-check claims about bf16 / fp16 / Tensor-Core support against the specific GPU generation before quoting numbers. Turing-and-earlier GPUs do not support bf16; Ampere and newer do.
5. **Figure label overflow.** Always read the PNG (using the Read tool) before declaring Stage 7 done. The eye catches what code paths don't.

---

## Time budget (rough, per chapter)

- Stage 1: 30-60 min wall clock (research agents run in background).
- Stage 2: 10 min.
- Stage 3: 30-60 min (brainstorming + writing the plan).
- Stage 4: 2-4 hours (the actual drafting).
- Stage 5: 20 min wall clock (parallel subagents).
- Stage 6: 30-60 min.
- Stage 7: 30 min if subagents work, 60-90 min if PIL fallback is needed.
- Stage 8: 15-20 min wall clock (reviewer runs in background).
- Stage 9: 30-90 min.
- Stage 10: 30 min including converter authoring and DOCX inspection.

End-to-end: a focused day per chapter once the recipe is internalized.

---

## When to invoke this skill

- Starting a new chapter draft from the proposal's TOC.
- Producing a second draft after substantive scope changes — repeat Stages 3–10.
- Producing a partial-chapter writeup for a single section — collapse to Stages 1, 4, 7, 10 against that section's scope.

Do **not** invoke this skill for:
- Copy-edit-level fixes on an existing draft (use `/book lint` directly).
- Generating standalone figures or listings outside a chapter context.
- Sending the chapter to Manning — that's a separate workflow (Google Docs upload, Erik's review, AU/PR cycles) and is not in scope here.
