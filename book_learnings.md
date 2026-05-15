# Generic Body-Chapter Learnings — How Other Books Do It

> **Purpose.** This is a *generic* drafting reference for **Chapter 2 onward** of *Build a Large Robot Model (From Scratch)*. It distills body-chapter craft from three sample books in `other books/`. (For Chapter 1 / intro chapter conventions, see the separate `chapter_1/resources/other_books_chapter_1_learnings.md`.)
>
> **Books analyzed:**
> 1. Sebastian Raschka, *Build a Reasoning Model (From Scratch)* (Manning, v5 MEAP, 363 pp.) — the closest model for our coding-heavy chapters (Ch 2–7).
> 2. Chi Wang & Donald Szeto, *Designing Deep Learning Systems* (Manning, 2023, 362 pp.) — the closest model for our systems-leaning chapters (Ch 9 sim-to-real, Ch 10 deployment, Ch 11 what's next).
> 3. Nathan Lambert, *RLHF Book* (self-published, 229 pp.) — the closest model for our math-heavy stretches (Ch 5 flow matching, Ch 7 RL with REINFORCE/PPO/GRPO). Not a Manning book; mine its math-exposition tactics but soften the density and add Manning-style callouts/summaries.
>
> Use this as a checklist when drafting any technical body chapter. The cross-book consensus patterns appear in **§1**; per-archetype patterns are in **§2–4**; the consolidated body-chapter template is in **§5**.

---

## 1. Cross-Book Consensus Patterns (Use These by Default)

These conventions show up in **all three** books (or the two Manning ones, where applicable). Treat them as non-negotiable defaults unless a chapter has a specific reason to break them.

### 1.1 Chapter opening — same three-beat structure across all three

1. **Chapter number + short title on its own page**, no subtitle.
2. **2–4 motivating paragraphs of prose** that (a) connect back to the prior chapter, (b) state the pain-point the chapter resolves, (c) preview where it leads. Raschka explicitly frames "what wise decision the reader is making"; Wang/Szeto use the same move; Lambert uses a more research-survey "role-in-pipeline" framing.
3. **A boxed "This chapter covers" bullet list of 3–5 items**, items phrased as gerunds or verb phrases ("Understanding…", "Designing…", "Building…", "Introducing…"). It appears *after* the motivating prose, not before — the prose flows around it.

Both Manning books then add:

- A **section-by-section preview paragraph** ("In section 2.1 you will learn… In section 2.2 we will demonstrate…").
- A **roadmap figure** (always `Figure X.1`) — Raschka re-uses the book-wide stages diagram in every chapter with the current stage highlighted. Strong reader-trust device.
- A **state-restoration ritual in the first numbered section** ("Loading a pre-trained model", "Setting up the environment") that re-instantiates whatever the prior chapter built.

### 1.2 Sections, nesting, and length

- **Three levels of numbered nesting** (`X.Y`, `X.Y.Z`). A fourth level is forbidden; books use **ALL-CAPS run-in headings** instead (Wang/Szeto: "PRINCIPLE 1: …", "HOW TO USE", "WHEN TO USE"; Lambert: bold-prefixed paragraphs).
- **Chapter length: ~15–25 pages typical**, 8–13 numbered sections, average section 3–8 pages.
- **Every section ends with a one- to two-sentence bridge** to the next section ("In the next section, we will…").

### 1.3 Voice

- **First-person plural "we" dominates** (~80%), second-person "you" for direct instructions (~20%), occasional first-person singular "I" reserved for explicit authorial choices ("I chose Qwen3 because…", "In my experiments…"). Lambert is comfortable with stronger "I" when he has direct authority.
- **Present tense throughout**, including when describing what figures show and what code does.
- **No hype words** ("revolutionary", "powerful"). No emoji in body text. When stakes are real, matter-of-fact phrasing makes them more credible.
- **Uncertainty is named explicitly**, not hidden ("It's not even clear that such a question can be definitively answered" — Raschka; "as of writing", "research is still developing" — Lambert).

### 1.4 Honest scoping is restated multiple times per chapter

All three authors explicitly disclaim what they will *not* cover, often more than once per chapter. Wang/Szeto's "we recommend you check out Appendix B"; Raschka's "outside the scope of this book… see Appendix A"; Lambert's "see [137] for full derivation". This is the single biggest reader-trust device across all three books — adopt it aggressively.

### 1.5 Summary at chapter end

Every chapter ends with a `Summary` section (Raschka and Wang/Szeto: bulleted, ~7–15 bullets, each a complete sentence stating a claim; Lambert: usually a "Looking Ahead" paragraph plus a "Further Reading" bullet list — softer than Manning style). **The final bullet forward-references the next chapter** in both Manning books. Wang/Szeto put forward references in the *opening* preview paragraph instead of the summary; we'll follow Raschka's pattern and put them in both.

---

## 2. Patterns for Coding-Heavy Chapters (Model: Raschka)

Apply these in **Ch 2 (simulation & control), Ch 3 (VLA backbone), Ch 4 (discrete BC), Ch 5 (flow matching), Ch 6 (LoRA curriculum), Ch 7 (RL)**.

### 2.1 Code listings (Manning house style)

- **Numbered, captioned**: `Listing 3.6 Normalizing extracted answers`. Caption is a noun phrase, not a full sentence.
- **Lead-in sentence** introduces every listing: "The normalization step shown in figure 3.7 is implemented via the `normalize_text` function in listing 3.6." The function name, purpose, and figure cross-reference all appear before the code.
- **Inline annotation labels** instead of full-sentence comments. Two equivalent house styles:
  - Raschka uses `#A`, `#B`, `#C` markers inside the code with a small legend block immediately before the listing:
    ```
    #A LaTeX formatting to be replaced
    #B Strip chat special tokens
    ```
  - Wang/Szeto use **right-side annotation callouts** (separate labels with arrows pointing to specific lines).
- **Listing size: 8–30 lines** by default. Anything over 40 lines is broken across pages or pushed to the GitHub repo with a link.
- **`# NEW` markers when modifying an earlier listing** — wrap the diff so readers can see exactly what changed.
- **Expected-output blocks** after almost every code execution. Output is shown verbatim (token/sec numbers, generated text, tensor shapes). Call out anything hardware-dependent ("output above was generated on a CPU — exact wording may vary").

### 2.2 The "let's run it" micro-loop (the spine of every section)

Raschka's section spine fires 4–8 times per section:

> *describe → listing → "Let's see this function in action:" → call → printed output → one-paragraph interpretation → bridge to next step*

Negative examples are demonstrated **before** they are fixed. The reader sees the limitation, then the extension. Worth copying verbatim.

### 2.3 Incremental code building

- **Skeleton → run → inspect → extend.** Each new version is presented as a **diff** described in prose: "is almost identical to listing 2.2, except for the KV cache-related changes highlighted in the comments".
- Same idea sometimes presented **twice**: full `nn.Module` class for completeness, *and* a 3–5 line snippet showing just the key idea (Lambert's dual-presentation move — copy for math-heavy chapters too).

### 2.4 Tensor shape reasoning in prose, not symbols

"The resulting logits (step 2 in figure 5.12) have shape `[sequence_length, vocab_size]` and are then converted into normalized probability distributions using `torch.softmax` (step 3)…" Shapes appear in monospace brackets, integrated into sentences. Use this for our VLA hidden states (`[B, T, d_embed]`), action tensors (`[B, H, D]`), and tokenized actions (`[B, H, D, n_bins]`).

### 2.5 Sanity checks build trust

- After every meaningful change: **"Importantly, we also see that the generated text is the same as before, which is an important sanity check to ensure that the KV cache is implemented correctly."**
- **Hardware comparison tables** (e.g., Raschka's Table 2.1: tokens/sec across Mac M4 CPU, M4 GPU, H100). For us: training time / FPS / closed-loop SR across Colab T4, RTX 4090, A100. Calibrates reader expectations against their setup.
- **Pre-emptive error notes**: "If you are using a Mac with Apple Silicon and encounter `InductorError`, please use PyTorch 2.9 or newer."
- **Reassurance after dense content**: "If this overview feels complicated on first reading, don't worry. We will build up the algorithm step by step."

### 2.6 Exercises

- **3–5 per chapter**, placed **inline as boxed sidebars** at natural pause points (not collected at chapter end).
- Titled `EXERCISE N.M: SHORT NAME`. Typically 4–10 lines: one paragraph stating the task + 1–2 tips.
- **Difficulty range: light.** Most are "modify function X to also do Y" (swap loss, retry with different hyperparam, run on different hardware).
- **Solutions deferred to an appendix.**

---

## 3. Patterns for Math-Heavy Stretches (Model: Lambert)

Apply these in **Ch 5 §5.2–5.6 (flow matching derivations) and Ch 7 §7.3 (REINFORCE → PPO → GRPO)**.

### 3.1 Open math chapters with a "Notation" callout

Lambert's first move in every math chapter: "Throughout this chapter, we use x to denote prompts and y to denote completions." Adopt this as a formal **NOTATION** callout for our flow-matching and RL chapters. He also points readers to a one-page **"RL Cheatsheet"** companion — a downloadable summary of every loss derived in the chapter. We should produce equivalent cheat-sheet pages.

### 3.2 Derivation rhythm: one transformation per paragraph

The dominant pattern is **one symbolic move per paragraph**, with the move *named* ("log-derivative trick", "rearranging", "from chain rule"):

> "Notice that we can use the log-derivative trick to rewrite the gradient of the integral as an expectation:" → display eq. → "Using this log-derivative trick:" → display eq. → "Where the final step uses the definition of an expectation…"

After 4–8 equation derivations, insert a **bulleted simplification note**: which terms vanish, which survive, and why. This converts symbolic manipulation into checklist logic.

### 3.3 Mandatory "in plain English" recap after every derivation

This is the most copy-worthy Lambert move. After the policy-gradient derivation:

> "Reaching this equation is a crucial point. We have gone far enough to see that the gradient of the trajectory distribution reduces to a sum of gradients from language-model policy probabilities (which are just the probabilities of tokens given by the model we're training)."

After a Greek-letter expression, reframe it in English: "the gradient says (1) which direction in parameter space makes action `a_t` more likely, weighted by (2) how good the outcome was."

### 3.4 Tight equation-to-code coupling

Right after the equation that defines a loss, show the 3–5 line PyTorch realization:

```python
seq_log_probs = (token_log_probs * completion_mask).sum(dim=-1)
loss = -(seq_log_probs * advantages).mean()
loss.backward()
```

This is the strongest single technique for an MQR audience — math → minimal code → full class. Apply for every loss we derive (CE in Ch 4, conditional flow-matching in Ch 5, REINFORCE/PPO/GRPO in Ch 7).

### 3.5 Algorithm presentation

Lambert uses prose-with-cases rather than formal Algorithm-1 boxes. **We should be more formal**: present REINFORCE, PPO-step, GRPO-step as Manning-style numbered **Algorithm callouts** with explicit inputs, outputs, and step list. The MQR (basic-DL background, no RL) needs the extra scaffolding.

### 3.6 One worked numerical example per algorithm

Lambert under-does this; we should over-do it. For each algorithm, show one tiny end-to-end numerical pass with three trajectories or three actions, all scalars, computed by hand. The reader should be able to reproduce every number in a notebook in <5 minutes.

### 3.7 Question-form recap bullets after comparison sections

Lambert's gem from §5.7:

> - **RM:** "How good is this whole answer?" → scalar value
> - **ORM:** "Which parts look correct?" → per-token correctness
> - **PRM:** "Are the reasoning steps sound?" → per-step scores
> - **Value:** "How much reward remains from here?" → baseline for advantages

Steal this pattern for Ch 7:

> - **REINFORCE:** "How good was this trajectory?" → high-variance gradient.
> - **PPO:** "Did this update step go too far?" → clipped trust region.
> - **GRPO:** "How does this rollout compare to its siblings?" → relative advantage.

### 3.8 Honest deference

When a derivation is too long, **defer with a specific citation**: "see Lipman et al. [X] for the full derivation". Don't pad with half-derivations. Lambert's "research is still developing on the fundamentals here" is the right register.

### 3.9 Chapter triptych

Lambert's math chapters use a clean **theory → implementation → auxiliary** three-act layout. Implementation gets its own numbered section (~30–40 % of chapter length). Apply for Ch 5 (theory of vector fields → implement flow head → conditioning tricks) and Ch 7 (policy-gradient theory → implement PPO update → GAE and entropy bonus).

---

## 4. Patterns for Systems-Leaning Chapters (Model: Wang & Szeto)

Apply these in **Ch 9 (sim-to-real), Ch 10 (deployment), Ch 11 (what's next)**.

### 4.1 First numbered section is always conceptual

"Understanding X" or "X: Design overview" comes before any implementation. Concepts → principles → component diagrams → code/configs.

### 4.2 Diagram styles to borrow

| Style | When to use | Example for our book |
| --- | --- | --- |
| **Reference-architecture diagram** with boxes (A)–(G) and arrows | The chapter's anchor visual, revisited multiple times | Full VLA inference pipeline on Jetson Nano (Ch 10) |
| **Before/after diagrams** | Motivate a design choice | Closed-loop with vs without temporal ensembling (Ch 4) |
| **Component-with-internals** + numbered arrows (1)–(7) | Walk through a request flow; enumerated in prose immediately after | 50 Hz inference path: camera → tokenizer → backbone → action head → motor command (Ch 10) |
| **Same canonical pipeline redrawn with progressive complication** | Show how a stack accretes complexity | Sim → Sim+DR → Sim+DR+RL → Real-robot loop (Ch 9) |
| **Topology variations on a fixed template** | Present 3+ deployment options compactly | On-device vs cloud vs hybrid VLA deployment (Ch 10), canary vs blue-green model rollout |

Aesthetic: **monochrome, box-and-arrow, no shaded backgrounds, no icons.** Engineering-doc, not marketing-deck. Captions interpret the diagram, never just label it.

### 4.3 Triptych tool surveys

When surveying tools/frameworks (e.g., sim-to-real frameworks, quantization toolchains, VLA codebases), use Wang/Szeto's parallel-structure triptych: each tool gets identical subsections (`HOW TO USE` / `PARALLELIZATION` / `WHEN TO USE`), so the reader does the comparison via structure rather than a comparison table.

### 4.4 Explicit defaults and recommendations

Wang/Szeto: "We recommend using Ray Tune over other HPO libraries for the following five reasons: (1)…". Numbered recommendations are explicit. **Always give the MQR a recommended default path**; they often lack the context to pick on their own.

### 4.5 Tradeoff prose patterns

- **"On the flip side, these benefits come at a cost…"** with a concrete number.
- **"A drawback to this approach is…"** at the end of each option in a triptych — exactly one paragraph of upside + one of downside.
- **Symmetric "When to use X" / "When to build your own" subsections** for buy-vs-build decisions.

### 4.6 Real-world systems references stay inline (no case-study sidebars)

Vendors/papers are name-dropped in paragraph form with concrete numbers anchoring abstract claims. For us: "OpenVLA runs at ≈6 Hz on an A100; π0-FAST reaches 50 Hz on the same hardware [arXiv X]." Use **Manning short-link style** (`http://mng.bz/...`) or arXiv abs links inline; reserve a "Further reading" list at chapter end for heavier surveys.

### 4.7 Wang/Szeto omit exercises; we won't

The systems book delegates hands-on work to GitHub repo "labs". We're committed to per-chapter exercises everywhere — for Ch 9–11 the exercises can lean toward **"lab walkthroughs"** (deploy this notebook, profile this rollout, run this nvidia-smi command) rather than coding extensions.

---

## 5. Callout / Sidebar Taxonomy (Unified)

Use these consistently across the book. Names are uppercase to match Manning house style.

| Callout | Length | When to use | Source pattern |
| --- | --- | --- | --- |
| **NOTE** | 1–4 sentences | Scope, dependencies, forward refs ("install SymPy via `uv pip install sympy`") | Raschka + Wang/Szeto |
| **TIP** | 1 line | Small gotcha or convenience ("the leading whitespace in the output appears because…") | Raschka |
| **DEFINITION** | 4–8 lines | First introduction of a robotics or ML term (proprioception, end-effector, DoF, GRPO) | Raschka + Wang/Szeto |
| **DEEP DIVE** | 10–20 lines | Optional theory the running text can skip ("for readers comfortable with mathematical notation…") | Raschka |
| **COMPARISON** | 5–15 lines | Disambiguate from a previously introduced technique ("how this differs from…") | Raschka |
| **NOTATION** | 4–8 lines | Chapter-opening symbol legend in math-heavy chapters | Lambert |
| **ALGORITHM** | numbered, titled | Formal pseudocode block with inputs/outputs | (we add this; Lambert doesn't) |
| **PITFALL** | 3–6 lines | Common reader mistake ("argmax on a bimodal distribution lands in the saddle…") | (we add this; Raschka has informal versions) |
| **EXERCISE** | 4–10 lines, inline | Reader's hands-on extension | Raschka |

What we *won't* use: WARNING (overlaps NOTE), CASE STUDY (none of our models do this in sidebars; stay inline).

---

## 6. Figure Strategy (Unified)

### 6.1 Density

- **Coding-heavy chapter:** ~1 figure per 2–3 pages (Raschka density). High.
- **Math-heavy section:** sparser, ~1 figure per 5 pages (Lambert density), but each figure carries more weight (vector-field visualization, advantage-distribution histogram).
- **Systems-leaning chapter:** ~1 figure per 3–4 pages (Wang/Szeto), with the architecture diagram as a recurring anchor.

### 6.2 Recurring figure types to use

1. **Book-wide roadmap (Figure X.1)** — re-used in every chapter with the current stage highlighted. Defined once, reused everywhere.
2. **Architecture / data-flow diagrams** — boxes with labeled arrows; numbered when prose enumerates the path.
3. **Tensor-shape / worked-numeric examples** — Raschka's `[seq_len, vocab_size]` and "five-word vocab" mini-walks. Show actual numbers flowing through ops.
4. **Before/after concept illustrations** — classical vs VLA, MSE vs categorical, BC vs BC+RL.
5. **Pipeline-with-progressive-complication** — same pipeline redrawn 3 times with one new component added each time.
6. **Topology variation triptych** — three near-identical diagrams differing in one component.
7. **Loss / metric curves** with annotated regions ("here entropy collapses", "here SR plateaus").
8. **Hardware/benchmark tables** — calibrate expectations across Colab T4 / RTX 4090 / A100.

### 6.3 Captions

- **2–4 sentences**. The first sentence states what the figure shows; the next 1–3 interpret it.
- Self-contained: a reader skimming captions alone should be able to follow the chapter.
- The figure is always referenced **by number** in the paragraph *preceding* it *and* in the paragraph *following* it.

---

## 7. Citations & References (Unified)

- **Inline parenthetical citations**, not numbered markers. Author + year + arXiv abs link or Manning short link.
- **No footnotes** in body text.
- A short **"Further reading"** bullet list at chapter end for papers we want readers to know about but didn't cite inline (mirrors Lambert's pattern; Wang/Szeto skips this; Raschka uses Appendix A).
- Lower citation density than Lambert in narrative prose — push survey-style references into the Further Reading list.

---

## 8. Consolidated Body-Chapter Template (Checklist)

Use this as the drafting checklist for **every** body chapter (Ch 2 onward):

**Opening**
- [ ] Chapter number + short title on its own page.
- [ ] 2–4 paragraphs of motivating prose: recap prior chapter → state pain point → preview destination.
- [ ] Boxed "This chapter covers" — 3–5 gerund bullets.
- [ ] Section-by-section preview paragraph.
- [ ] Figure X.1 = book-wide roadmap with current stage highlighted.

**Structure**
- [ ] 8–13 numbered sections, max 3 levels of nesting (use ALL-CAPS run-in headings beyond that).
- [ ] First numbered section restores state from prior chapter ("Loading the model from Ch 3", "Setting up the simulator").
- [ ] Each section ends with a 1–2 sentence bridge to the next.

**Code (for code-heavy sections)**
- [ ] Numbered `Listing X.Y` blocks of 8–30 lines.
- [ ] Lead-in sentence before each listing naming function + purpose + figure cross-ref.
- [ ] `#A`/`#B` annotation labels with legend block before the listing.
- [ ] `# NEW` markers when modifying an earlier listing.
- [ ] Expected-output block after every code execution, with hardware-dependence callout if relevant.
- [ ] "Let's run it" micro-loop (describe → listing → run → output → interpret → bridge) firing 4–8× per section.

**Math (for math-heavy sections)**
- [ ] NOTATION callout near chapter top.
- [ ] One transformation per paragraph, each move named ("log-derivative trick", "interpolation by linearity").
- [ ] Bulleted "what vanishes / what survives" after 4–8 chained equations.
- [ ] Mandatory "in plain English" recap paragraph after every derivation block.
- [ ] Tight equation → 3–5 line PyTorch snippet coupling.
- [ ] Algorithm callouts (numbered, titled) for REINFORCE/PPO/GRPO/flow-matching update.
- [ ] One worked numerical example per algorithm (scalars, by hand).
- [ ] Question-form recap bullets at end of comparison sections.

**Systems (for systems-leaning sections)**
- [ ] First subsection is conceptual ("Understanding X" / "X: Design overview") before any implementation.
- [ ] Reference-architecture diagram as the chapter's anchor visual.
- [ ] Component-with-numbered-arrows diagrams mirrored by numbered prose.
- [ ] Triptych structure for tool/framework surveys (parallel `HOW TO USE / WHEN TO USE` subsections).
- [ ] Explicit defaults / recommended path stated up front.
- [ ] Tradeoffs: upside paragraph + downside paragraph per option.

**Callouts**
- [ ] 3–6 callouts per chapter, tagged from the §5 taxonomy.
- [ ] At least one PITFALL per chapter.
- [ ] Honest scoping disclaimer ≥ 2× per chapter ("from scratch refers to X, not Y"; "we defer the full derivation to [paper]").

**Figures**
- [ ] ~1 figure per 2–3 pages (coding) / per 3–5 pages (math/systems).
- [ ] Captions 2–4 sentences, self-contained, interpretive.
- [ ] Each figure referenced by number in the paragraphs immediately before and after.
- [ ] Hardware/benchmark table near the end of the chapter where appropriate.

**Exercises**
- [ ] 3–5 inline EXERCISE boxes at natural pause points.
- [ ] Difficulty light → moderate; solutions deferred to an appendix.

**Closing**
- [ ] `Summary` section: 8–15 bulleted claim-style sentences, the last forward-referencing the next chapter.
- [ ] Short `Further reading` list (5–15 bullets) for non-cited papers/codebases.

**Voice**
- [ ] Present tense, ~80 % "we" / 20 % "you", occasional "I" for explicit authorial choices.
- [ ] No hype words; explicit uncertainty where warranted.
- [ ] Honest scoping at least twice.

---

## 9. Anti-Patterns to Avoid

- **Don't write a literature survey** in body chapters. Cite key systems, don't catalog them — push exhaustive surveys to Further Reading.
- **Don't start sections with code.** Concept → motivation → diagram → listing.
- **Don't drop listings >40 lines** without breaking them up; link the full file in the repo.
- **Don't use a fourth level of numbered nesting** (`X.Y.Z.W`) — switch to ALL-CAPS run-in headings.
- **Don't put forward references only in the summary** — put them in the opening preview too (Raschka's pattern).
- **Don't omit expected output.** A listing without an output block fails the "let's run it" rhythm.
- **Don't use Algorithm-by-prose** for non-trivial math (Lambert's main weakness for our audience) — use formal numbered Algorithm callouts.
- **Don't fabricate certainty.** Match Lambert's "as of writing", Raschka's "open research question". Date claims that may rot ("OpenVLA runs at ≈6 Hz on an A100 as of mid-2024").
- **Don't oversell.** All three authors are matter-of-fact even about impressive results. Hype is a trust leak.

---

## 10. Source-of-Patterns Quick Lookup

| Pattern | Primary source | Strength |
| --- | --- | --- |
| Three-beat chapter opener | Raschka + Wang/Szeto (consensus) | High |
| Roadmap Figure X.1 reused per chapter | Raschka | High |
| State-restoration first section | Raschka + Wang/Szeto | High |
| `Listing X.Y` + `#A`/`#B` annotation labels | Raschka (Manning) | High |
| Right-side annotation callouts on listings | Wang/Szeto (Manning) | High |
| "Let's run it" micro-loop | Raschka | Very high |
| `# NEW` diff markers | Raschka | High |
| Hardware benchmark tables | Raschka | High |
| Inline EXERCISE boxes, solutions in appendix | Raschka | High |
| NOTATION callout for math chapters | Lambert | High |
| One transformation per paragraph in derivations | Lambert | Very high |
| "In plain English" recap after derivations | Lambert | Very high |
| Equation → 3–5 line code coupling | Lambert | Very high |
| Question-form recap bullets | Lambert | High |
| Honest deference ("see paper X for full derivation") | Lambert | High |
| Triptych tool survey | Wang/Szeto | High |
| Numbered-arrow flow diagrams mirrored by numbered prose | Wang/Szeto | Very high |
| Same-pipeline-with-progressive-complication diagrams | Wang/Szeto | High |
| Topology variation triptych | Wang/Szeto | High |
| Inline vendor/paper references with concrete numbers | Wang/Szeto | High |
| Claim-style bulleted Summary | Raschka + Wang/Szeto | High |
| Forward reference in final summary bullet | Raschka | High |
| "Further reading" bullet list | Lambert | Medium |
| Present tense + ~80/20 we/you voice | All three | High |
| ≥2 honest-scoping disclaimers per chapter | All three | Very high |
