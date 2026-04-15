# Chapter 1 Figure Prompts

These prompts generate figures for Chapter 1: "The Generative Robot."
Each prompt should be prefixed with the base prompt from FIGURE_STYLE_GUIDE.md.

Status: Draft prompts. Regenerate when chapter text is finalized.

---

## Figure 1.1: The RT-2 Emergent Reasoning Demonstration

**Purpose**: Hook figure. Shows the dinosaur grasping scene that opens the chapter.
**Location**: After the caution paragraph in Section 1.1
**Type**: Scene illustration (not architectural)

```
Create a clean, minimal illustration of a robotic manipulation scene
for a Manning technical book.

SCENE:
- A tabletop with two objects: a toy dinosaur (left) and a toy truck (right)
- An SO-100 robot arm reaching toward the toy dinosaur
- The arm's gripper is approaching the dinosaur, not the truck

INSTRUCTION OVERLAY:
- A text box at the bottom or top with the instruction:
  "Pick up the extinct animal"

ATTENTION INDICATOR:
- A subtle dashed circle or highlight around the dinosaur showing 
  the model's attention/selection
- Label this highlight: "Model selects dinosaur"
- Do NOT highlight the truck

ANNOTATION:
- Below the scene, a single annotation box:
  "The model learned 'extinct' from internet pre-training, 
   not from robotics demonstrations"

STYLE:
- Clean, minimal, professional
- The objects should be simple iconic representations, not photorealistic
- The SO-100 arm should match the standard line drawing from the 
  figure style guide
- Works in grayscale: use dashed circle for attention, not color
- No background clutter
- Title: "Emergent Reasoning in a Large Robotics Model"

SIZE: 5.6 inches wide, ~4 inches tall
```

**Caption**:
> Figure 1.1 The RT-2 emergent reasoning demonstration. A robot arm is presented with two objects (a toy dinosaur and a toy truck) and the instruction "Pick up the extinct animal." The model identifies the dinosaur as the correct target, despite never encountering the concept of extinction during robotics training. The understanding of "extinct" was inherited from the vision-language backbone's internet pre-training, where the model encountered dinosaurs described as extinct in encyclopedias, museum websites, and children's books.

---

## Figure 1.2: Classical Modular Stack vs. LRM

**Purpose**: Contrast the old and new approaches to robot control.
**Location**: After Section 1.2.3 (the end-to-end alternative)
**Type**: Comparison diagram (left-to-right)

```
Create a side-by-side comparison diagram for a Manning technical book.

LEFT SIDE - "Classical Modular Stack":
- Title at top: "Classical Modular Stack"
- Vertical stack of 6 boxes, connected by dashed red/gray arrows 
  pointing downward:
  1. "Camera / Sensors" (top)
  2. "SLAM & Mapping"
  3. "Global Path Planner"
  4. "Local Motion Planner"
  5. "Inverse Kinematics"
  6. "PID Motor Controller" (bottom)
- Between each box, a small "X" or crack symbol on the arrow 
  indicating a brittle interface
- Bottom label: "Brittle interfaces - Compounding errors"

SEPARATOR:
- Vertical dashed line between the two sides

RIGHT SIDE - "LRM Generative Stack":
- Title at top: "LRM Generative Stack"
- Three input boxes at top, side by side:
  - "Sensor Input" (generic, not just camera)
  - "Task Specification" (generic, not just language)
  - "Body State"
- All three arrows converge into one large box:
  - "Learned Model" (heavy border, medium gray fill)
- One arrow out to:
  - "Motor Commands" (bottom)
- Bottom label: "End-to-end - Learned interfaces"

STYLE:
- Left side uses thin borders, light fill, showing the fragility
- Right side uses heavier borders, darker fill for the learned model
- The height contrast (6 boxes vs 1 box) should be visually striking
- Works in grayscale
- No color-dependent teaching
- Title: "Classical Modular Stack vs. Generative LRM Stack"

NOTE ON TERMINOLOGY:
- Right side should say "Sensor Input" not "Camera Image" 
  (LRM is generic, not VLA-specific yet)
- Right side should say "Task Specification" not "Text Instruction"
  (LRM can take any task format, not just language)
- This figure shows the LRM paradigm generally, before narrowing to VLA

SIZE: 5.6 inches wide, ~5 inches tall
```

**Caption**:
> Figure 1.2 The classical modular stack (left) requires hand-engineering every interface between components. A single error in perception cascades through every downstream module. A Large Robotics Model (right) replaces the entire stack with a single learned model that maps sensor inputs, task specifications, and body state directly to motor commands. The height difference between the two stacks reflects the reduction in hand-designed interfaces: six explicit handoffs on the left become a single learned forward pass on the right.

---

## Figure 1.3: The Three Modalities of a VLA

**Purpose**: Show how VLAs specifically narrow the LRM inputs to vision + language + proprioception.
**Location**: After Section 1.3.2 (proprioception)
**Type**: Architecture diagram (top-to-bottom, simple)

```
Create a simple three-input, one-output diagram for a Manning 
technical book.

This figure shows how a VLA (a specific type of LRM) takes three 
specific input modalities and produces motor commands.

TOP ROW - Three input boxes side by side:

#A - Vision
- Rounded rectangle, light gray fill
- Title: "Vision"
- Subtitle: "What the robot sees"
- Small camera icon or image grid thumbnail

#B - Language  
- Rounded rectangle, light gray fill
- Title: "Language"
- Subtitle: "What it is told to do"
- Small text lines icon

#C - Proprioception
- Rounded rectangle, light gray fill
- Title: "Proprioception"
- Subtitle: "Where its body is"
- Small robot arm joint icon

MIDDLE - One large box receiving all three inputs:
- Heavy border, medium gray fill
- Title: "VLA Model"
- Three arrows converging from #A, #B, #C into this box

BOTTOM - Output box:
- Light gray fill, double border
- Title: "Motor Commands"
- Subtitle: "50Hz control"
- Arrow from VLA Model to Motor Commands

BOTTOM - Small SO-100 arm icon executing a task

STYLE:
- Simple, clean, symmetric
- The three input boxes should be the same size and aligned
- The VLA Model box should be visually larger/heavier
- Works in grayscale
- Title: "The Three Modalities of a Vision-Language-Action Model"
- Show that this is a specific case of the generic LRM from Figure 1.2
  (sensor input narrowed to vision, task specification narrowed to 
  language, body state narrowed to proprioception)

SIZE: 5.6 inches wide, ~4.5 inches tall
```

**Caption**:
> Figure 1.3 The three modalities of a Vision-Language-Action model. Vision provides scene context (camera images), language provides the goal (natural-language instructions), and proprioception provides body state (joint angles and velocities). The model fuses all three into a single representation and produces motor commands as output. This three-way fusion is what distinguishes a VLA from a standard vision-language model, which can describe a scene but cannot physically interact with it. A VLA is a specific type of LRM (figure 1.2) that narrows the generic sensor and task inputs to camera, language, and joint state.

---

## Figure 1.4: Action Tokenization

**Purpose**: Show how continuous joint commands become discrete tokens.
**Location**: After Section 1.4.1 (actions as tokens)
**Type**: Process diagram (left-to-right transformation)

```
Create a diagram showing how continuous robot joint angles are 
tokenized into discrete tokens for a Manning technical book.

LEFT SIDE - Continuous:
- Title: "Continuous Joint Angle"
- A horizontal number line from 0° to 180°
- A marker at 45.3° showing the actual desired position
- Label: "Desired: 45.3°"

MIDDLE - Binning:
- The number line is divided into visible bins (show maybe 5-6 bins 
  around the 45° region)
- Each bin is labeled with its index: "...44, 45, 46, 47..."
- The bin containing 45.3° is highlighted
- Arrow pointing to the bin: "Rounded to bin 45"
- Label: "256 bins per joint"

RIGHT SIDE - Token:
- A single token box with "45" inside
- Below it, a sequence of 7 token boxes representing a full action:
  [45] [128] [67] [12] [90] [180] [1]
- Label below: "7 tokens = one motor command"
- Subtitle: "Same format as word tokens in an LLM"

BOTTOM - Connection to language:
- Show a parallel: 
  - Language: "pick" "up" "the" "cup" → [8821] [492] [1] [3847]
  - Action:  "base" "shoulder" "elbow" ... → [45] [128] [67] ...
- Label: "The Transformer predicts both the same way"

STYLE:
- Clean, horizontal flow left to right
- The binning visualization should be clear and educational
- The parallel between language tokens and action tokens should be 
  visually obvious (same box style for both)
- Works in grayscale
- Title: "Action Tokenization: From Continuous Angles to Discrete Tokens"

SIZE: 5.6 inches wide, ~4 inches tall
```

**Caption**:
> Figure 1.4 Action tokenization converts continuous joint commands into discrete tokens. Each joint's range is divided into 256 bins, and the continuous angle is rounded to the nearest bin, producing an integer token. A complete motor command for a 7-DOF arm becomes a sequence of 7 tokens, predicted by the Transformer using the same mechanism it uses to predict words. The bottom row shows the parallel: language tokens and action tokens are structurally identical from the Transformer's perspective. This is the approach used by RT-2 and the discrete action head you will build.

---

## Figure 1.5: The VLA Forward Pass (Mental Model)

**Purpose**: The Mental Model figure. Traces one specific task through the entire pipeline with tensor shapes.
**Location**: Section 1.6.1, before the code listing
**Type**: Pipeline diagram (top-to-bottom, detailed)

```
[Use the revised Gemini prompt from the earlier session - the one 
with all 16 fixes applied. The figure shows "push the green cube left" 
flowing through:
#A Camera Image [3, 224, 224]
#B Language Instruction "push the green cube left"  
#C Joint Positions [45°, 90°, 30°, 15°, 60°, 0°, open]
#D Vision Encoder (SigLIP) → [196, 512]
#E Language Backbone (SmolLM) → [8, 512]
#F State Encoder → [1, 512]
#G Fusion Transformer (6 layers, 512-dim)
#H Discrete Action Head (256 bins per joint)
#I Flow Matching Head (continuous trajectories)
#J Motor Commands [0.12, -0.05, 0.30, 0.0, 0.0, 0.78, 0.5]

With section headers: INPUTS, ENCODERS, FUSION, ACTION DECODING, OUTPUT
Dashed/dotted lines for the two alternative action paths
SO-100 arm at bottom executing the push]
```

**Caption**:
> Figure 1.5 The VLA forward pass for the task "push the green cube left." A camera image (#A) is encoded by the vision encoder into 196 patch embeddings (#D). The language instruction (#B) is encoded by the language backbone into 8 token embeddings (#E). The robot's current joint positions (#C) are projected into the same 512-dimensional embedding space by the state encoder (#F). The fusion transformer (#G) combines all three streams. The fused representation passes to either a discrete action head (#H) that predicts binned joint positions or a flow matching head (#I) that generates continuous trajectories. Both paths produce a 7-dimensional motor command (#J) sent to the robot arm at 50Hz.

---

## Figure Summary

| Figure | Type | Section | Key teaching point |
|--------|------|---------|-------------------|
| 1.1 | Scene illustration | 1.1 | Emergent reasoning (the hook) |
| 1.2 | Comparison | 1.2.3 | Classical stack vs LRM (generic) |
| 1.3 | Architecture | 1.3.2 | Three modalities of VLA (specific) |
| 1.4 | Process | 1.4.1 | Action tokenization (core insight) |
| 1.5 | Pipeline (Mental Model) | 1.6.1 | Full VLA forward pass with tensor shapes |

Five figures in ~19 pages = 1 figure per 3.8 pages. Within Manning benchmark (1 per 2-3 pages for Raschka, 1 per 5 for DAS).

## Note on Terminology in Figures

Figures 1.1 and 1.2 use LRM terminology (generic: sensors, task specification, body state). This matches the chapter's progression where LRM is the broad category.

Figures 1.3, 1.4, and 1.5 use VLA terminology (specific: camera, language, proprioception). This matches the narrowing from LRM to VLA that happens in Section 1.1.1.

This visual narrowing mirrors the prose: LRM (broad) → VLA (specific) → the VLA you build (concrete).
