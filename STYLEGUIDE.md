# Style Guide: Build a Large Robot Model (From Scratch)

**Authors**: Siddharth Singh, Vatsal Chanana, Krishnam Gupta
**Publisher**: Manning Publications
**Editor**: Erik Pillar

This document is the team-wide locked style guide. All decisions here are binding across all 11 chapters and 5 appendices. Update only after team agreement.

This is the **contract** between the three co-authors. When in doubt, defer to this document.

---

## 1. Punctuation and Formatting

### Em dashes
**RULE: Never use em dashes (—).**

Replace with:
- Comma: "the dinosaur, not the truck"
- Colon: "three constraints: latency, continuity, safety"
- Period: "It worked. Until it didn't."
- Parentheses: "the model (RT-2) reached for the dinosaur"

This is a hard rule. No exceptions.

### Serial comma
Use it. "Vision, language, and action." Not "vision, language and action."

### Capitalization
- **Headings**: Sentence case. "The physical reality gap" not "The Physical Reality Gap"
- **Cross-references**: Lowercase. "as shown in figure 3.6" not "Figure 3.6"
- **Subtitle after colon in heading**: Capitalize. "Chapter 1: The generative robot"
- **Front matter**: Lowercase. "preface", "about this book"

### Quotation marks
- Curly in prose
- Straight in code
- Period and comma INSIDE quotes

### Numbers
- Spell out under 10 ("seven joints", "three constraints")
- Numerals for units ("50 Hz", "20 ms", "300 dpi")
- Numerals for versions ("Chapter 5", "Python 3.12")
- If mixing categories, use numerals for all if any is >= 10

---

## 2. Voice and Tone

### Person
- **"You"** for reader actions: "You will build a vision encoder."
- **"We"** for shared thinking: "We can think of action tokenization as..."
- **Never** "we" meaning "the authors". That's vague.
- **Never** "the user" or "one". Too formal.

### Tense
- Present tense preferred.
- "The model predicts the next action" not "The model will predict"
- Past tense only for historical facts ("RT-2 was released in 2023")

### Voice
- Active voice strongly preferred.
- "Google DeepMind built RT-2" not "RT-2 was built by Google DeepMind"

### Sentence rhythm
- Short sentences for key claims. Let them land.
- Longer sentences for explanation and qualification.
- Aim for variety. Avoid more than three long sentences in a row.

---

## 3. Banned Language

### Marketing words (never use)
- revolutionary, groundbreaking, cutting-edge, state-of-the-art
- game-changing, transformative, breakthrough
- exciting, incredible, amazing, awesome
- powerful, novel, elegant, remarkable, striking

Use instead: recent, effective, practical, useful, common, well-studied, established, notable.

### Meta-language (Erik's hard rule)
- "in this book"
- "in this chapter"
- "this chapter will cover"
- "we will see"
- "as we will discuss in chapter X"
- "later we cover"
- "Chapter X covers"
- "in the next chapter"

Chapter content should teach, not advertise. Cross-references to other chapters are banned in Chapter 1 and used sparingly elsewhere.

### Hedge words to minimize
- "very", "really", "quite", "actually", "basically", "essentially"
- These add no information. Cut them.

---

## 4. Terminology (Locked Decisions)

These terms are used consistently across all 11 chapters. Never deviate.

| Concept | Use | Don't Use |
|---------|-----|-----------|
| The full system | Vision-Language-Action model (VLA) | VLA policy, robot foundation model |
| The output component | action head | action decoder, action module, policy head |
| Two action approaches | discrete tokenization / continuous flow matching | autoregressive/generative, classification/regression |
| Learning from demos | behavior cloning | imitation learning (use BC specifically) |
| The training paradigm | imitation learning | learning from demonstration |
| Going beyond demos | reinforcement learning | RL fine-tuning, policy improvement |
| The vision component | vision encoder | image encoder, visual backbone |
| The language component | language backbone | language model, text encoder, LLM (in this book context) |
| The fusion step | multimodal fusion | cross-modal attention, feature fusion |
| Robot's body awareness | proprioception | joint state, robot state (use proprioception when introducing) |
| The output unit | motor command | action, joint command (use "motor command" in prose, "action" in code) |
| Joint count | degrees of freedom (DOF) | joints (use "joints" only after defining DOF) |
| Edge hardware | Jetson | NVIDIA Jetson, edge device (use "Jetson" specifically) |
| Real-world transfer | sim-to-real | sim2real, domain transfer |
| Variation training | domain randomization | data augmentation (in robotics context) |
| Efficient fine-tuning | LoRA | low-rank adaptation (define then use abbreviation) |
| Action prediction at multiple steps | action chunking | action sequence prediction |

When you need to introduce a synonym (e.g., "the policy" for "the model"), do it once in parentheses, then commit: "the VLA model (sometimes called a robot policy) outputs..."

---

## 5. Code Conventions

### Languages
- **PyTorch only**. No JAX, no TensorFlow.
- Python 3.12+ required (LeRobot 0.4.4 compatibility).

### Style
- Lines max 76 characters (max 55 if annotated)
- 4-space indents (no tabs)
- Straight quotes, never curly
- Type hints encouraged but not required for pedagogical clarity
- Imports at top of listing, no inline imports

### Naming
- `vision_encoder` not `ve` or `vis`
- `language_backbone` not `lm` or `llm`
- `action_head` not `ah`
- `fusion_transformer` not `ft`
- Constants: `EMBED_DIM = 512`, `NUM_BINS = 256`
- Configs: lowercase with underscores

### Listing structure
```python
import torch
from vla import VisionEncoder, LanguageBackbone, FusionTransformer, ActionHead

# Build components
vision = VisionEncoder.from_pretrained("siglip-base")
language = LanguageBackbone.from_pretrained("SmolLM-360M")
fusion = FusionTransformer(embed_dim=512, num_layers=6)
action_head = ActionHead(action_dim=7, num_bins=256)

# Forward pass
image = env.get_observation()                    #A
visual_tokens = vision.encode(image)             #B
lang_tokens = language.encode("pick up the cup") #C

fused = fusion(visual_tokens, lang_tokens)
action = action_head.predict(fused)              #D
env.step(action)

#A Camera observation as RGB tensor [3, 224, 224]
#B Patch embeddings from the vision encoder [196, 512]
#C Tokenized instruction from the language backbone [12, 512]
#D Predicted motor commands [7] for a 7-DOF arm
```

### Annotations
- Use `#A`, `#B`, `#C` at end of line
- Place explanations below the listing
- Each annotation is 1-2 sentences max
- Annotations describe what code DOES, not what it IS

---

## 6. Figures

### Captions
- Always a complete sentence (or several)
- Describe what is HAPPENING, not just label
- Bad: "Figure 1.1 VLA architecture"
- Good: "Figure 1.1 The VLA inference pipeline: a camera image and language instruction enter the model, are encoded into tokens, fused by the transformer backbone, and decoded into motor commands at 50Hz."

### Annotations on diagrams
- Short labels with action verbs, not paragraphs
- Under 10 words per annotation
- No periods
- Use #A, #B, #C step labels (Manning convention)
- Long explanations belong in the rich caption, not on the diagram

### Numbering and labels
- Use letters (#A, #B, #C) for step indicators in figures
- NOT cueball numbers (Manning convention)
- Number figures by chapter: Figure 1.1, Figure 1.2, ...

### Color
- Most readers see grayscale (print)
- Never refer to color in captions or text
- Use patterns, dashes, shapes for differentiation
- "The dotted line shows..." not "The red line shows..."

### File format
- Vector: SVG, EPS, or PDF (with editable text)
- Plus PNG for reference
- Filename: `CH01_F01_VLAArchitecture` (underscores, not spaces)
- Max size: 5.6 inches wide x 7 inches tall

### Tools
- Excalidraw or draw.io recommended
- Manning provides icon library

---

## 7. Chapter Structure

Every chapter must have:

1. **Opening bullets**
   - "This chapter covers" header
   - 3-5 bullets, max 8 lines, max 45 chars per line
   - Phrases not sentences
   - No gerunds like "Understanding", "Learning", "Discovering"

2. **Introduction** (2-6 paragraphs)
   - Compelling, story-driven if possible
   - Answers: what is this, why care, how does it work at a high level

3. **Numbered sections** (1.1, 1.2, etc.)
   - Each section has its own intro before subsections
   - Subsections (1.2.1) for natural breaks

4. **Mental model** (Chapter 1 only, REQUIRED)
   - 4-8 pages
   - Concrete scenario + diagram + annotations + rich caption + complete prose explanation

5. **Code listings** (every chapter except 1 and 11)
   - Numbered: Listing 4.1, Listing 4.2
   - Introduced in text before showing
   - Full annotation with #A, #B explanations

6. **Closing summary**
   - Bullets in complete sentences
   - Abstract, applicable takeaways
   - Not "we covered X" but "X is true because Y"

---

## 8. Author Coordination

### Chapter ownership (from the proposal)
- **Siddharth**: Chapters 1-3 (intro, simulation, VLA backbone)
- **Vatsal**: Chapters 4-8 (BC, flow matching, curriculum, RL, reasoning)
- **Krishnam**: Chapters 9-11 (sim-to-real, deployment, what's next)

### Cross-review
- Every chapter reviewed by at least one other author before submission to Erik
- Voice unification pass by single designated author per chapter

### Voice unification owner
- TBD at next sync. Recommendation: Krishnam volunteers for this role to maintain consistency.

### Communication
- Weekly sync (30 min)
- Slack/Discord channel for async
- Decision log for non-obvious choices

---

## 9. Production

### File locations
- Chapter drafts: Markdown in the repo (one folder per chapter), then transferred to Google Docs
- Code: Companion GitHub repo (TBD) for runnable listings
- Graphics: Box.com Graphics folder, organized by chapter
- Figures: Production-quality SVG/EPS in Box, plus PNG for reference

### Workflow
1. Outline in markdown
2. Draft prose in markdown
3. Use the `/book` skill in `agents/book/` for linting and review (optional, for Claude Code users)
4. Transfer to Google Docs with Manning template
5. Co-author review in Suggesting mode
6. Move to Manuscript folder when ready for Erik
7. Erik reviews, returns with comments
8. Address comments, resubmit

### MEAP cadence
- Manning expects monthly chapter releases once MEAP starts
- Build 1-week buffer per release
- Communicate slips to Erik immediately

---

## 10. Open Decisions (TBD)

These need to be resolved at the next co-author sync:

- [ ] Voice unification owner (lock who does final pass on each chapter)
- [ ] Companion code repo name and org
- [ ] Slack/Discord channel setup
- [ ] Tech reviewer recruitment (Charlie?)
- [ ] MEAP launch date
- [ ] Whether to commit to stretch goals (real robot deployment, additional appendices)

---

## How to update this document

This is the team contract. Before making changes:

1. Raise the proposed change at the weekly sync or in the team Slack
2. Get agreement from all three authors
3. Update this document
4. Commit with a message explaining the rationale
5. Note the change in the weekly sync

Unilateral changes to locked decisions (especially terminology) should be avoided. The point of the contract is that it's stable.

---

**Last updated**: 2026-04-10
**Maintainer**: Krishnam (until otherwise assigned)
