# 2

# Simulation & Data

This chapter covers

- Setting up the SO-101 simulation environment using ManiSkill3 and the Gymnasium interface
- Observations, actions, episodes, and rewards for a 6-DOF arm with a parallel-jaw gripper
- Writing a scripted pick-and-place policy as an explicit state machine, and observing its failure modes
- Loading, inspecting, and visualizing expert demonstration data from the LeRobot Hub
- Building a normalized DataLoader that becomes the data contract for chapter 3

A robot policy is only as good as the data that trains it, and only as transferable as the embodiment that produces that data. Before you write a single line of model code, you need three things: a simulated arm to act in, demonstrations from that arm to learn from, and a pipeline that feeds both into training. The sections that follow build all three, on the embodiment you will use across every remaining chapter.

The task is *PickCubeSO100-v1*, a simulated pick-and-place job for a 6-DOF (six degrees of freedom) arm with a parallel-jaw gripper (two opposing fingers that close symmetrically to pinch an object). The arm must approach a cube on a workspace, grasp it, lift it clear, transport it above a target zone, and release. It looks simple, and that is the point. Pick-and-place is complex enough to expose why hand-coded heuristics struggle, with contact dynamics, gripper timing, and recovery from misalignment all conspiring against a few rules, yet simple enough to train on a laptop in an evening. After section 2.5 you have a working data pipeline that chapter 3 plugs directly into, and an embodiment that does not change again until chapter 11.

ManiSkill3 and the Gymnasium interface come first; a random agent establishes the performance floor. A scripted state-machine policy then shows where hand-coded rules plateau and why. Expert demonstrations from the LeRobot Hub follow, including the `delta_timestamps` mechanism that chapters 4 and 5 use for action chunking. The expert data is visualized side by side with the scripted and random baselines, making the policy gap concrete. Normalization statistics, the `normalize` and `denormalize` functions, and the `make_pickplace_dataloader` export that chapter 3 imports unchanged close out the chapter.

![Figure 2.1 The book's five-part progression with chapter 2 highlighted. Chapter 2 builds the simulation environment, scripted-policy baseline, and normalized data pipeline that every later chapter consumes. The embodiment and data interface are fixed by the end of section 2.5 and stay fixed for the remaining nine chapters.](figures/figure_2_1_roadmap.png)

> **NOTE — How to follow along.** Two reader-facing paths are shipped with the chapter 2 companion repo. The canonical path is the Jupyter notebook at `notebooks/ch02.ipynb`. The prose maps cell by cell to that notebook, and we promise it will run end to end on a fresh kernel. The experimental path is a Claude Code companion agent (`agents/chapter-02-guide.md`) that walks you through the same listings in dialogue, useful if you want to ask clarification questions or get unstuck. Tests in `tests/` are author and CI infrastructure; you can run them as an install smoke check, but engaging with them is not part of the curriculum.

## 2.1 The SO-101 pick-and-place environment

Before any data work, you need an environment. The simulated arm needs to live somewhere it can move, the gripper needs something to grasp, and you need an interface to send commands and read state. ManiSkill3 provides all of this, and the Gymnasium interface that wraps it is the same interface every later chapter uses, in simulation and on hardware. This section installs the tooling, builds an environment instance, and runs a random agent to confirm that everything works and to set a baseline.

### 2.1.1 Installation and first launch

The two packages you need are `lerobot` for the dataset side and `mani-skill` for the simulator side. Both run on Python 3.12. Both pull in their own dependencies (`torch`, `numpy`, `gymnasium`, `sapien`), so a minute or two of install time is normal on a fresh environment.

> **PITFALL — SAPIEN and Vulkan setup on Colab.** ManiSkill3 renders through SAPIEN, which needs Vulkan installable client drivers (ICDs). Colab runtimes do not ship them by default. The symptom is a confusing `vk::Result::eErrorIncompatibleDriver` or `Cannot find any valid ICD` traceback the first time you call `env.reset()`, not when ManiSkill imports. Before you `pip install mani-skill` on Colab, run the seven-line Vulkan setup recipe documented in the repo's README. The recipe is copied verbatim from ManiSkill's official quickstart notebook and works on the free-tier T4. On local Linux or macOS this is rarely an issue; the system Vulkan drivers are usually already installed.

Once the packages are installed, listing 2.1 brings up the environment.

**Listing 2.1 Installing the SO-100 sim and creating the environment.**

```python
# pip install lerobot mani-skill
import gymnasium as gym                          #A
import mani_skill.envs                           #B

env = gym.make(                                  #C
    "PickCubeSO100-v1",
    obs_mode="rgb",                              #D
    control_mode="pd_joint_delta_pos",           #E
    render_mode="rgb_array",
)
obs, info = env.reset(seed=42)                   #F
print(f"Observation keys: {list(obs.keys())}")
print(f"Action space: {env.action_space}")      #G
```

#A Import the Gymnasium API, the simulation interface every environment in the book exposes through `reset()` and `step()`.
#B Importing `mani_skill.envs` registers `PickCubeSO100-v1` and the rest of the SO-100 task family with Gymnasium.
#C Create the SO-100 pick-and-place environment instance.
#D Return RGB camera observations. Alternatives are `"state"` for vector-only, `"rgbd"` for RGB plus depth, and `"state_dict"` for vector observations broken out by source.
#E Joint-space delta actions in the same control mode the SO-100 hardware uses. The sim-to-real gap discussed in the callout below is small but real.
#F `reset` returns the initial observation dictionary and an info dictionary.
#G The action space is `Box(7,)`: six joint deltas (one per arm joint) plus one gripper command, all in the range `[-1, 1]`.

Running this prints the observation keys and the action space:

```
Observation keys: ['agent', 'extra', 'sensor_data']
Action space: Box(-1.0, 1.0, (7,), float32)
```

Exact key names depend on the ManiSkill version. Verify them with `print(obs)` if anything downstream complains about a missing key. The `agent` group contains the arm's self-awareness of its own state, called *proprioception*: joint angles, joint velocities, and gripper state. The `extra` group contains task-level observations: the cube pose, the target pose, and the end-effector pose (the position and orientation of the gripper assembly at the tip of the arm). The `sensor_data` group contains the camera images.

> **WHAT IS GYMNASIUM?** Gymnasium (formerly OpenAI Gym) is the standard Python API for simulation and reinforcement-learning environments. Every environment exposes `reset()` and `step(action)`, a universal interface regardless of the task or embodiment. LeRobot and ManiSkill3 both register their environments as Gymnasium environments, so the same code patterns work for simulation, real-hardware wrappers, and benchmark tasks across the ecosystem.

### 2.1.2 The Gymnasium interface

A Gymnasium environment is a state machine with two operations. You call `reset()` once at the start of an episode to put the environment in a known initial state and read the first observation. After that, you call `step(action)` repeatedly, each call advancing the simulation by one control timestep and returning the next observation, a scalar reward, two termination flags, and a small info dictionary.

![Figure 2.2 The Gymnasium interaction loop. The agent receives an observation, selects an action, and the environment returns the next observation, a scalar reward, and termination flags. Every environment we work with, in simulation and on hardware, exposes this interface.](figures/figure_2_2_gymnasium_loop.png)

Five concepts make up the interface:

An *observation* is what the agent sees on a given timestep. For PickCubeSO100, an observation is a dictionary containing joint positions and velocities for the six arm joints, the gripper state, the cube and target poses, and two RGB camera images: a top-down third-person view and a wrist-mounted view that moves with the gripper.

An *action* is what the agent does. PickCubeSO100 uses a 7-dimensional continuous action: six joint position deltas plus one gripper command, each in the range `[-1, 1]`. A joint delta of zero means "hold position." A positive gripper command closes the gripper; a negative gripper command opens it.

A *step* is a single (observation, action, reward, next_observation) transition, executed at the simulator's control frequency. For this environment the control frequency is 20 Hz, so one step is 50 milliseconds of simulated time.

An *episode* is a sequence of steps from `reset()` to termination. PickCubeSO100 terminates an episode either when the cube reaches the target zone (a success) or when the maximum episode length is reached (a timeout, also called a truncation). Episodes are independent; nothing carries over from one episode to the next.

A *reward* is a scalar that quantifies how good the current step was. PickCubeSO100 uses a shaped reward: a continuous signal that combines the distance from the gripper to the cube while approaching, a lift bonus once the cube has been grasped, the distance from the cube to the target during transport, and a discrete success bonus when the cube enters the target zone. Shaped rewards give the agent a useful signal at every step rather than only on the rare success events that a sparse reward would provide. That matters both for the heuristic in section 2.2 and for the reinforcement-learning chapter later, where exploration is hard.

![Figure 2.3 The PickCubeSO100 task. A 6-DOF arm with a parallel-jaw gripper (SO-100 URDF in simulation, SO-101 on hardware) must grasp the cube on the workspace and release it inside the target zone. The reward combines approach distance, grasp success, transport distance, and a discrete success bonus on placement.](figures/figure_2_3_pickcube_task.png)

Table 2.1 summarizes the observation and action shapes for quick reference.

**Table 2.1 SO-101 observation and action spaces.**

| Component | Shape | Type | Description |
|-----------|-------|------|-------------|
| `agent.qpos` | (6,) | float32 | Joint positions in radians |
| `agent.qvel` | (6,) | float32 | Joint velocities |
| `extra.cube_pose` | (7,) | float32 | Cube pose: (x, y, z, qw, qx, qy, qz) |
| `extra.target_pose` | (7,) | float32 | Target pose, same convention |
| `sensor_data.base_camera.rgb` | (224, 224, 3) | uint8 | Top-down RGB camera |
| `sensor_data.hand_camera.rgb` | (224, 224, 3) | uint8 | Wrist-mounted RGB camera |
| Action | (7,) | float32 | Six joint position deltas plus gripper command |

> **WHY SO-100 IN SIM, SO-101 ON HARDWARE?** ManiSkill3 ships the SO-100 as a first-class robot in `mani_skill/agents/robots/so100/`. SO-101 is the newer revision with slightly different servos and tuning. The observation and action interfaces are identical, so policy training is unaffected. The residual kinematic gap is real but small, and isolating it to a single sim-to-real chapter (chapter 9) is cleaner pedagogy than pretending it does not exist. This is the canonical sim-to-real problem in miniature, and chapter 9 is built around it.

### 2.1.3 Running a random agent

The simplest agent samples actions uniformly from the action space. A random agent solves almost no robotics tasks (more on that in a moment), but running one is the right first move. It exercises the full Gymnasium loop, surfaces any installation problems, and establishes a performance floor that every later policy must clear.

The `run_random_agent` function in listing 2.2 executes the loop with uniformly sampled actions and reports the success rate over a fixed number of episodes. This is type-along code; you write it once and then the same loop structure shows up under every learned policy we build later.

**Listing 2.2 Running a random agent on PickCubeSO100.**

```python
import numpy as np

def run_random_agent(env, n_episodes=10):
    successes, returns = 0, []
    for ep in range(n_episodes):
        obs, info = env.reset(seed=ep)
        ep_return = 0.0
        done = False
        while not done:
            action = env.action_space.sample()           #A
            obs, reward, terminated, truncated, info = env.step(action)
            ep_return += reward
            done = terminated or truncated
        successes += int(info.get("is_success", False)) #B
        returns.append(ep_return)
    return successes / n_episodes, np.mean(returns)

success_rate, mean_return = run_random_agent(env)
print(f"Random agent: success={success_rate:.0%} "
      f"return={mean_return:.2f}")                       #C
```

#A Sample uniformly from the 7-DOF continuous action space.
#B The environment reports success when the cube has reached the target zone for several consecutive steps.
#C Expect a near-zero success rate. Flailing the arm rarely grasps anything, much less transports it.

Running this prints something like:

```
Random agent: success=0% return=-0.42
```

Numbers vary by seed; what matters is that the success rate is effectively zero and the return is negative. The return is negative because the shaped reward penalizes distance from the cube and from the target, and the random agent spends most of its time far from both. A negative return is informative: it tells you the reward signal is dense, not sparse, which matters for the scripted policy in §2.2 and the RL training in chapter 7.

> **EXERCISE 2.1: Joint-space vs. end-effector-space control.** Re-create the environment with `control_mode="pd_ee_delta_pose"` instead of the default joint-space mode and re-run the random agent. The action space changes from `Box(6,)` to a Cartesian delta pose plus gripper. Do random rollouts succeed any more often? Why or why not? *Tip: end-effector control hides one form of difficulty (joint coordination) while exposing another (workspace boundaries). The random agent benefits from neither.*

A random agent flailing the arm produces a 0% success rate. That is the floor. The next obvious move is to write a smart rule, and watching even a *smart* rule struggle is what motivates the rest of the book.

## 2.2 A scripted policy

A pick-and-place task decomposes naturally into phases. The gripper has to be above the cube, then on it, then closed around it, then up, then over the target, then down, then open. Each phase has a clear geometric goal and a clear transition condition. So why not write it down?

This section does exactly that. You build a state-machine controller that walks through seven phases in order. You watch it solve the task in nominal conditions, watch it fail in interesting ones, and read off the lesson: even a thoughtful heuristic plateaus far below expert performance. Learned policies pick up where rules stop.

### 2.2.1 Designing the heuristic

The seven phases are *approach*, *descend*, *grasp*, *lift*, *transport*, *place*, and *release*. Each phase has a target position for the end-effector and a transition condition: a distance threshold, a height threshold, or a frame count. The controller carries a small state dictionary across calls. On each step, it reads the current observation, looks at the current phase, computes a target end-effector position for that phase, derives a joint-space action that moves toward it, and advances the phase index when the transition condition fires.

Two things make this a fair baseline rather than a strawman. The phases are honest: every successful demonstration in the expert dataset (§2.3) follows the same sequence, including the *release* phase that scripts often skip. The transitions use distance thresholds tight enough that the controller commits to each phase rather than oscillating. A worse-engineered heuristic would underperform this one, but a *better*-engineered heuristic would not change the conclusion.

### 2.2.2 Implementation

Listing 2.3 is the type-along listing for this section. It implements `scripted_policy(obs, state)` as a pure function: given the current observation and the caller's state dict, it returns a 7-dimensional action and mutates `state` to advance the phase if appropriate. The caller resets the state with `state = {"phase": "approach"}` at the start of each episode.

> **NOTE — Two simplifications worth flagging up front.** Listing 2.3 reads the observation with shorthand keys (`obs["agent_pos"]`, `obs["cube_pos"]`, `obs["target_pos"]`) where the actual ManiSkill3 dictionary is nested as `obs["agent"]["qpos"]`, `obs["extra"]["cube_pose"]`, and `obs["extra"]["target_pose"]`. The companion notebook unpacks the nested keys into the shorthand names in the cell above the listing so the prose stays readable. Listing 2.3 also uses a deliberately crude Cartesian-to-joint mapping: it treats the first three components of the joint-delta vector as Cartesian (x, y, z) displacements and pads the remaining three with zeros. A production scripted policy would compute proper inverse kinematics; the version here trades correctness for clarity and still solves the task often enough to demonstrate the heuristic plateau.

**Listing 2.3 A multi-phase scripted pick-and-place policy.**

```python
import numpy as np

PHASES = ["approach", "descend", "grasp",
          "lift", "transport", "place", "release"]      #A

def scripted_policy(obs, state):
    """A simple state-machine controller for pick-and-place."""
    phase = state["phase"]
    ee_pos = obs["agent_pos"][-3:]                       #B
    cube = obs["cube_pos"]
    target = obs["target_pos"]

    if phase == "approach":                              #C
        goal = cube + np.array([0.0, 0.0, 0.10])
        gripper = -1.0
        if np.linalg.norm(ee_pos - goal) < 0.01:
            state["phase"] = "descend"
    elif phase == "descend":
        goal = cube + np.array([0.0, 0.0, 0.005])
        gripper = -1.0
        if abs(ee_pos[2] - goal[2]) < 0.005:
            state["phase"] = "grasp"
    elif phase == "grasp":                               #D
        goal = ee_pos
        gripper = 1.0
        state["grasp_steps"] = state.get("grasp_steps", 0) + 1
        if state["grasp_steps"] >= 5:
            state["phase"] = "lift"
    elif phase == "lift":
        goal = cube + np.array([0.0, 0.0, 0.15])
        gripper = 1.0
        if ee_pos[2] >= goal[2] - 0.01:
            state["phase"] = "transport"
    elif phase == "transport":
        goal = target + np.array([0.0, 0.0, 0.15])
        gripper = 1.0
        if np.linalg.norm(ee_pos[:2] - goal[:2]) < 0.01:
            state["phase"] = "place"
    elif phase == "place":
        goal = target + np.array([0.0, 0.0, 0.02])
        gripper = 1.0
        if abs(ee_pos[2] - goal[2]) < 0.005:
            state["phase"] = "release"
    else:  # release
        goal = ee_pos
        gripper = -1.0                                   #E

    direction = goal - ee_pos
    cartesian_delta = np.clip(direction * 5.0, -1.0, 1.0)  #F
    return np.concatenate([cartesian_delta,
                           np.zeros(3),
                           [gripper]]).astype(np.float32)
```

#A The seven phases in execution order. *Transport* stops at altitude above the target; *place* handles the final descent. Splitting them lets each transition use an independent condition (horizontal distance for transport, vertical distance for place).
#B The end-effector position lives in the last three slots of the shorthand `agent_pos` vector (unpacked from `obs["agent"]["qpos"]` in the notebook cell above this one).
#C *Approach* hovers 10 cm above the cube with the gripper open.
#D *Grasp* holds position and closes the gripper for several frames so contact stabilizes before lifting.
#E *Release* opens the gripper at the target.
#F Pack the desired Cartesian motion into the first three slots of the action; the remaining three joints get zero, and the gripper occupies slot seven. The result is a `(7,)` action that matches `env.action_space`.

Listing 2.4 wraps this into the same episode-loop pattern as the random-agent eval.

**Listing 2.4 Evaluating the scripted policy.**

```python
def run_scripted_agent(env, n_episodes=10):
    successes = 0
    for ep in range(n_episodes):
        obs, info = env.reset(seed=ep)
        state = {"phase": "approach"}                    #A
        done = False
        while not done:
            action = scripted_policy(obs, state)
            obs, reward, terminated, truncated, info = env.step(action)
            done = terminated or truncated
        successes += int(info.get("is_success", False))
    return successes / n_episodes

rate = run_scripted_agent(env)
print(f"Scripted agent success rate: {rate:.0%}")       #B
```

#A The state machine carries its phase across steps via this dict, reset at the start of each episode.
#B Expect a moderate success rate, better than random and well below expert.

Running this prints something like:

```
Scripted agent success rate: 50%
```

Anywhere from roughly 30% to 70% is normal. The exact rate depends on which cube spawn positions the random seeds produce. The point is that the success rate is substantially better than 0% and substantially below 100%: the heuristic plateau is real, and you have just measured it.

### 2.2.3 Where the heuristic fails

The scripted policy succeeds when nothing surprises it. It fails in three predictable ways. *Grasp timing*: the gripper closes a frame too early or too late, the cube slips, and the *lift* phase ends with nothing to lift. *Workspace edges*: the cube spawns near the reachable boundary, the arm cannot fully orient over it, and the *descend* phase tries to drive through a joint limit. *Contact perturbations*: the descent pushes the cube as it touches, the cube rolls out from under the gripper, and the controller has no way to notice or react.

All three failures share a structure. The policy is *open-loop* within each phase: it commits to a fixed plan without reacting to sensor feedback mid-phase. Once the *descend* phase starts, the controller will keep descending whether or not the cube is still where it expects. A learned policy can be conditional on what the cameras actually see in the moment, including the state of the cube as it is being touched. That conditionality is what behavior cloning learns to replicate from the expert demonstrations in chapter 3.

> **WHY NOT JUST ENGINEER A BETTER HEURISTIC?** You could add error recovery, force feedback, retry-on-miss, and a finer phase decomposition. Every improvement is a few lines of code. The problem is the long tail: each new edge case (different cube color, novel target position, occlusion, contact perturbation) compounds the complexity, and the engineering effort grows faster than the success rate. This is the long-tail argument from chapter 1 in miniature. Heuristics plateau; learned policies keep improving with more data.

> **EXERCISE 2.2: Grasp-failure recovery.** Extend `scripted_policy` with a new phase that detects when the cube has not risen above a height threshold after the *lift* phase, returns to *approach*, and retries the grasp. How much does the success rate improve? At what point does adding phases stop helping? *Tip: instrument `state` with a `retries` counter and bail out at three retries to avoid infinite loops.*

The scripted policy shows what one person's intuition can achieve in an afternoon. Expert demonstrations show what practiced teleoperation looks like as data.

## 2.3 The LeRobot dataset standard

Expert demonstrations are recorded teleoperation. A human operator drives a real or simulated SO-100 through dozens of pick-and-place episodes, and at every control timestep the system saves the observation the operator saw and the command they issued. The result is a stream of (observation, action) pairs grouped into episodes. With enough such pairs, a learned policy can imitate the distribution they came from.

The LeRobot project, maintained by Hugging Face, defines a standard format for this kind of data and hosts hundreds of datasets on the Hugging Face Hub, a public model and dataset registry. This section loads an SO-100 pick-and-place dataset, walks through its feature schema, and introduces the `delta_timestamps` mechanism that chapter 4 uses for action chunking.

### 2.3.1 Loading from the Hub

`LeRobotDataset` is the entry point. You pass it a Hugging Face Hub dataset identifier, it downloads the parquet shards and image archives once (caching them under `~/.cache/huggingface/`), and it gives you back a PyTorch-style indexed dataset. Each integer index returns one frame, with all of that frame's features in a single dictionary.

**Listing 2.5 Loading the SO-101 pick-and-place expert dataset.**

```python
from lerobot.common.datasets.lerobot_dataset import LeRobotDataset

dataset = LeRobotDataset(
    "lerobot/svla_so101_pickplace",                     #A
)
print(f"Total frames: {len(dataset)}")                  #B
print(f"Episodes: {dataset.num_episodes}")
print(f"Features: {list(dataset.features.keys())}")     #C
```

#A The dataset identifier on the Hugging Face Hub. The exact dataset for the SO-101 carrier task is pinned in the companion repo's `docs/decisions/` and refreshed when better data lands.
#B The total number of (observation, action) frames across all episodes.
#C The feature names include the state vector, two camera streams, the action, and trajectory metadata.

Running this prints something like:

```
Total frames: 18421
Episodes: 50
Features: ['observation.state', 'observation.images.top',
           'observation.images.wrist', 'action', 'episode_index',
           'frame_index', 'timestamp', 'next.done']
```

The frame and episode counts depend on which dataset is pinned. What matters is the shape of the feature list: a state vector, two image streams, an action, and trajectory metadata. That is the canonical layout for every LeRobot manipulation dataset, and every later chapter assumes it.

### 2.3.2 The feature schema

A single frame looks like this. Listing 2.6 indexes into the dataset and prints the shape and dtype of each feature in a sample frame, then collects every frame belonging to episode 0 so you can see what one trajectory looks like end to end.

**Listing 2.6 Inspecting a single frame and one episode.**

```python
frame = dataset[0]                                      #A
for key, val in frame.items():
    if hasattr(val, "shape"):
        print(f"  {key}: shape={val.shape}, dtype={val.dtype}")
    else:
        print(f"  {key}: {val}")

ep_indices = [i for i in range(len(dataset))            #B
              if dataset[i]["episode_index"] == 0]
print(f"\nEpisode 0 length: {len(ep_indices)} steps")
```

#A Access a single frame by integer index. The returned dictionary mixes tensors (the observation, the action) with scalars (the indices and timestamp).
#B Collect all frame indices belonging to episode 0 to inspect a complete trajectory.

The output:

```
  observation.state: shape=torch.Size([7]), dtype=torch.float32
  observation.images.top: shape=torch.Size([3, 224, 224]), dtype=torch.uint8
  observation.images.wrist: shape=torch.Size([3, 224, 224]), dtype=torch.uint8
  action: shape=torch.Size([7]), dtype=torch.float32
  episode_index: 0
  frame_index: 0
  timestamp: 0.0

Episode 0 length: 348 steps
```

Two things are worth noticing. Images are stored channel-first, as `(C, H, W)` tensors. This is the PyTorch convention; matplotlib wants channel-last, so you have to call `.permute(1, 2, 0)` before plotting. Episode lengths vary, typically between 200 and 500 frames, because teleoperators complete the task at different speeds depending on the cube position.

Table 2.2 collects the schema for reference.

**Table 2.2 LeRobot SO-101 pick-and-place dataset features.**

| Feature | Shape | Type | Description |
|---------|-------|------|-------------|
| `observation.state` | (7,) | float32 | Six joint positions plus gripper state |
| `observation.images.top` | (3, 224, 224) | uint8 | Top-down third-person camera |
| `observation.images.wrist` | (3, 224, 224) | uint8 | Wrist-mounted camera |
| `action` | (7,) | float32 | Recorded teleoperation command |
| `episode_index` | scalar | int64 | Episode this frame belongs to |
| `frame_index` | scalar | int64 | Position within the episode |
| `timestamp` | scalar | float32 | Seconds from episode start |

### 2.3.3 Understanding delta_timestamps

For the listings above, each call to `dataset[i]` returns one frame. That is enough for plain behavior cloning, where the model predicts the action at time `t` from the observation at time `t`. Later chapters need *temporal context*: chapter 4's action chunking predicts a short sequence of future actions instead of a single one. Both need to ask for "the action at this frame plus the next two" or "the observation at this frame plus the previous one."

LeRobot exposes this through a `delta_timestamps` argument on the dataset constructor. You pass a dictionary mapping feature names to lists of time offsets in seconds. For each key in that dictionary, the dataset stacks the requested frames into a single tensor along a new leading dimension.

> **WHAT IS delta_timestamps?** Setting `delta_timestamps={"action": [0.0, 0.05, 0.10]}` returns the current action plus the next two future actions, stacked into shape `(3, 7)`. The numeric offsets are in seconds; at the SO-100 sim's 20 Hz control rate, 0.05 s is one frame. This enables action chunking, predicting a short sequence of future actions instead of one at a time, and it is the data-side foundation for the ACT and diffusion-policy heads in chapters 4 and 5.

You do not need to use `delta_timestamps` for the work that follows. The DataLoader built in §2.5 yields single-frame batches, and chapter 3's first model trains on those. Action chunking in chapter 4 is where `delta_timestamps` first appears in a training loop. For now, knowing the mechanism exists is enough.

> **EXERCISE 2.3: Single-episode statistics.** Compute the mean and standard deviation of `observation.state` across the frames of a single episode, then compare to the statistics across the whole dataset. How large is the discrepancy on the joint dimensions? On the gripper? *Tip: this is the basic argument for normalizing on the dataset, not per-batch. It also foreshadows why a tiny fine-tuning dataset can destabilize a model that was trained with population-level statistics.*

Numbers in a table tell you the data's shape. Plots and rendered frames tell you what success looks like.

## 2.4 Visualizing the data

Building intuition for expert behavior is the cheap, important step that most teams skip. Before you train any model on this data, you should have looked at it: at images from a few successful trajectories, at the distribution of actions the experts used, at the variation across episodes. The discrepancies you find here predict where a learned policy will and will not generalize. The patterns you find here are what a behavior cloning policy is being asked to reproduce.

This section produces three visualizations using provided utilities from `ch02.viz`: a two-row filmstrip of an expert episode, a per-joint histogram comparing expert, scripted, and random action distributions, and a plot of expert joint trajectories across several episodes.

### 2.4.1 Rendering expert episodes

The first thing to look at is what one successful trajectory looks like. `render_keyframes` samples evenly spaced frames from one episode and tiles the top-down and wrist-camera views into a two-row figure. The top row shows the macroscopic motion of the arm; the bottom row shows the contact-level detail of the grasp. Listing 2.7 is a provided utility imported from `ch02.viz`; the call site is shown here, and the implementation lives in the chapter's source package for transparency.

**Listing 2.7 Rendering expert keyframes from both camera views.**

```python
from ch02.viz import render_keyframes

fig = render_keyframes(dataset, episode_idx=0, n_frames=6)
fig.savefig("figures/figure_2_4_expert_keyframes.png", dpi=300)
```

The underlying implementation iterates over the dataset, filters to episode 0, samples six evenly spaced indices, and renders both camera views into a 2×6 matplotlib grid. It returns the `Figure` object so the notebook can both render it inline and save it to disk.

![Figure 2.4 Keyframes from one expert episode. The top-down view shows the macroscopic motion of the arm; the wrist view shows the contact-level detail of the grasp. A learned policy must capture both perspectives to handle objects whose position is only partially visible from above.](figures/figure_2_4_expert_keyframes.png)

What you see in figure 2.4 is one successful pick-and-place. The arm approaches from above, descends to make contact, the gripper closes, and the cube travels with the gripper to the target zone. The wrist view is especially informative: in the third and fourth frames you can see the gripper closing around the cube from the camera's own perspective, the kind of close-range visual feedback a learned policy can use to time its grasp.

### 2.4.2 Action distributions

A single trajectory tells you what success looks like. The *distribution* of actions across many trajectories tells you what kind of patterns a policy has to capture. The `plot_action_distributions` helper in `ch02.viz` overlays per-dimension histograms from the three action sources. Listing 2.8 shows the call site; the histogram block beneath collects actions from the expert dataset (type-along), runs the scripted policy and random agent, and hands all three to the plotting utility.

**Listing 2.8 Per-joint action distributions, expert vs. scripted vs. random.**

```python
from ch02.viz import collect_actions, plot_action_distributions

expert = np.stack([dataset[i]["action"].numpy()
                   for i in range(len(dataset))])
scripted = collect_actions(env, scripted_policy)
random_ = collect_actions(env, policy_fn=None)

fig = plot_action_distributions(expert, scripted, random_)
fig.savefig("figures/figure_2_5_action_distributions.png", dpi=300)
```

![Figure 2.5 Action distributions for each of the seven action dimensions. Expert actions show structured, multimodal clusters that reflect different grasp strategies. The scripted policy produces a simpler, lower-variance pattern. Random actions are uniform across the range. The gap between the scripted and expert histograms is what a learned policy must close.](figures/figure_2_5_action_distributions.png)

The expert histograms have *structure*. Several of them are *multimodal* in the statistical sense (multiple peaks): one cluster of joint values for cubes that spawn on the left side of the workspace, another for cubes on the right. The gripper histogram is sharply bimodal, with most mass at fully open (`-1`) and fully closed (`+1`) and almost nothing in between, which is what you would expect from a discrete decision masked by continuous-valued reporting. The scripted distributions are simpler: each joint mostly clusters around the values the heuristic uses for its phases, and the gripper is bimodal but with sharper edges. The random distributions are flat by construction.

The gap between the scripted and expert histograms is the gap a learned policy is being asked to close. It is not just "better numbers"; it is a *different shape* of distribution. The expert spreads probability mass across many trajectory styles that all succeed. The scripted policy commits to one. A model that can represent multimodal distributions (chapters 4 and 5 both can, in different ways) is the one that can match the expert.

### 2.4.3 Expert trajectories

The third view is trajectory-level. The `plot_joint_trajectories` helper (also provided in `ch02.viz`) plots joint angles over time for several episodes, overlaid on the same axes:

```python
from ch02.viz import plot_joint_trajectories

fig = plot_joint_trajectories(dataset, episode_indices=range(5))
fig.savefig("figures/figure_2_6_joint_trajectories.png", dpi=300)
```

![Figure 2.6 Joint trajectories from five expert episodes. Episodes share the same coarse structure (approach, lift, transport, place) but diverge in timing and amplitude depending on the initial cube pose. A successful policy must be conditional on the current observation, not a memorized sequence.](figures/figure_2_6_joint_trajectories.png)

Five overlaid trajectories have the same skeleton but never the same details. The shoulder joint always rises during the *lift* phase but rises by different amounts. The base joint always rotates during the *transport* phase but by different signs depending on which side the target is on. Memorizing a single trajectory would solve none of these episodes; the policy has to be *conditional* on what the observation currently shows.

The environment is set up. A baseline is on the table. Expert demonstrations are loaded and looked at. The remaining step is to package the data into a form a neural network can train on.

## 2.5 The data pipeline

A PyTorch model is happy to consume normalized tensors in batches. The raw LeRobot dataset is none of those things: features live on different scales, frames are individually indexed rather than batched, and there is no consistent rule for how to handle images versus state vectors. The data pipeline closes those gaps. It computes normalization statistics, wraps the dataset in a `DataLoader`, and applies the right normalization to each feature type at batch time.

This section's deliverable is `make_pickplace_dataloader`, the API contract chapter 3 imports unchanged. Everything else in the section exists to explain or test it.

### 2.5.1 Why normalize?

Neural networks train faster and more stably when inputs are zero-centered and unit-scaled. The mathematical reason is that gradient descent updates weights in proportion to the magnitudes of the inputs that flow into them; when one feature ranges over a few radians and another ranges over 255 pixel intensities, the gradient is dominated by the larger-magnitude feature regardless of how informative it is. Normalization removes that asymmetry.

The choice of which normalization to apply is feature-dependent, not network-dependent. Joint positions and recorded actions live on roughly Gaussian distributions; z-score normalization (subtract the mean, divide by the standard deviation) makes them zero-mean and unit-variance. Image pixels live in the bounded integer range `[0, 255]`; min-max scaling (`x / 255`) maps them to the `[0, 1]` range that the vision encoders in chapter 3 expect. The book uses both, with z-score for state and actions, and `x / 255` for images. The choice is consistent across every chapter.

> **Z-SCORE vs. MIN-MAX NORMALIZATION.** Z-score normalization is `(x - mean) / std`. It centers data at zero and scales by spread. It is the right choice when features are roughly Gaussian, as joint angles and recorded teleop actions tend to be. Min-max normalization is `(x - min) / (max - min)`. It scales data to `[0, 1]`. It is the right choice when bounded outputs are needed, as with image pixels handed to a vision encoder that expects `[0, 1]`. The book uses z-score for state and actions and `x / 255` for images.

### 2.5.2 Computing statistics from scratch

`compute_stats` walks the dataset once and computes per-dimension mean, standard deviation, minimum, and maximum for the state vector and the action vector. Listing 2.9 is the type-along version.

**Listing 2.9 Computing normalization statistics manually.**

```python
import torch

def compute_stats(dataset):
    """Compute per-feature mean, std, min, max for state and action."""
    states, actions = [], []
    for i in range(len(dataset)):
        frame = dataset[i]
        states.append(frame["observation.state"])         #A
        actions.append(frame["action"])
    states = torch.stack(states)
    actions = torch.stack(actions)

    return {
        "observation.state": {
            "mean": states.mean(0), "std": states.std(0),
            "min": states.min(0).values, "max": states.max(0).values,
        },
        "action": {
            "mean": actions.mean(0), "std": actions.std(0),
            "min": actions.min(0).values, "max": actions.max(0).values,
        },
    }

stats = compute_stats(dataset)
```

#A Collect every state and action across the entire dataset to compute exact statistics.

The function is deliberately simple. It pulls every frame into memory, stacks them, and calls `.mean(0)`, `.std(0)`, `.min(0)`, `.max(0)` along the batch dimension. For an 18k-frame dataset, the memory footprint is a few megabytes; for million-frame datasets, you would batch the computation in a streaming pass. The dataset here is small enough that the in-memory version is the right one.

> **WHY BOTH MIN/MAX AND MEAN/STD?** The book uses mean/std for normalization. The min/max values are kept around as guardrails: the RL chapter clips predicted actions to the empirical action range before sending them to the simulator, and several downstream debugging tools compare predictions against the min/max envelope to catch obvious drift. Stashing both costs nothing and saves later refactors.

LeRobot also ships precomputed statistics for every dataset in a `meta/stats.json` file inside the dataset directory. The LeRobot 0.5.1 API exposes them via the dataset's `stats` attribute after construction. Computing the stats yourself here once means the dictionary you get back is no longer a black box; from chapter 3 onward, you can reuse the cached values.

### 2.5.3 The normalize and denormalize functions

`normalize` and `denormalize` are short, but they are the most-used functions in the rest of the book. Every model input that is not an image goes through `normalize` before training, and every action output that is not a token goes through `denormalize` before `env.step()`. Listing 2.10 is the type-along version. Pay attention to the round-trip assertion; that test catches more bugs than any post-hoc logging system.

**Listing 2.10 Normalize and denormalize functions.**

```python
def normalize(x, stats, key):
    """Z-score normalization: (x - mean) / std."""
    return (x - stats[key]["mean"]) / (stats[key]["std"] + 1e-8)  #A

def denormalize(x, stats, key):
    """Inverse z-score normalization."""
    return x * (stats[key]["std"] + 1e-8) + stats[key]["mean"]

sample = dataset[0]["observation.state"]
normed = normalize(sample, stats, "observation.state")
recovered = denormalize(normed, stats, "observation.state")
assert torch.allclose(sample, recovered, atol=1e-5)       #B
```

#A The small epsilon prevents division by zero for features that are constant across the dataset, which would otherwise produce NaNs that silently propagate through every downstream tensor.
#B Verify the round trip is lossless to floating-point precision. This single line catches stat-dict key typos, dtype mismatches, and the easiest class of normalization bug.

The functions are deliberately stateless. They take the tensor, the stats dictionary, and the key as arguments; they hold no global state of their own. That property matters when you start using them in parallel data loaders, distributed training, and inference servers, where any module-level state is a debugging trap.

![Figure 2.7 The normalization round-trip. Observations and actions are z-score normalized before entering the model. Predicted actions are denormalized back to radians and gripper commands before being sent to the simulator. The stats dictionary bridges both directions and is the only piece of training-time state that has to survive into inference.](figures/figure_2_7_normalization_roundtrip.png)

### 2.5.4 Building the DataLoader

`make_pickplace_dataloader` ties dataset, statistics, and normalization into the chapter's primary export. The function lives in `ch02.pipeline` and you import it as `from ch02.pipeline import make_pickplace_dataloader`. The listing below is shown so you understand what the function does; you do not retype it.

**Listing 2.11 Building the DataLoader: the chapter 3 API contract.**

```python
from torch.utils.data import DataLoader

def make_pickplace_dataloader(
    dataset_id="lerobot/svla_so101_pickplace",            #A
    batch_size=64,
    shuffle=True,
):
    """Return a normalized DataLoader and stats for the SO-101 task."""
    dataset = LeRobotDataset(dataset_id)
    stats = compute_stats(dataset)

    def collate_fn(batch):
        out = {}
        for key in batch[0].keys():
            vals = [b[key] for b in batch]
            if not isinstance(vals[0], torch.Tensor):
                out[key] = torch.tensor(vals)
                continue
            stacked = torch.stack(vals)
            if key in stats:                              #B
                stacked = normalize(stacked, stats, key)
            elif key.startswith("observation.images"):    #C
                stacked = stacked.float() / 255.0
            out[key] = stacked
        return out

    loader = DataLoader(dataset, batch_size=batch_size,
                        shuffle=shuffle, collate_fn=collate_fn,
                        num_workers=4)
    return loader, stats

loader, stats = make_pickplace_dataloader(batch_size=32)
batch = next(iter(loader))
print(f"state mean per dim: "
      f"{batch['observation.state'].mean(0)}")            #D
print(f"image range: [{batch['observation.images.top'].min():.2f},"
      f" {batch['observation.images.top'].max():.2f}]")
```

#A `dataset_id` is parameterized so later chapters can swap in their own datasets without changing the function signature. The default points at the SO-100 pick-and-place dataset that chapter 3 expects.
#B Z-score normalize state and action using precomputed stats.
#C Image features take a different normalization path: divide by 255 to land in `[0, 1]`.
#D After normalization, the per-dimension state mean should be near zero across a batch.

Running this smoke test prints:

```
state mean per dim: tensor([-0.01,  0.02,  0.00, -0.03,  0.01,  0.00, -0.02])
image range: [0.00, 1.00]
```

Each state-mean component should sit within roughly ±0.1 of zero (within-batch noise around the dataset-level mean of zero), and the image pixel range should land in `[0, 1]` after the `x / 255` step. Any wildly off value here means a stat-dict key was mis-spelled or a feature slipped past the collate function. That smoke test is short enough to keep at the top of every training run.

![Figure 2.8 The complete data pipeline built across sections 2.1 to 2.5. Expert demonstrations are loaded from the Hub, normalization statistics are computed once, and a DataLoader applies the right normalization to each feature type at batch time. `make_pickplace_dataloader` encapsulates the entire flow as the API contract for chapter 3.](figures/figure_2_8_data_pipeline.png)

> **THE CHAPTER 3 CONTRACT.** Chapter 3 imports `make_pickplace_dataloader`, `normalize`, and `denormalize` directly from `ch02`. The expected call site is `loader, stats = make_pickplace_dataloader(dataset_id, batch_size, shuffle)`. Each batch is a dictionary with keys `observation.state` (normalized), `observation.images.top` and `observation.images.wrist` (in `[0, 1]`), and `action` (normalized). After the model predicts a normalized action, `denormalize` converts it back to environment scale before `env.step()` runs. Treat the function signature as frozen; renaming or re-ordering arguments breaks every downstream chapter.

> **EXERCISE 2.4: Min-max normalization on actions.** Replace `compute_stats` and `normalize` with min-max scaling (`(x - min) / (max - min)`) for `observation.state` and `action` only. Verify the round trip still works to floating-point precision. Then draw one batch and compare per-dimension batch means against z-score: which is closer to zero? Why does that matter for downstream learning? *Tip: think about how the loss gradient flows through a denormalized action prediction at the start of training, before the model has learned anything.*

### 2.5.5 Pipeline performance across hardware

A reasonable concern here is *will this run on my hardware*. The honest answer is yes, comfortably, on the platforms covered in table 2.3. Mac MPS and CPU-only paths are not benchmarked below but are documented in the companion repo README.

**Table 2.3 Chapter 2 pipeline timings across hardware.**

| Step | Colab T4 (free) | RTX 4090 | A100 (Colab Pro) |
|------|-----------------|----------|------------------|
| First-time dataset download (~1.5 GB) | ~3–5 min (network bound) | ~1–2 min | ~1–2 min |
| `compute_stats` (single pass over dataset) | ~30–60 s | ~10–20 s | ~10–20 s |
| One pipeline smoke (batch of 32 with images) | < 2 s | < 1 s | < 1 s |
| Random-agent rollout (10 episodes) | ~20–40 s | ~10–15 s | ~10–15 s |
| Scripted-agent rollout (10 episodes) | ~30–60 s | ~15–25 s | ~15–25 s |

Numbers are wall-clock approximations measured with `lerobot 0.5.1` and `mani-skill 3.0.1`; rerun on first install and after any major dependency update. The network-bound dataset download dominates the first-run experience on every platform. Subsequent runs in the same kernel session have no setup overhead because the dataset and simulator stay loaded in memory. The Vulkan setup on Colab is a one-time ~20–30 s cell at the top of the notebook. No training happens here, so GPU memory pressure is not yet a concern; chapter 3 is where that conversation starts.

## 2.6 Summary

The work above built the simulation and data foundation that every later chapter consumes.

- `PickCubeSO100-v1` is a single-object pick-and-place task on a 6-DOF arm with a parallel-jaw gripper, served by ManiSkill3 over SAPIEN. It is the carrier task and the carrier embodiment for the rest of the book.
- The Gymnasium API provides a universal interface: `reset()` returns an initial observation, `step(action)` returns the next observation, a scalar reward, and termination flags. Every environment we work with, in simulation and on hardware, exposes this interface.
- A random agent on a 6-DOF arm almost never succeeds. A multi-phase scripted policy (approach, descend, grasp, lift, transport, place, release) raises the success rate but plateaus far below expert performance because it is open-loop within each phase and cannot recover from misalignment or contact perturbations.
- Expert demonstrations from teleoperation are stored in the LeRobot dataset format on the Hugging Face Hub. Each frame includes the joint state, two camera views, the recorded action, and trajectory metadata. The `delta_timestamps` mechanism enables requesting data at relative time offsets and is the data-side foundation for action chunking in chapters 4 and 5.
- Visualizing expert action distributions per-joint reveals structured, multimodal patterns that neither random sampling nor a hand-coded heuristic can reproduce. Visualizing expert joint trajectories shows that successful policies must be conditional on the current observation, not memorized sequences.
- Neural networks need normalized inputs. Z-score normalization is applied to state and action; images are scaled to `[0, 1]`. Denormalization recovers environment-scale actions for use with `env.step()`. The round-trip assertion in listing 2.10 catches the entire easy class of normalization bug.
- The chapter's primary export is `make_pickplace_dataloader(dataset_id, batch_size, shuffle)`, parameterized on `dataset_id` so later chapters can swap in custom datasets without changing the interface. Together with `normalize` and `denormalize`, it is the frozen data contract between chapter 2 and chapter 3.
- Chapter 3 picks up exactly where the data pipeline ends: the same DataLoader, the same normalization conventions, the same SO-100 embodiment. It adds the piece deliberately left out here, a model that learns to predict actions from observations, using a vision-language backbone and the first incarnation of a generative robot policy.

## Further reading

A short, opinionated reading list for readers who want to dig deeper into the topics introduced above. References are grouped by section; ordering within each group is "start here" first.

Simulator and embodiment

- Tao et al. (2024), *ManiSkill3: GPU-Parallelized Robotics Simulation and Rendering*. arXiv:2410.00425. The paper behind the simulator we use.
- The Robot Studio (ongoing), *SO-ARM100 hardware design*. <https://github.com/TheRobotStudio/SO-ARM100>. The open-source CAD and BOM for the physical arm.
- Hugging Face LeRobot (2026), *SO-101 release notes*. Linked from the LeRobot docs. What changed from SO-100 and why.

LeRobot dataset format

- Cadene et al. (2024), *LeRobot: State-of-the-art ML for real-world robotics in PyTorch*. <https://github.com/huggingface/lerobot>. The framework's design document.
- Hugging Face Hub, *Datasets for robot learning*. <https://huggingface.co/datasets?other=lerobot>. The SO-100 and SO-101 pick-and-place datasets with episode-count and quality notes on the dataset cards.

Pick-and-place as a learning benchmark

- Florence et al. (CoRL 2021), *Implicit Behavioral Cloning*. arXiv:2109.00137. Introduces the PushT task and an early energy-based BC formulation. PushT is the canonical alternative to pick-and-place as a starter task; we chose pick-and-place because it shares an embodiment, action space, and observation pipeline with the eventual hardware deployment chapters.
- Zhao et al. (RSS 2023), *Action Chunking with Transformers* (ACT). arXiv:2304.13705. The `delta_timestamps` mechanism in §2.3 is the data-side enabler for ACT, which chapter 4 builds.
- Chi et al. (RSS 2023), *Diffusion Policy*. arXiv:2303.04137. The continuous-action approach in chapter 5 uses the same normalized DataLoader you built here.
