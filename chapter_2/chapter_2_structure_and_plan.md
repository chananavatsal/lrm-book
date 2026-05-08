# Chapter 2: Simulation & Data — Structure & Content Plan

## Archetype

**Primary:** Hands-on Setup (Raschka-style) — environment, data, and pipeline construction.

Code-heavy chapter. The reader installs tools, runs a simulator, writes a scripted policy, loads expert data, and builds a reusable DataLoader. Every concept (observation, action, episode, reward) is taught through PushT directly — no CartPole detour.

---

## Chapter Opening

### "This chapter covers" block (5 bullets)
- Setting up the PushT simulation environment using Gymnasium and LeRobot
- Understanding the Gymnasium interface: observations, actions, episodes, steps, and rewards
- Writing a scripted heuristic policy and observing its failure modes — motivation for learned policies
- Loading, inspecting, and visualizing expert demonstration data from the LeRobot Hub
- Building a normalized DataLoader that serves as the data contract for Chapter 3

### Hook paragraphs (2 paragraphs)
- **Paragraph 1:** A neural network that controls a robot is only as good as the data it trains on. Before writing a single line of model code, you need three things: an environment to act in, demonstrations to learn from, and a pipeline to feed that data into training. This chapter builds all three.
- **Paragraph 2:** You will work with PushT, a 2D pushing task where a circular end-effector must nudge a T-shaped block into a target pose. It looks simple — and that is the point. PushT is complex enough to expose why hand-coded policies struggle, yet fast enough to train on a laptop in minutes. By the end of this chapter, you will have a working data pipeline that Chapter 3 plugs directly into.

---

## Section 2.1: The PushT Environment

**Purpose:** Install the tooling, launch PushT, and teach the Gymnasium API through direct interaction with the task.

**Target length:** ~5 pages

**Content:**

### 2.1.1 Installation and First Launch
- Install LeRobot and its dependencies (gymnasium, pygame, etc.)
- Create a PushT environment instance
- Call `env.reset()` and inspect what comes back: the observation dictionary
- Call `env.step(action)` and inspect the return tuple: `(obs, reward, terminated, truncated, info)`

### 2.1.2 The Gymnasium Interface
- Define the core loop: `reset → step → step → ... → done`
- **Observation space:** What the agent sees — for PushT this is the agent position (x, y) and the T-block pose (x, y, θ), or optionally a rendered image
- **Action space:** What the agent can do — for PushT this is a 2D continuous action (Δx, Δy)
- **Episode:** One complete attempt from reset to termination
- **Step:** A single (observation, action, reward, next_observation) transition
- **Reward:** PushT's reward is the overlap between the T-block's current pose and the target pose (coverage metric)

### 2.1.3 Running a Random Agent
- Write a loop that samples random actions from the action space
- Collect total reward over an episode
- Run multiple episodes and report mean reward
- The random agent performs poorly — establishes the baseline

**Listing 2.1: Installing LeRobot and creating the PushT environment**
```python
# pip install lerobot gymnasium pygame
import gymnasium as gym                          #A
import lerobot                                   #B

env = gym.make("gym_pusht/PushT-v0",             #C
               render_mode="rgb_array")           #C
obs, info = env.reset(seed=42)                    #D
print(f"Observation keys: {obs.keys()}")          #E
print(f"Action space: {env.action_space}")        #F
```
- #A Core simulation library
- #B Registers PushT and other LeRobot environments
- #C Create the PushT environment with image rendering
- #D Reset returns initial observation and info dict
- #E Observation is a dictionary with agent and block state
- #F Action space is Box(2,) — continuous 2D displacement

**Listing 2.2: Running a random agent in PushT**
```python
import numpy as np

def run_random_agent(env, n_episodes=10):
    rewards = []
    for ep in range(n_episodes):
        obs, info = env.reset(seed=ep)
        total_reward = 0.0
        done = False
        while not done:
            action = env.action_space.sample()           #A
            obs, reward, terminated, truncated, done_info = env.step(action)
            total_reward += reward
            done = terminated or truncated
        rewards.append(total_reward)
    return np.mean(rewards), np.std(rewards)

mean_r, std_r = run_random_agent(env)
print(f"Random agent: {mean_r:.1f} ± {std_r:.1f}")      #B
```
- #A Sample uniformly from the 2D continuous action space
- #B Expect low reward — random pushes rarely align the T-block

**Callout Box: "WHAT IS GYMNASIUM?"**
- Gymnasium (formerly OpenAI Gym) is the standard Python API for reinforcement learning environments
- Every environment exposes `reset()` and `step(action)` — a universal interface regardless of the task
- LeRobot registers its environments as Gymnasium environments, so the same code patterns work for PushT, ALOHA, and real robot deployments

**Figure 2.1: The PushT Task**
- Annotated screenshot of the PushT environment
- Labels: end-effector (circle), T-block (current pose), target pose (ghosted outline), coverage overlap region
- Caption: "The PushT task. A circular end-effector (blue) must push a T-shaped block (dark) to match a fixed target pose (outline). The reward measures the overlap between the block's current pose and the target."

**Figure 2.2: The Gymnasium Loop**
- Flow diagram: reset() → observation → agent selects action → step(action) → (obs, reward, terminated, truncated, info) → loop back or done
- Caption: "The Gymnasium interaction loop. The agent receives an observation, selects an action, and the environment returns the next observation, a scalar reward, and termination flags. This loop is the universal interface for all environments in this book."

**Table 2.1: PushT Observation and Action Spaces**

| Feature | Shape | Type | Description |
|---------|-------|------|-------------|
| Agent position | (2,) | float32 | End-effector (x, y) coordinates |
| Block pose | (3,) | float32 | T-block (x, y, θ) pose |
| Image (optional) | (96, 96, 3) | uint8 | Top-down RGB rendering |
| Action | (2,) | float32 | End-effector displacement (Δx, Δy) |

**Transition:** "The random agent flails. Can you do better with a simple rule?"

---

## Section 2.2: A Scripted Policy

**Purpose:** Give the reader agency by writing a heuristic policy. Show that even a reasonable hand-coded approach has clear failure modes — motivating learned policies.

**Target length:** ~4 pages

**Content:**

### 2.2.1 Designing the Heuristic
- Strategy: compute the vector from the end-effector to the T-block center, move toward it, then push toward the target pose
- Two-phase approach: (1) approach the block, (2) push it toward the goal
- Simple proportional control: action = clipped direction vector scaled by a gain

### 2.2.2 Implementation
- Write the scripted policy function
- Run it for multiple episodes
- Compare mean reward against the random baseline — significant improvement

### 2.2.3 Where the Heuristic Fails
- The scripted policy cannot handle rotations well — pushing a T-block to match orientation requires multi-step reasoning about contact geometry
- It gets stuck in local optima (e.g., pushing the block past the target, then unable to recover)
- It has no memory — each step is a memoryless reaction to current state
- Key insight: even a "smart" heuristic is far from expert performance. Learned policies can capture the subtle contact dynamics that rules miss.

**Listing 2.3: A scripted push-toward-target policy**
```python
def scripted_policy(obs, gain=1.0):
    """Move toward the T-block, then push it toward the target."""
    agent_pos = obs["agent_pos"]                       #A
    block_pos = obs["block_pos"][:2]                   #B
    target_pos = obs["target_pos"][:2]                 #C

    to_block = block_pos - agent_pos                   #D
    dist_to_block = np.linalg.norm(to_block)

    if dist_to_block > 10.0:                           #E
        direction = to_block / (dist_to_block + 1e-8)
    else:
        to_target = target_pos - block_pos             #F
        direction = to_target / (np.linalg.norm(to_target) + 1e-8)

    action = np.clip(direction * gain, -1.0, 1.0)     #G
    return action
```
- #A Current end-effector (x, y)
- #B T-block center position
- #C Target position for the block
- #D Vector pointing from agent to block
- #E Phase 1: if far from block, approach it
- #F Phase 2: if close to block, push toward target
- #G Clip to valid action range

**Listing 2.4: Evaluating the scripted policy**
```python
def run_scripted_agent(env, n_episodes=10):
    rewards = []
    for ep in range(n_episodes):
        obs, info = env.reset(seed=ep)
        total_reward = 0.0
        done = False
        while not done:
            action = scripted_policy(obs)
            obs, reward, terminated, truncated, _ = env.step(action)
            total_reward += reward
            done = terminated or truncated
        rewards.append(total_reward)
    return np.mean(rewards), np.std(rewards)

mean_r, std_r = run_scripted_agent(env)
print(f"Scripted agent: {mean_r:.1f} ± {std_r:.1f}")   #A
```
- #A Better than random, but far from expert coverage

**Callout Box: "WHY NOT JUST ENGINEER A BETTER HEURISTIC?"**
- You could add rotation handling, multi-step planning, and recovery behaviors
- But every improvement requires more hand-coded rules — and each new edge case compounds the complexity
- This is the "long tail" problem from Chapter 1 in miniature: heuristics plateau, learned policies scale with data

**Transition:** "The scripted policy shows what one person's intuition can achieve. Expert demonstrations show what practiced skill looks like as data."

---

## Section 2.3: The LeRobot Dataset Standard

**Purpose:** Load expert demonstrations from the Hub, understand the data format, and see what successful task completion looks like numerically.

**Target length:** ~5 pages

**Content:**

### 2.3.1 Loading from the Hub
- Use `lerobot.common.datasets.lerobot_dataset.LeRobotDataset` to load PushT expert data
- The dataset is hosted on Hugging Face Hub — single line to download
- Inspect dataset length, number of episodes, and feature names

### 2.3.2 The Feature Schema
- Each sample is a dictionary with keys: `observation.state`, `action`, `episode_index`, `frame_index`, `timestamp`, etc.
- Observation contains the agent and block state vectors
- Actions are the expert's recorded (Δx, Δy) commands
- `episode_index` groups frames into episodes; `frame_index` orders them within each episode

### 2.3.3 Understanding delta_timestamps
- LeRobot's `delta_timestamps` mechanism: request observation/action at relative time offsets from the current frame
- Example: `delta_timestamps = {"observation.state": [-0.1, 0.0], "action": [0.0, 0.1, 0.2]}` returns the previous and current observations, plus the current and two future actions
- This is the mechanism for action chunking (predicting multiple future actions) — preview for Chapter 4

### 2.3.4 Episode Structure
- Iterate through one episode and print shapes at each step
- Show how episodes begin (reset state) and end (success or timeout)
- Count steps per episode — episodes have variable length

**Listing 2.5: Loading the PushT expert dataset**
```python
from lerobot.common.datasets.lerobot_dataset import LeRobotDataset

dataset = LeRobotDataset(
    "lerobot/pusht",                                    #A
)
print(f"Total frames: {len(dataset)}")                  #B
print(f"Episodes: {dataset.num_episodes}")
print(f"Features: {list(dataset.features.keys())}")     #C
```
- #A Dataset identifier on Hugging Face Hub
- #B Total number of (observation, action) frames across all episodes
- #C Feature names: observation.state, action, episode_index, etc.

**Listing 2.6: Inspecting a single frame and episode**
```python
frame = dataset[0]                                      #A
for key, val in frame.items():
    if hasattr(val, 'shape'):
        print(f"  {key}: shape={val.shape}, dtype={val.dtype}")
    else:
        print(f"  {key}: {val}")

episode_0 = [dataset[i] for i in range(len(dataset))    #B
             if dataset[i]["episode_index"] == 0]
print(f"\nEpisode 0 length: {len(episode_0)} steps")
```
- #A Access a single frame by integer index
- #B Collect all frames belonging to episode 0

**Table 2.2: LeRobot PushT Dataset Feature Summary**

| Feature | Shape | Type | Description |
|---------|-------|------|-------------|
| observation.state | (5,) | float32 | Agent (x,y) + block (x,y,θ) |
| observation.image | (96, 96, 3) | uint8 | Top-down RGB rendering |
| action | (2,) | float32 | Expert (Δx, Δy) command |
| episode_index | scalar | int64 | Which episode this frame belongs to |
| frame_index | scalar | int64 | Position within the episode |
| timestamp | scalar | float32 | Time in seconds from episode start |

**Callout Box: "WHAT IS delta_timestamps?"**
- LeRobot's mechanism for requesting data at relative time offsets from a given frame
- Setting `delta_timestamps={"action": [0.0, 0.1, 0.2]}` returns the current action plus two future actions, stacked into shape (3, 2)
- This enables *action chunking*: predicting a short sequence of future actions instead of one at a time
- Action chunking improves policy smoothness and is used in Chapters 4 and 5

**Transition:** "Numbers in a table tell you the data's shape. Plots tell you its character."

---

## Section 2.4: Visualizing the Data

**Purpose:** Build visual intuition about expert behavior. Compare expert, scripted, and random action distributions to make the quality gap concrete.

**Target length:** ~4 pages

**Content:**

### 2.4.1 Rendering Expert Episodes
- Extract keyframes from an expert episode at regular intervals
- Display as a filmstrip — a row of images showing the progression of a successful push
- Annotate with reward at each keyframe

### 2.4.2 Action Distributions: Three-Way Comparison
- Collect actions from: (1) the expert dataset, (2) the scripted policy running for N episodes, (3) random sampling
- Plot 2D scatter/density plots of (Δx, Δy) for each
- Expert actions show structured, multi-modal clusters (approach from different angles depending on block orientation)
- Scripted actions show a simpler, more concentrated pattern
- Random actions fill the action space uniformly
- The gap between scripted and expert distributions is the gap that learning must close

### 2.4.3 State Trajectories
- Plot expert end-effector trajectories in (x, y) space across several episodes
- Overlay the T-block start and target positions
- Show that experts take different paths depending on initial conditions — the policy must be *conditional*, not a single memorized trajectory

**Listing 2.7: Rendering expert episode keyframes**
```python
import matplotlib.pyplot as plt

def render_keyframes(dataset, episode_idx=0, n_frames=6):
    """Extract and display keyframes from one expert episode."""
    ep_frames = [dataset[i] for i in range(len(dataset))
                 if dataset[i]["episode_index"] == episode_idx]
    indices = np.linspace(0, len(ep_frames) - 1,
                          n_frames, dtype=int)            #A

    fig, axes = plt.subplots(1, n_frames, figsize=(18, 3))
    for ax, idx in zip(axes, indices):
        img = ep_frames[idx]["observation.image"]
        ax.imshow(img.permute(1, 2, 0).numpy())           #B
        ax.set_title(f"Step {idx}")
        ax.axis("off")
    plt.tight_layout()
    plt.savefig("expert_keyframes.png", dpi=150)
    plt.show()
```
- #A Sample n_frames evenly spaced across the episode
- #B LeRobot stores images as (C, H, W) tensors — permute to (H, W, C) for display

**Listing 2.8: Comparing action distributions (expert vs. scripted vs. random)**
```python
def collect_actions(env, policy_fn, n_episodes=10):
    """Run a policy and collect all actions taken."""
    all_actions = []
    for ep in range(n_episodes):
        obs, _ = env.reset(seed=ep + 100)
        done = False
        while not done:
            action = policy_fn(obs) if policy_fn else \
                     env.action_space.sample()             #A
            all_actions.append(action.copy())
            obs, _, terminated, truncated, _ = env.step(action)
            done = terminated or truncated
    return np.array(all_actions)

expert_actions = np.stack(                                 #B
    [dataset[i]["action"].numpy() for i in range(len(dataset))]
)
scripted_actions = collect_actions(env, scripted_policy)
random_actions = collect_actions(env, None)

fig, axes = plt.subplots(1, 3, figsize=(15, 4))
for ax, actions, title in zip(
    axes,
    [expert_actions, scripted_actions, random_actions],
    ["Expert", "Scripted", "Random"]
):
    ax.scatter(actions[:, 0], actions[:, 1],
               alpha=0.1, s=1)                             #C
    ax.set_title(title)
    ax.set_xlabel("Δx"); ax.set_ylabel("Δy")
    ax.set_xlim(-1, 1); ax.set_ylim(-1, 1)
    ax.set_aspect("equal")
plt.tight_layout()
plt.savefig("action_distributions.png", dpi=150)
```
- #A None means random policy — sample from action space
- #B Stack all expert actions from the dataset
- #C Low alpha reveals density structure in the distributions

**Figure 2.3: Expert Episode Keyframes**
- A filmstrip of 6 keyframes from a single expert episode, showing progressive alignment of the T-block with the target
- Caption: "Keyframes from one expert episode. The end-effector approaches the T-block, makes contact, and pushes it into alignment with the target pose. The sequence illustrates the multi-step reasoning that a learned policy must capture."

**Figure 2.4: Action Distributions — Expert vs. Scripted vs. Random**
- Three side-by-side 2D scatter plots of (Δx, Δy) actions
- Caption: "Action distributions for the expert demonstrations (left), scripted heuristic (center), and random agent (right). Expert actions show structured, multi-modal clusters corresponding to different approach strategies. The scripted policy produces a simpler pattern. Random actions fill the space uniformly. A learned policy must capture the expert's structured distribution."

**Figure 2.5: Expert Trajectories in State Space**
- Multiple expert end-effector paths plotted in (x, y) with T-block start and target markers
- Caption: "End-effector trajectories from five expert episodes. Each episode starts from a different initial configuration, producing a different path. A successful policy must generalize across these initial conditions rather than memorizing a single trajectory."

**Transition:** "You have the environment, a baseline, and expert data. The final step is packaging that data into a form your neural network can consume."

---

## Section 2.5: The Data Pipeline

**Purpose:** Build the DataLoader wrapper that Chapter 3 will use. Implement normalization from first principles, then connect to LeRobot's stats. This section's output is the chapter's API contract with downstream chapters.

**Target length:** ~5 pages

**Content:**

### 2.5.1 Why Normalize?
- Neural networks train faster and more stably when inputs are zero-centered and unit-scaled
- The agent position might range from 0-512 while actions range from -1 to 1 — different scales cause uneven gradient flow
- Two common strategies: z-score (mean/std) and min-max (to [0, 1] or [-1, 1])

### 2.5.2 Computing Statistics from Scratch
- Iterate through the dataset and compute per-feature mean, std, min, max
- Implement `normalize(x, stats)` and `denormalize(x, stats)` functions
- Verify round-trip: `denormalize(normalize(x)) ≈ x`

### 2.5.3 LeRobot's meta/stats.json
- Reveal that LeRobot ships precomputed statistics in `meta/stats.json`
- Load them and compare against the manually computed values — they should match
- Going forward, use the precomputed stats for convenience, but the reader now understands what they contain and why

### 2.5.4 The DataLoader Wrapper
- Wrap the LeRobot dataset in a PyTorch DataLoader with batching and shuffling
- Apply normalization inside the collate function or as a transform
- Export the function `make_pusht_dataloader(batch_size, split)` — the API that Chapter 3 imports

### 2.5.5 Verifying the Pipeline
- Draw a batch, print shapes, confirm normalization (mean ≈ 0, std ≈ 1)
- Denormalize and verify values are back in the original range
- This is the "smoke test" that ensures the pipeline is correct before training

**Listing 2.9: Computing normalization statistics manually**
```python
import torch

def compute_stats(dataset):
    """Compute per-feature mean, std, min, max."""
    all_states = []
    all_actions = []
    for i in range(len(dataset)):
        frame = dataset[i]
        all_states.append(frame["observation.state"])      #A
        all_actions.append(frame["action"])

    states = torch.stack(all_states)
    actions = torch.stack(all_actions)

    stats = {
        "observation.state": {
            "mean": states.mean(dim=0),
            "std": states.std(dim=0),
            "min": states.min(dim=0).values,
            "max": states.max(dim=0).values,
        },
        "action": {
            "mean": actions.mean(dim=0),
            "std": actions.std(dim=0),
            "min": actions.min(dim=0).values,
            "max": actions.max(dim=0).values,
        },
    }
    return stats
```
- #A Accumulate all observation and action tensors

**Listing 2.10: Normalize and denormalize functions**
```python
def normalize(x, stats, key):
    """Z-score normalization: (x - mean) / std."""
    return (x - stats[key]["mean"]) / (stats[key]["std"] + 1e-8)  #A

def denormalize(x, stats, key):
    """Inverse of z-score normalization."""
    return x * (stats[key]["std"] + 1e-8) + stats[key]["mean"]

# Round-trip verification
sample = dataset[0]["observation.state"]
normed = normalize(sample, stats, "observation.state")
recovered = denormalize(normed, stats, "observation.state")
assert torch.allclose(sample, recovered, atol=1e-5)       #B
```
- #A Add epsilon to prevent division by zero for constant features
- #B Verify the round-trip is lossless (up to float precision)

**Listing 2.11: Building the DataLoader — the Ch3 API contract**
```python
from torch.utils.data import DataLoader

def make_pusht_dataloader(batch_size=64, shuffle=True):
    """Create a normalized DataLoader for PushT expert data.

    Returns:
        dataloader: PyTorch DataLoader yielding normalized batches
        stats: dict of per-feature normalization statistics
    """
    dataset = LeRobotDataset("lerobot/pusht")
    stats = compute_stats(dataset)                         #A

    def collate_fn(batch):
        collated = {}
        for key in batch[0].keys():
            vals = [b[key] for b in batch]
            if isinstance(vals[0], torch.Tensor):
                stacked = torch.stack(vals)
                if key in stats:                           #B
                    stacked = normalize(stacked, stats, key)
                collated[key] = stacked
            else:
                collated[key] = torch.tensor(vals)
        return collated

    dataloader = DataLoader(
        dataset,
        batch_size=batch_size,
        shuffle=shuffle,
        collate_fn=collate_fn,
        num_workers=4,                                     #C
    )
    return dataloader, stats

# Smoke test
loader, stats = make_pusht_dataloader(batch_size=32)
batch = next(iter(loader))
print(f"Batch observation.state: "
      f"shape={batch['observation.state'].shape}, "
      f"mean={batch['observation.state'].mean(0)}")        #D
```
- #A Compute stats once; reuse for normalize/denormalize
- #B Only normalize tensor features that have computed stats
- #C Parallel data loading for training speed
- #D After normalization, per-feature mean should be close to 0

**Callout Box: "Z-SCORE vs. MIN-MAX NORMALIZATION"**
- Z-score: `(x - mean) / std` — centers at 0, scales by spread. Preferred when features are roughly Gaussian.
- Min-max: `(x - min) / (max - min)` — scales to [0, 1]. Preferred when you need bounded outputs.
- This book uses z-score for state and actions. Image pixels are scaled to [0, 1] by dividing by 255.

**Callout Box: "THE CHAPTER 3 CONTRACT"**
- Chapter 3 imports `make_pusht_dataloader()` and `denormalize()`
- It expects batches with keys `observation.state` (normalized) and `action` (normalized)
- After the model predicts normalized actions, `denormalize()` converts them back to environment-scale for `env.step()`
- If you modify the pipeline, downstream chapters will break — treat this function signature as frozen

**Figure 2.6: The Normalization Round-Trip**
- Diagram: raw data → normalize(x, stats) → zero-centered data → model prediction → denormalize(ŷ, stats) → environment-scale action → env.step()
- Caption: "The normalization round-trip. Raw observations and actions are z-score normalized before entering the model. Predicted actions are denormalized back to environment scale before being sent to the simulator. The stats dictionary bridges both directions."

**Figure 2.7: End-to-End Data Pipeline**
- Flow diagram: Hugging Face Hub → LeRobotDataset → compute_stats() → DataLoader (with normalize in collate) → training batch {obs, action} → Chapter 3 model
- Caption: "The complete data pipeline built in this chapter. Expert demonstrations are loaded from the Hub, normalization statistics are computed, and a DataLoader applies normalization at batch time. The `make_pusht_dataloader()` function encapsulates this entire flow as the API contract for Chapter 3."

---

## Section 2.6: Summary

**Target length:** ~1 page

Comprehensive bulleted summary:

- PushT is a 2D pushing task where a circular end-effector must push a T-shaped block to match a target pose. It serves as the primary training environment for Chapters 2–5.
- The Gymnasium API provides a universal interface: `reset()` returns an initial observation, `step(action)` returns the next observation, reward, and termination flags. Every environment in this book uses this interface.
- A random agent establishes the performance floor. A scripted heuristic (move toward block, push toward target) improves significantly but fails on rotations and recovery — demonstrating why learned policies are necessary.
- Expert demonstrations are stored in LeRobot's dataset format on Hugging Face Hub. Each frame contains observation state, actions, episode and frame indices, and timestamps.
- LeRobot's `delta_timestamps` mechanism enables requesting data at relative time offsets, which is the foundation for action chunking in later chapters.
- Expert action distributions show structured, multi-modal patterns that neither random sampling nor simple heuristics can reproduce. Visualizing this gap motivates the learning approach.
- Neural networks require normalized inputs for stable training. Z-score normalization (`(x - mean) / std`) is applied to both observations and actions, with denormalization used to convert predictions back to environment scale.
- The `make_pusht_dataloader()` function is the chapter's primary export and the API contract for Chapter 3. It returns a DataLoader yielding normalized batches and the statistics dictionary needed for denormalization.

---

## Listings Summary

| Listing | Title | Section |
|---------|-------|---------|
| 2.1 | Installing LeRobot and creating the PushT environment | 2.1 |
| 2.2 | Running a random agent in PushT | 2.1 |
| 2.3 | A scripted push-toward-target policy | 2.2 |
| 2.4 | Evaluating the scripted policy | 2.2 |
| 2.5 | Loading the PushT expert dataset | 2.3 |
| 2.6 | Inspecting a single frame and episode | 2.3 |
| 2.7 | Rendering expert episode keyframes | 2.4 |
| 2.8 | Comparing action distributions (expert vs. scripted vs. random) | 2.4 |
| 2.9 | Computing normalization statistics manually | 2.5 |
| 2.10 | Normalize and denormalize functions | 2.5 |
| 2.11 | Building the DataLoader — the Ch3 API contract | 2.5 |

## Figure Summary

| Figure | Description | Type | Section |
|--------|------------|------|---------|
| 2.1 | The PushT Task (annotated environment screenshot) | Annotated screenshot | 2.1 |
| 2.2 | The Gymnasium Loop (reset/step flow) | Flow diagram | 2.1 |
| 2.3 | Expert Episode Keyframes (filmstrip) | Image sequence | 2.4 |
| 2.4 | Action Distributions — Expert vs. Scripted vs. Random | Scatter/density plots | 2.4 |
| 2.5 | Expert Trajectories in State Space | Trajectory plot | 2.4 |
| 2.6 | The Normalization Round-Trip | Flow diagram | 2.5 |
| 2.7 | End-to-End Data Pipeline | Flow diagram | 2.5 |

## Callout Box Summary

| Callout | Section | Purpose |
|---------|---------|---------|
| "WHAT IS GYMNASIUM?" | 2.1 | Define the simulation API for ML readers |
| "WHY NOT JUST ENGINEER A BETTER HEURISTIC?" | 2.2 | Connect to Chapter 1's long-tail argument |
| "WHAT IS delta_timestamps?" | 2.3 | Explain LeRobot's temporal indexing mechanism |
| "Z-SCORE vs. MIN-MAX NORMALIZATION" | 2.5 | Normalization strategy rationale |
| "THE CHAPTER 3 CONTRACT" | 2.5 | Define the API boundary between chapters |
| (Reserve) | — | Available for reviewer-requested additions |

## Table Summary

| Table | Description | Section |
|-------|-------------|---------|
| 2.1 | PushT Observation and Action Spaces | 2.1 |
| 2.2 | LeRobot PushT Dataset Feature Summary | 2.3 |

## Ch2 Exports for Ch3

The following symbols are the chapter's public API — Chapter 3 imports these directly:

| Export | Type | Purpose |
|--------|------|---------|
| `make_pusht_dataloader(batch_size, shuffle)` | function | Returns `(DataLoader, stats_dict)` with normalized batches |
| `normalize(x, stats, key)` | function | Z-score normalize a tensor using precomputed stats |
| `denormalize(x, stats, key)` | function | Inverse z-score to recover environment-scale values |
| `stats` | dict | Per-feature `{mean, std, min, max}` for `observation.state` and `action` |

---

## Estimated Length: 22–25 pages (Manning format)
## Estimated Word Count: ~9,000–11,000 words
