# Graphic Generation Prompts for Chapter 2

This file contains prompts for generating the eight key technical graphics for Chapter 2: Simulation & Data. Figures fall into three categories:

- **Reused** (Figure 2.1) — the book-wide roadmap from Chapter 1 with the Chapter 2 stage highlighted.
- **Authored diagrams** (Figures 2.2, 2.3, 2.7, 2.8) — architectural / flow diagrams generated from descriptions.
- **Data-rendered** (Figures 2.4, 2.5, 2.6) — produced by running the chapter notebook (`notebooks/ch02.ipynb`) against the LeRobot dataset and saving the matplotlib output.

The four authored diagrams should share a consistent aesthetic: monochrome line work with **exactly one accent color per diagram**, a white background, sans-serif labels (Inter or Manning's house font when available), and 300 DPI for print. Captions are in the chapter manuscript, not in the figure.

---

## Figure 2.1: Where this chapter sits in the book
**Category:** Reuse.
**Source:** `chapter_1/figures/diagrams/figure_1_7_book_roadmap.png` (or whatever the latest Chapter 1 export is).
**Modification:** Highlight the "Foundations (Ch 1-2)" stage with the chapter-2 box outlined in the accent color and a small "you are here" indicator.
**File output:** `figures/diagrams/figure_2_1_roadmap.png`

---

## Figure 2.2: The Gymnasium interaction loop
**Command:** `/diagram --type=flowchart --complexity=simple --style=technical`
**Accent color:** Blue (#1f77b4).
**Prompt:**
> A clean cyclic flow diagram of the Gymnasium reset/step loop, suitable for an engineering documentation book.
> Five labeled boxes arranged in a circular flow:
> 1. **reset()** (top center) — small label "initial observation".
> 2. **Agent** (right) — labeled "selects action".
> 3. **env.step(action)** (bottom right) — labeled with the return tuple "(obs, reward, terminated, truncated, info)".
> 4. **Next observation** (bottom left) — feeds back into Agent.
> 5. **Done?** decision diamond at the left, with a "no" arrow looping back to Agent, and a "yes" arrow exiting to a small terminator box labeled "episode end".
> Use simple right-angle arrows with arrowheads. White background. Black box outlines. Use the accent color only for the loop arrow returning from "next observation" to "agent". Sans-serif labels.

**File output:** `figures/diagrams/figure_2_2_gymnasium_loop.png`

---

## Figure 2.3: The PickCubeSO100 task
**Category:** Authored screenshot (rendered from the simulator).
**Source:** Render at simulation startup with the env in its initial pose. Use ManiSkill3's `env.render()` with `render_mode="rgb_array"`, then upscale to 6 inches at 300 DPI for print.
**Annotations to overlay in matplotlib:**
- An arrow + label pointing at the gripper: "End-effector (gripper)"
- An arrow + label pointing at the cube: "Cube (object)"
- A dashed outline around the target zone with label: "Target zone"
- A small inset at the top-right corner showing the wrist-camera view
- Title bar reading "PickCubeSO100-v1 (ManiSkill3)"

**File output:** `figures/diagrams/figure_2_3_pickcube_task.png`

**Generation snippet (lives in `notebooks/ch02.ipynb` §2.1):**

```python
import matplotlib.pyplot as plt
obs, _ = env.reset(seed=42)
top = env.render()
wrist = obs["sensor_data"]["hand_camera"]["rgb"]
fig, ax = plt.subplots(figsize=(8, 6))
ax.imshow(top)
ax.set_title("PickCubeSO100-v1 (ManiSkill3)")
# ... annotation overlays via ax.annotate ...
fig.savefig("../figures/diagrams/figure_2_3_pickcube_task.png", dpi=300, bbox_inches="tight")
```

---

## Figure 2.4: Expert pick-and-place keyframes
**Category:** Data-rendered. Produced by `ch02.viz.render_keyframes(dataset, episode_idx=0, n_frames=6)`.
**Spec:**
- 2 rows × 6 columns of subplots.
- Top row: top-down third-person camera frames.
- Bottom row: wrist-mounted camera frames.
- Columns are evenly spaced timesteps across one expert episode.
- Column titles: "step N" where N is the frame index.
- Row labels (y-label on first column): "Top view" / "Wrist view".
- All subplots `axis("off")` except for the row labels.
- Saved as `figures/diagrams/figure_2_4_expert_keyframes.png` at 300 DPI.

The implementation lives in `src/ch02/viz.py`. No model-generated graphic needed; the notebook produces this directly.

---

## Figure 2.5: Per-joint action distributions (expert vs. scripted vs. random)
**Category:** Data-rendered. Produced by `ch02.viz.plot_action_distributions(expert, scripted, random_)`.
**Spec:**
- 2 rows × 4 columns of subplots (7 used, last cell empty or hidden).
- One subplot per action dimension: joint_0 through joint_5, plus gripper.
- Three overlaid density histograms per subplot, with `alpha=0.4`:
  - Expert (blue, `#1f77b4`)
  - Scripted (orange, `#ff7f0e`)
  - Random (gray, `#7f7f7f`)
- Subplot title: the joint name.
- Shared x-axis range: `[-1, 1]` (the action range).
- Legend in the first subplot, font size 8.
- Saved as `figures/diagrams/figure_2_5_action_distributions.png` at 300 DPI.

Implementation in `src/ch02/viz.py`.

---

## Figure 2.6: Expert joint trajectories
**Category:** Data-rendered. Produced by `ch02.viz.plot_joint_trajectories(dataset, episode_indices=range(5))`.
**Spec:**
- 2 rows × 3 columns of subplots (6 cells for 6 joints).
- One subplot per arm joint (joint_0 through joint_5).
- Within each subplot, plot joint angle vs. frame index for 5 different episodes, each line semi-transparent (`alpha=0.6`).
- Subplot title: the joint name.
- y-axis: "angle (rad)". x-axis: "frame".
- Saved as `figures/diagrams/figure_2_6_joint_trajectories.png` at 300 DPI.

Implementation in `src/ch02/viz.py`.

---

## Figure 2.7: The normalization round-trip
**Command:** `/diagram --type=flowchart --complexity=detailed --style=technical`
**Accent color:** Green (#2ca02c) for the forward (training) path; orange (#ff7f0e) for the reverse (inference) path.
**Prompt:**
> A horizontal flow diagram showing the bidirectional normalization pipeline used in a robot policy training and inference loop.
>
> **Forward (training) path, top:**
> 1. A box labeled "Raw observation / action" (showing example value "joint_pos ≈ 1.2 rad").
> 2. Arrow into a box labeled "normalize(x, stats, key)".
> 3. Arrow into a box labeled "Zero-centered input" (showing example value "~0.03").
> 4. Arrow into a stylized "Model" box.
>
> **Reverse (inference) path, bottom (right-to-left):**
> 1. The Model emits a "Normalized action prediction" box (example value "~-0.05").
> 2. Arrow leftward into "denormalize(ŷ, stats, key)".
> 3. Arrow into "Environment-scale action" (example "joint_delta ≈ 0.08 rad").
> 4. Arrow into "env.step()".
>
> Both paths share a central "stats dict" box (containing mean, std, min, max per feature) connected to both normalize and denormalize via dashed lines, indicating it is referenced read-only by both.
>
> Use solid arrows in the accent colors. Sans-serif labels. White background.

**File output:** `figures/diagrams/figure_2_7_normalization_roundtrip.png`

---

## Figure 2.8: End-to-end data pipeline
**Command:** `/diagram --type=architecture --complexity=detailed --style=technical`
**Accent color:** Blue (#1f77b4).
**Prompt:**
> A horizontal data-flow architecture diagram showing the complete pipeline built across sections 2.1–2.5.
>
> Five labeled boxes left to right, connected by thick right-pointing arrows:
> 1. **Hugging Face Hub** — small Hugging Face logo, label "lerobot/svla_so101_pickplace".
> 2. **LeRobotDataset** — labeled "parquet shards + image archives".
> 3. **compute_stats()** — short label "one pass over the dataset".
> 4. **DataLoader** — labeled "batch + shuffle + normalize-in-collate". Show a small substructure inside this box: a sub-flow with three branches labeled "state → z-score", "action → z-score", "images → x / 255".
> 5. **Chapter 3 model** — placeholder block with a question mark inside, labeled "to be built".
>
> Below the DataLoader, a small callout box pointing up to it: `make_pickplace_dataloader(dataset_id, batch_size, shuffle)`.
>
> Use the accent color for the arrows. Sans-serif labels. White background. Tight composition, suitable for a 6-inch print width.

**File output:** `figures/diagrams/figure_2_8_data_pipeline.png`

---

## Production checklist (before submitting to Manning)

For every figure:
- [ ] Saved as PNG at 300 DPI, white background.
- [ ] Sans-serif labels, no exotic embedded fonts.
- [ ] Single accent color for diagrams; multi-color only where the data demands (figure 2.5).
- [ ] Captions in `chapter_2/manuscript/chapter_2.md`, not in the figure file.
- [ ] No marketing words or emojis in labels.
- [ ] Visually inspected at print size (the eye catches label collisions code paths don't).
- [ ] Data-rendered figures (2.4, 2.5, 2.6) regenerable from a fixed-seed notebook cell.
- [ ] File names match the manuscript references: `figure_2_<n>_<slug>.png`.

For full requirements (color palette beyond the accent, axis-label conventions, caption format), defer to `../../FIGURE_STYLE_GUIDE.md`.
