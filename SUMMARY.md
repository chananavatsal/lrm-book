# Build a Large Robot Model (From Scratch) — Project Summary

Last updated: 2026-05-15

This file is the single source of truth for **where everything lives, which automation is available, and what state each chapter is in**. Read this first when picking up the project after time away or when onboarding a new collaborator (human or AI).

---

## 1. Context

A Manning Publications book teaching a working software engineer / ML practitioner / classical roboticist how to **build a complete Vision-Language-Action (VLA) policy from scratch** on a single consumer GPU. Eleven chapters across five parts; everything from raw demonstration data through INT8 deployment on a Jetson.

- **Authors:** Siddharth Singh, Vatsal Chanana, Krishnam Gupta.
- **Editor:** Erik Pillar (Manning).
- **Audience:** the Minimally Qualified Reader profile at `mqr/mqr.md` — intermediate Python, basic PyTorch, willing-to-learn, no prior robotics, no industrial hardware, no compute cluster.
- **End state of the book:** the reader has a VLA backbone, both discrete and continuous action heads, a RL refinement loop, INT8/NF4 quantization, and an asynchronous 50 Hz control deployment running on a Jetson Nano.
- **Compute envelope:** single RTX 4090 or free-tier Google Colab. No exceptions.

The full proposal — including learning goals, chapter-level scope contracts, and competitive landscape — is at `proposal/proposal.md`. Any deviation from the proposal in any chapter must be a *deliberate* call, recorded in that chapter's structure plan.

---

## 2. Resource map

### Top-level reference files

| Path | What it is |
|------|------------|
| `proposal/proposal.md` | Authoritative TOC, learning goals, audience, scope contract. |
| `CLAUDE.md` | Project instructions auto-loaded by Claude Code (includes the audience description and proposal pointer). |
| `mqr/mqr.md` | Minimally Qualified Reader profile — what the reader has and lacks. |
| `writing_instructions/writing_instructions.md` | Manning's hard formatting rules (Heading 4 for listings, etc.). |
| `book_learnings.md` | Patterns harvested from Raschka's *LLMs From Scratch*, Lambert's *RLHF Book*, *Build a Reasoning Model*, *Designing Deep Learning Systems*. Apply during structure planning. |
| `STYLEGUIDE.md` | Locked authorial decisions (terminology choices, voice rules). |
| `index.html` | Public-facing project landing page. |

### Per-chapter folder convention

Every chapter follows the same layout (the `research_and_draft_chapter` skill creates this skeleton):

```text
chapter_<N>/
├── resources/                              # research dossiers + seed sources
├── chapter_<N>_structure_and_plan.md       # structural plan
├── structure.md                            # short index of the plan
├── manuscript/
│   ├── chapter_<N>.md                      # markdown draft
│   ├── _build_docx.py                      # python-docx converter
│   └── chapter_<N>.docx                    # Manning-styled deliverable
└── figures/
    ├── graphic_generation.md               # figure prompts
    └── diagrams/
        ├── _build_figures.py               # PIL renderer (fallback)
        └── figure_<N>_<k>.{drawio,png}
```

### Other top-level folders

| Path | What it is |
|------|------------|
| `chapter_template/` | The Manning DOCX template. Used by `_build_docx.py` as the style source. |
| `other books/` | PDFs + extracted `.txt` of reference books for research grounding. |
| `resources/` | Cross-chapter research material that doesn't belong to any single chapter. |
| `nanobanana-output/` | Image-generation experiments. |
| `agents/` | In-repo Claude Code skills (see §3). |
| `.claude/` | Claude Code project config — subagent definitions, settings, agent memory. |

---

## 3. Skills and agents available

### Skills (in `agents/`)

| Skill | What it does | When to use |
|-------|--------------|-------------|
| `agents/book/SKILL.md` (`/book` command) | Manning style lint, structure check, code-listing validation, simulated reviewers, MQR check, voice check, stylebook. | During and after drafting any chapter. Especially before sending anything to Erik. |
| `agents/research_and_draft_chapter/SKILL.md` | End-to-end 10-stage pipeline that produced Chapter 4. Research → plan → draft → review → figures → DOCX. | Starting a new chapter from scratch. |

### Agents (in `.claude/agents/`)

| Agent | Role | Invoke as |
|-------|------|-----------|
| `lrm-chapter-reviewer` | Senior technical book reviewer; checks structure, voice, MQR comprehensibility, Manning style; produces a graded BLOCKER/MAJOR/MINOR/NIT report. | `Agent(subagent_type="lrm-chapter-reviewer", ...)` once a chapter has reached "ready for final review" quality. |

### External skills used during the Chapter 4 pass

- `superpowers:brainstorming` — used at the start of Stage 3 (structure planning).
- `document-skills:docx` — used to inspect template styles and validate the final DOCX.
- `document-skills:pdf` — used to extract `.txt` from the books in `other books/` for research grounding.
- `draw-io:draw-io` — *attempted* for figure generation; subagents hit permission denials, so the PIL fallback in `chapter_<N>/figures/diagrams/_build_figures.py` is the documented path.

---

## 4. Chapter status

Status legend:
- **Pending** — no work started.
- **In progress** — research and/or planning underway; no manuscript yet.
- **Draft** — full markdown manuscript exists, reviewed at least once, figures rendered, DOCX produced. Code listings may still be placeholders.
- **Editor review** — DOCX submitted to Erik; awaiting his pass.
- **Revisions** — Erik's feedback applied; back to draft for re-review.
- **Final** — accepted by Manning.

| Ch | Title | Status | Notes |
|----|-------|--------|-------|
| 1 | The Generative Robot | **Editor review** | DOCX at `chapter_1/manuscript/Chapter 1_ The Generative Robot.docx`. With Erik. Voice/structure reference for every other chapter. |
| 2 | Simulation and Control | **In progress** | Research / planning underway. No manuscript yet. |
| 3 | Building the VLA Backbone | **In progress** | Research / planning underway. No manuscript yet. |
| 4 | Discrete Behavior Cloning | **Draft** | Full markdown manuscript at `chapter_4/manuscript/chapter_4.md`. 8 figures rendered. Manning-styled DOCX at `chapter_4/manuscript/chapter_4.docx`. **Code listings are still placeholders** — slated for the next pass. |
| 5 | Continuous Control with Flow Matching | **Pending** | |
| 6 | The Staged Curriculum Hypothesis | **Pending** | |
| 7 | Reinforcement Learning | **Pending** | |
| 8 | Reasoning for Robotics | **Pending** | |
| 9 | Sim-to-Real: Closing the Gap | **Pending** | |
| 10 | Efficient Deployment | **Pending** | |
| 11 | What's Next | **Pending** | |

### Chapter 4 — what remains before submission

1. Fill in the 12 listing bodies in `chapter_4/manuscript/chapter_4.md` (currently marked `[code placeholder — to be filled in next pass]`).
2. Re-run `_build_docx.py` to regenerate `chapter_4.docx` with the real listings.
3. Hand the regenerated DOCX to `lrm-chapter-reviewer` for a final pass.
4. Submit to Erik.

### Cross-cutting deliverables not tied to a single chapter

- **Appendices A-E** (per the proposal): pending. Author after Chapter 11 is in editor review.
- **GitHub companion repo** with runnable notebooks: pending; tracked separately.
- **Marketing / TOC summary for Manning's MEAP page:** pending.

---

## 5. How to use this file

- **Picking up the project after time away:** read §4 first, then drop into the chapter you intend to work on.
- **Starting a new chapter:** read the `research_and_draft_chapter` skill end to end before opening any files. The skill encodes the recipe; reading it cold prevents re-deriving the process for every chapter.
- **Adding a new skill or agent:** update §3 and `agents/README.md`.
- **Updating chapter status:** edit §4. Keep the status field constrained to the legend (`Pending` / `In progress` / `Draft` / `Editor review` / `Revisions` / `Final`).
- **Adding a new top-level reference file:** update §2.

This file is meant to stay short. If a section here grows past one screen, move the long content out to a dedicated file and leave a pointer behind.
