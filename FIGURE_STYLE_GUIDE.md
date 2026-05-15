# Figure Style Guide: Build a Large Robotics Model (From Scratch)

All figures across all 11 chapters and 5 appendices must follow this design system. The goal is that every figure feels like it belongs to the same book, regardless of which author created it.

## The SO-100 Robot Arm

A simplified line drawing of the SO-100 robot arm appears throughout the book as the consistent visual anchor. The same illustration is used every time, at the same angle (3/4 front view, slightly above), at appropriate sizes.

### Design rules
- Simple geometric line drawing, NOT photorealistic
- No face, eyes, or personality features
- 6 visible joints + gripper (matching real SO-100 anatomy)
- Consistent angle: 3/4 front view, arm slightly extended
- Must be recognizable at 0.5 inch width (thumbnail test)
- Three versions needed:
  - **Outline**: for complex diagrams where the arm is context, not focus
  - **Filled**: for emphasis when the arm is the subject
  - **In-action**: arm interacting with objects (grasping, pushing)

### When to use
- Ch 1: Mental Model figure (executing a task)
- Ch 2: simulation environment diagrams
- Ch 3: receiving motor commands from the VLA pipeline
- Ch 4-5: during training (demonstrations being recorded)
- Ch 6: curriculum training progression
- Ch 9: side-by-side sim vs real comparison
- Ch 10: on Jetson hardware, deployed
- Ch 11: future applications

### When NOT to use
- Pure architecture diagrams (component boxes and arrows only)
- Math/equation figures
- Training curves and loss plots
- Code output screenshots

---

## Component Visual Identity

Every time a VLA component appears in any figure across any chapter, it uses the same visual treatment. This ensures the reader recognizes "vision encoder" whether they see it in Ch 1, Ch 3, or Ch 10.

### Component boxes

| Component | Shape | Fill (grayscale) | Border |
|-----------|-------|-------------------|--------|
| Raw inputs (camera, text, state) | Rounded rectangle | White / very light gray | Thin solid |
| Vision encoder | Rectangle | Light gray (20%) | Medium solid |
| Language backbone | Rectangle | Light gray (20%) | Medium solid |
| State encoder | Rectangle | Light gray (20%) | Medium solid |
| Fusion transformer | Large rectangle | Medium gray (40%) | Heavy solid |
| Discrete action head | Rectangle | White | Medium dashed |
| Flow matching head | Rectangle | White | Medium dashed |
| Motor commands output | Rectangle | Light gray (20%) | Double border |
| Safety/monitoring | Rectangle | White | Dotted border |
| External systems (LeRobot, Gymnasium) | Rectangle | No fill | Thin solid |

### Labels inside boxes
- **Title**: Arial 8pt, bold. The component name. E.g., "Vision Encoder (SigLIP)"
- **Subtitle**: Arial 7pt, regular. Brief description or tensor shape. E.g., "[196, 512]"
- **No italics** in figure text (Manning rule)

### Locked component names in figures
Use these exact names. No variations.

| In figures | Not this |
|------------|----------|
| Vision Encoder (SigLIP) | ViT Encoder, Image Encoder |
| Language Backbone (SmolLM) | LLM Encoder, Text Encoder |
| Fusion Transformer | Multimodal Fusion, Cross-Attention |
| Discrete Action Head | Tokenizer, Classifier |
| Flow Matching Head | Diffusion Head, Continuous Decoder |
| State Encoder | Proprioception Encoder |
| Motor Commands | Actions, Joint Commands |

---

## Arrow Styles

| Arrow type | Visual style | Meaning | Example |
|------------|-------------|---------|---------|
| Solid, black | --------> | Data flow (tensors) | Image to vision encoder |
| Dashed, black | - - - - > | Alternative path | Discrete vs continuous |
| Dotted, black | . . . . > | Optional/conditional | Sim-to-real transfer |
| Thick solid | ========> | Primary/highlighted flow | The path being discussed |

### Arrow labels
- Place on or near the arrow, not at the destination
- Arial 7pt
- Describe what is flowing: "visual tokens [196, 512]" not just an arrow

---

## Annotation System

### Letter labels (#A, #B, #C)
- Place in top-left corner of each annotated component
- Arial 8pt, bold
- Sequential order follows the data flow (top to bottom, left to right)
- Same letter = same concept across figure and code listing in the same section

### Consistency between figures and code
If Figure 1.5 uses #A for "Camera Image" and Listing 1.1 uses #A for `env.get_observation()`, the reader should see that these are the same thing. Never let #A mean different things in a figure and its corresponding code listing.

---

## Layout Rules

### Pipeline diagrams (the most common type)
- **Flow direction**: top to bottom
- **Inputs at top**, outputs at bottom
- **Section headers on left side**: INPUTS, ENCODERS, FUSION, ACTION DECODING, OUTPUT (in caps, Arial 8pt bold)
- **Even vertical spacing** between sections
- **Parallel paths** (discrete vs continuous) shown side by side with dashed separator

### Comparison diagrams
- **Flow direction**: left to right
- **Old/classical on left**, new/LRM on right
- **Vertical dashed line** separating the two approaches
- **Bottom labels** summarizing each side's key property

### Training progression diagrams
- **Flow direction**: left to right (time axis)
- **Stages marked** with vertical separators
- **The SO-100 arm appears** at each stage showing progression

### System architecture diagrams
- **Layered**: inputs at top, processing in middle, outputs at bottom
- **Service boundaries** shown as large containers with components inside

---

## The Tabletop Scene

When showing the robot's workspace (the tabletop manipulation environment), use a consistent layout:

- Flat surface (table/desk), shown from a 3/4 elevated angle
- 2-4 simple objects: colored cubes, a mug, a small container
- The SO-100 arm positioned on the left side of the table
- Clean background, no clutter, no walls
- Objects are simple geometric shapes (easy to render consistently)

The default task shown in examples: "push the green cube left"

When the figure must be grayscale: objects distinguished by shape (cube, cylinder, sphere) rather than color. Label objects with text annotations if needed.

---

## Tensor Shape Annotations

When showing data flowing through the pipeline, include tensor shapes at each stage. These help the reader trace dimensionality and catch implementation errors.

| Stage | Shape to show |
|-------|--------------|
| Camera image | [3, 224, 224] |
| Patch embeddings | [196, 512] |
| Language tokens | [8, 512] (varies with instruction length) |
| State embedding | [1, 512] |
| Fused representation | [205, 512] (varies) |
| Motor commands | [7] |
| Action chunk | [K, 7] where K = chunk size |

Show shapes in brackets, Arial 7pt, below or beside the relevant arrow/box.

---

## Grayscale Rules

Most Manning books print in black and white. Every figure must work in grayscale.

### Do
- Use fill patterns (solid gray levels: 0%, 20%, 40%, 60%)
- Use different border styles (solid, dashed, dotted, double)
- Use different border weights (thin, medium, heavy)
- Use shape differences (rounded vs sharp corners, circles vs rectangles)
- Use text labels and annotations for all critical distinctions

### Do not
- Never reference color in captions: "the red box shows..." will not work in print
- Never use color as the only differentiator between two elements
- Never use saturated colors that collapse to the same gray level

### Testing
Before finalizing any figure, convert to grayscale and verify:
- All components are still distinguishable
- All labels are still readable
- The flow direction is still clear
- Annotations are still legible

---

## File Naming and Format

### Naming convention
```
CH01_F01_RTDemoEmergentReasoning.svg
CH01_F02_ClassicalVsLRM.svg
CH01_F03_ThreeModalities.svg
CH01_F04_ActionTokenization.svg
CH01_F05_VLAForwardPass.svg
CH03_F01_VisionEncoderArchitecture.svg
```

Pattern: `CH{nn}_F{nn}_{DescriptiveName}`
- Underscores, not spaces
- Descriptive name in CamelCase
- No chapter references in the name (the CH prefix is sufficient)

### Required file formats
For each figure, provide:
1. **SVG** (or EPS/PDF) with editable text - for Manning production
2. **PNG at 300 dpi** - for reference and MEAP embedding

### Size limits
- Maximum width: 5.6 inches (403 pixels at 72 dpi)
- Maximum height: 7 inches (504 pixels at 72 dpi)
- Must fit on one page

---

## Caption Style

Every caption must:
1. Be at least 3 sentences (Manning benchmark from reference books: 4-5)
2. Describe what is HAPPENING, not just label the figure
3. Walk the reader through the key elements using #A, #B references
4. Never reference color
5. Be readable standalone (someone flipping through should learn from captions alone)

### Caption template
> Figure X.Y [One sentence describing the overall scene/purpose]. [One sentence describing the input side]. [One sentence describing the processing/transformation]. [One sentence describing the output and what it means]. [Optional: one sentence connecting to the broader concept being taught].

### Examples
Bad: "Figure 3.1 The vision encoder architecture."
Good: "Figure 3.1 The vision encoder converts a 224x224 camera image into 196 patch embeddings. The image is divided into a 14x14 grid of non-overlapping patches (#A), each projected into a 512-dimensional embedding (#B). Position encodings are added to preserve spatial layout (#C). The resulting sequence of visual tokens serves as input to the fusion transformer, carrying both what objects are present and where they are in the scene."

---

## Gemini/AI Figure Generation Base Prompt

When using AI tools to generate figures, start every prompt with this preamble:

```
You are creating a figure for the Manning book "Build a Large Robotics 
Model (From Scratch)." Follow these design rules exactly:

ROBOT CHARACTER: A simple geometric line drawing of an SO-100 robot arm 
(6-DOF tabletop arm with gripper). 3/4 front view. No face or personality 
features. Same drawing every time.

COMPONENT BOXES:
- Inputs: rounded rectangles, white/very light gray fill, thin border
- Encoders: rectangles, light gray fill, medium border
- Fusion: large rectangle, medium gray fill, heavy border
- Action heads: rectangles, white fill, dashed border
- Outputs: rectangles, light gray fill, double border

ARROWS: solid = data flow, dashed = alternative path, dotted = optional

ANNOTATIONS: Use #A, #B, #C letter labels in top-left of boxes. Arial 7pt 
for content, 8pt for titles. Include tensor shapes in brackets [3, 224, 224].

LAYOUT: Top to bottom for pipelines. Left to right for comparisons.

GRAYSCALE: Must work in black and white. Use fill levels (0%, 20%, 40%, 60%) 
and border styles (solid, dashed, dotted, double) for distinction. Never 
rely on color.

SIZE: Max 5.6" wide, 7" tall. Font minimum 7pt.

TABLETOP SCENE (when shown): Flat surface, 2-4 simple objects, SO-100 arm 
on left, clean background.
```

Then add your figure-specific prompt after this preamble.

---

## Progression Across Chapters

The visual design system enables a "zooming in" pattern across chapters:

| Chapter | Figure focus | Level of detail |
|---------|-------------|-----------------|
| Ch 1 | Full pipeline overview | All components shown as boxes |
| Ch 2 | Simulation environment | LeRobot/Gymnasium + SO-100 arm + tabletop |
| Ch 3 | Vision encoder + language backbone + fusion | Internal architecture of each component |
| Ch 4 | Discrete action head | Internal tokenization mechanism |
| Ch 5 | Flow matching head | Internal denoising mechanism |
| Ch 6 | Training curriculum | Three-stage progression with LoRA |
| Ch 7 | RL training loop | Policy gradient flow |
| Ch 8 | Reasoning architecture | System 1/System 2 |
| Ch 9 | Sim-to-real pipeline | Side-by-side sim and real |
| Ch 10 | Deployment pipeline | Quantization, TensorRT, Jetson |
| Ch 11 | Future directions | Extended architectures |

In Ch 1, the vision encoder is a single box labeled "Vision Encoder (SigLIP)." In Ch 3, that same box is opened up to reveal patch embedding, position encoding, self-attention layers. The reader recognizes the component from Ch 1 and now sees inside it. This is the visual equivalent of "zooming in."

---

## Checklist Before Submitting Any Figure

- [ ] Uses the correct component box shapes from this guide
- [ ] Uses locked component names (no variations)
- [ ] Annotations use #A, #B, #C format (not cueball numbers)
- [ ] Annotation labels match corresponding code listing labels
- [ ] Works in grayscale (tested by converting to B&W)
- [ ] Caption is 3+ sentences describing what's happening
- [ ] Caption does not reference color
- [ ] File named correctly: CH{nn}_F{nn}_{Name}.svg
- [ ] Both SVG and PNG versions provided
- [ ] Fits within 5.6" x 7"
- [ ] Font sizes: 7pt content, 8pt titles (minimum)
- [ ] SO-100 arm (if present) uses the standard illustration
- [ ] Tensor shapes shown at relevant stages

---

**Last updated**: 2026-04-15
**Maintainer**: Krishnam
