# Graphic Generation Prompts for Chapter 1

This file contains detailed prompts for the Nano Banana 2 (gemini-3.1-flash-image-preview) model to generate the five key technical graphics for Chapter 1: The Generative Robot.

---

## Graphic 1: The Paradigm Shift (Spaghetti vs. Pipeline)
**Command:** `/diagram --type=architecture --complexity=comprehensive --style=professional`
**Prompt:** 
> A side-by-side technical comparison of robot control architectures. 
> **Left Side (The Classical Modular Stack):** A complex, "spaghetti" web of interconnected boxes labeled 'Lidar/Vision', 'SLAM (Mapping)', 'Global Path Planner', 'Local Motion Planner', 'Inverse Kinematics', and 'PID Motor Controller'. Use thin, jagged arrows to show the fragmented flow of data.
> **Right Side (The Modern VLA Generative Stack):** A clean, linear pipeline. Two input icons (a camera image and a text bubble with "Pick up the cup") enter a single, large, glowing blue block labeled 'Unified VLA Transformer'. A single thick arrow exits the block to a robot arm icon labeled 'Direct Motor Commands'. 
> Use a professional color palette (dark blues, greys, and accent highlights).

---

## Graphic 2: Motion as a Modality (Action Tokenization)
**Command:** `/diagram --type=flowchart --complexity=detailed --style=technical`
**Prompt:**
> A technical visualization of 'Action Tokenization'. 
> On the left, a realistic 3D model of a 7-DOF robot arm with joint axes highlighted. 
> In the center, a set of 'Translator' arrows pointing toward a digital interface. 
> On the right, a sequence of stylized code/text tokens in a horizontal row. Each token is a box with a value: [TOKEN_422: BASE_ROTATION], [TOKEN_12: SHOULDER_LIFT], [TOKEN_88: ELBOW_FLEX], [TOKEN_5: GRIPPER_CLOSE]. 
> The visual should illustrate the conversion of smooth physical motion into a discrete digital vocabulary. Minimalist design, high contrast.

---

## Graphic 3: The Multimodal Injection (Architecture)
**Command:** `/diagram --type=architecture --complexity=detailed --style=professional`
**Prompt:**
> A detailed internal architectural diagram of 'Multimodal Injection' for a VLA. 
> Show a horizontal stream of tokens entering a 'Transformer Decoder Block'. 
> Represent the tokens using two distinct visual styles: 
> 1. Blue boxes containing text snippets (e.g., "The", "robot", "should").
> 2. Small square grid patches extracted from a photograph of a kitchen counter. 
> Show these patches and words interleaved in a single row. 
> Above the row, add labels: 'Semantic Word Embeddings' and 'Spatial Vision Embeddings'. 
> Use arrows to show them all being processed by the same 'Attention Mechanism' layers.

---

## Graphic 4: Positive Transfer (Emergent Reasoning)
**Command:** `/generate_image --styles photorealistic, modern --variations lighting, angle`
**Prompt:**
> A professional POV (Point of View) shot of a robot's workspace. 
> On a white laboratory table, there is a toy plastic T-Rex (dinosaur) and a small blue toy sedan (car). 
> At the bottom of the frame, a UI overlay shows the text prompt: "Action: Pick up the extinct animal." 
> A semi-transparent yellow 'Attention Heatmap' glows specifically over the toy T-Rex, while the toy car remains un-highlighted. 
> The lighting should be clean and clinical, representing a high-tech AI lab environment.

---

## Graphic 5: The Bitter Lesson (Scaling Curve)
**Command:** `/diagram --type=flowchart --complexity=simple --style=professional`
**Prompt:**
> A clean technical line graph titled 'The Bitter Lesson for Robotics'. 
> **X-Axis:** 'Data & Compute Scale (Log Scale)'. 
> **Y-Axis:** 'Task Success Rate (%)'. 
> **Line 1 (Red):** Labeled 'Human-designed Heuristics'. It starts high at low data levels but quickly plateaus and stays flat as data increases. 
> **Line 2 (Green):** Labeled 'Generative VLA Models'. It starts very low (near zero) at the beginning but grows exponentially, crossing over the red line and continuing upward toward 100%. 
> Add a vertical dotted line where they cross labeled 'The Generative Crossover'.
