# Comprehensive Resource Library: Robot Learning & VLA

A compiled list of online courses, research papers, and technical resources relevant to the "Build a Large Robot Model (From Scratch)" project.

## I. Foundational Courses (Structure & Pedagogy)
*   **Stanford CS224R: Deep Reinforcement Learning (Chelsea Finn)**
    *   *Focus:* Frontiers of Robot Foundation Models, RL for LLMs, and Scaling Laws.
    *   *URL:* [cs224r.stanford.edu](https://cs224r.stanford.edu/)
*   **UC Berkeley CS285: Deep Reinforcement Learning (Sergey Levine)**
    *   *Focus:* Generative models for control, Diffusion Policies, and Probabilistic Robotics.
    *   *URL:* [rail.eecs.berkeley.edu/deeprlcourse](https://rail.eecs.berkeley.edu/deeprlcourse/)
*   **ETH Zurich: Robot Learning (Oier Mees, 2026)**
    *   *Focus:* Fundamentals to VLA, Embodied Reasoning, and World Models.
    *   *URL:* [cvg.ethz.ch/teaching/robot-learning/](https://cvg.ethz.ch/teaching/robot-learning/)
*   **CMU 16-831: Introduction to Robot Learning**
    *   *Focus:* Transporter Networks, RT-1, SayCan, and Sim2Real transfer.
    *   *URL:* [16-831-s24.github.io/lectures](https://16-831-s24.github.io/lectures)

## II. Primary Research Papers (The "Canon")
### 1. The VLA Architecture
*   **PaLM-E:** [PaLM-E: An Embodied Multimodal Language Model](https://palm-e.github.io/)
*   **RT-2:** [RT-2: Vision-Language-Action Models Transfer Web Knowledge to Robotic Control](https://arxiv.org/abs/2307.15818)
*   **OpenVLA:** [OpenVLA: An Open-Source Vision-Language-Action Model](https://openvla.github.io/)

### 2. Action Representations (Chapters 4 & 5)
*   **Diffusion Policy:** [Diffusion Policy: Visuomotor Policy Learning via Action Diffusion](https://diffusion-policy.cs.columbia.edu/)
*   **Flow Matching ($\pi_0$):** [Physical Intelligence - π0 Technical Report](https://www.physicalintelligence.company/blog/pi0)
*   **ACT (Action Chunking Transformer):** [Learning Fine Manual Manipulation with Low-Cost Hardware](https://tonyzhaoh.github.io/aloha/)

### 3. Reinforcement Learning & Alignment (Chapter 7)
*   **GRPO:** [DeepSeek-V3 Technical Report (Section on GRPO)](https://arxiv.org/abs/2412.19437)
*   **PPO:** [Proximal Policy Optimization Algorithms](https://arxiv.org/abs/1707.06347)

## III. Tools & Infrastructure
*   **Hugging Face LeRobot:** [github.com/huggingface/lerobot](https://github.com/huggingface/lerobot) - The dataset standard and training library used in the book.
*   **Open X-Embodiment:** [robotics-transformer-x.github.io](https://robotics-transformer-x.github.io/) - The massive multi-robot dataset.
*   **Gymnasium / Isaac Sim:** [gymnasium.farama.org](https://gymnasium.farama.org/) - Simulation environments for RL/IL.

## IV. Technical Blog Posts & Practical Guides
*   **Andrej Karpathy's Neural Networks Series:** [Zero to Hero](https://karpathy.ai/zero-to-hero.html) - Foundational for building Transformers from scratch.
*   **The Robot Data Diet (Stanford):** Discussion on the transition from teleop to data-driven transfer.
*   **Manning: RLHF Book (Nathan Lambert):** Crucial for understanding the transition from PPO to DPO in alignment.

## V. Arxiv References Provided by User
*   [1604.00289](https://arxiv.org/pdf/1604.00289): Learning from Demonstrations (Behavior Cloning foundations).
*   [2212.06817](https://arxiv.org/pdf/2212.06817): RT-1: Robotics Transformer for Real-World Control at Scale.
