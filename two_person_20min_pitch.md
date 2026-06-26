# Two-Person 20-Minute Thesis Defense Pitch

Use this as speaker notes, not as text to memorize word-for-word. The goal is to sound confident and explanatory: describe why each slide matters, then move on.

Suggested split:

- Speaker 1: Mey Aya
- Speaker 2: Lyne
- Pattern: Mey, Lyne, Mey, Lyne, Mey, Lyne
- Total target time: 20 minutes

## 0. One-Minute Core Pitch

Our thesis addresses a scheduling problem in Earth Observation constellations. Modern EO missions are no longer only one satellite taking images from orbit. They are moving toward fleets of satellites that must decide, at each moment, which target to image, which satellite should do it, and how to avoid wasting power, memory, and visibility windows.

The difficulty is that the problem is dynamic and constrained. Satellites have limited battery, limited memory, short target visibility windows, clouds can invalidate an image, and urgent events such as disasters can appear after the original plan is made.

So our contribution is a physics-grounded multi-agent reinforcement learning framework. Each satellite is treated as an agent. We use STK-derived orbital geometry, a hard action mask that blocks infeasible choices, and two cooperative MARL algorithms: IPPO and MAPPO. We evaluate them against four classical baselines.

The main result is that cooperative MARL can learn feasible, reactive, and energy-aware scheduling policies. MAPPO gives the best overall balance: highest reward, 50 percent event response, zero executed slew-feasibility violations, and around 88.8 percent mean final battery, while still being small enough to run onboard.

## Section 1 - Problem And Motivation

Speaker: Mey Aya  
Slides: 1-5  
Target time: 2.5 minutes

### Slide 1 - Title

Good morning. We are Benoudjit Lyne and Chikhi Mey Aya, and today we present our engineering thesis: Physics-Grounded Multi-Agent Reinforcement Learning for Reactive and Power-Aware Earth Observation Constellation Scheduling.

In simple terms, we study how a fleet of Earth observation satellites can make imaging decisions autonomously, while respecting real operational constraints such as orbital visibility, battery, memory, clouds, and attitude maneuvers.

### Slide 2 - EO satellites and constellations

Earth Observation satellites image the surface of the Earth for applications such as climate monitoring, disasters, mapping, agriculture, and security.

Historically, missions often relied on one or a few large satellites. Now the trend is toward constellations: multiple satellites working together. The advantage is higher revisit frequency and better coverage, because if one satellite misses a target, another may pass over it later.

But this also changes the planning problem. We no longer schedule a single platform. We need coordinated decisions across a fleet.

### Slide 3 - What is scheduling?

Scheduling means answering four operational questions: which target should be imaged, which satellite should image it, when should it happen, and when should the data be downlinked.

This is hard because each decision interacts with the next. Imaging one cell uses energy and memory. Slewing to a target consumes time. A satellite may have a visibility window of only a few minutes. So the scheduler must balance mission value against feasibility.

### Slide 4 - Four core challenges

Our thesis focuses on four major challenges.

First, power. Imaging and maneuvers drain the battery, and the satellite only recharges when it is sunlit, not during eclipse.

Second, orbital and physical constraints. A target is only visible during short access windows, and the satellite cannot instantly point anywhere.

Third, decentralization. In a constellation, each satellite may only have local information. A central planner is not always practical at scale.

Fourth, reactivity. Clouds, disasters, and emergent events can invalidate an offline plan, so the system must react during execution.

### Slide 5 - Motivation

This is why the problem matters now. EO missions are shifting from single satellites toward constellations. From our ASAL field review, local mission planning is still strongly single-satellite oriented, while future operations will need multi-satellite coordination.

So the motivation of this work is both scientific and local: we want a scheduling framework that is built for the constellation shift, and that can support reactive, power-aware decision-making.

Transition to Lyne:

To explain how we approach this, Lyne will first introduce the learning concepts and the gap we found in the literature.

## Section 2 - Foundations And Research Gap

Speaker: Lyne  
Slides: 6-13  
Target time: 4 minutes

### Slide 6 - Reinforcement learning

Reinforcement learning is a framework where an agent observes the state of an environment, chooses an action, receives a reward, and improves its policy over time.

In our case, the agent is a satellite. The state includes information such as visible cells, battery, memory, clouds, and current pointing state. The action is either to stay idle or image one candidate cell. The reward encourages useful coverage, priority targets, and quick response to emergent events, while penalizing energy use and redundancy.

### Slide 7 - PPO

We use PPO as the policy optimization backbone. PPO is popular because it is stable and efficient for actor-critic training. Its clipped objective avoids overly large policy updates, which helps training remain controlled.

Every learning method in this thesis is based on PPO. The difference is how PPO is extended to multiple satellites.

### Slide 8 - Dec-POMDP formal model

Our problem is naturally a decentralized partially observable Markov decision process, or Dec-POMDP.

Decentralized means that several agents act at the same time. Partially observable means no satellite sees the full global state directly. Each satellite sees its own local candidates and own state, but the mission objective is shared by the whole constellation.

This is important because it matches real constellation scheduling better than a single global agent.

### Slide 9 - IPPO and MAPPO

We compare two multi-agent PPO methods.

IPPO means Independent PPO. Each satellite has its own local critic, so training is mostly decentralized. It is simple and deployment-friendly.

MAPPO uses centralized training and decentralized execution. During training, the critic can access a compressed global state, which helps it evaluate the effect of each satellite's action in the context of the full constellation. But during execution, the critic is discarded and each satellite only runs the actor locally.

So the key question is: does the centralized critic help coordination?

### Slide 10 - Landscape analysis

In the literature, different method families solve different parts of the problem.

Exact optimization can provide strong constraints and near-optimal schedules, but it scales poorly and is usually offline.

Metaheuristics scale better but remain mostly ground-centric and batch-based.

Single-agent RL can react quickly, but it does not naturally solve constellation-level coordination.

MARL is promising for decentralized coordination, but existing work often uses weak physical models or lacks hard feasibility guarantees.

### Slide 11 - Literature framework

To organize this, we built a three-axis taxonomy of 75 studies.

The axes are: control architecture, decision paradigm, and physical fidelity. This lets us position each work according to whether it is centralized or decentralized, optimization or learning-based, and abstract or physically realistic.

The taxonomy helped us identify where the literature is strong, and where it is missing something.

### Slide 12 - Research questions

The central question is: can cooperative, physics-grounded MARL schedule an EO constellation feasibly, reactively, and within power?

We split it into four research questions:

Can learned policies respect physical constraints? Can decentralized agents coordinate without a central assignment? Can the fleet react to clouds and emergent events? And does MARL, especially MAPPO, improve over classical heuristics?

### Slide 13 - Our position

This slide summarizes the gap and our contributions.

The missing cell is physics-grounded MARL for EO constellation scheduling. Our work contributes a taxonomy, a Dec-POMDP formulation, an STK-based physics-grounded environment, IPPO and MAPPO schedulers, and a reactive power-aware empirical comparison.

Transition to Mey:

Now Mey will explain how we turned this formulation into an actual scheduling environment.

## Section 3 - Proposed System And Formalization

Speaker: Mey Aya  
Slides: 14-18  
Target time: 4 minutes

### Slide 14 - Experimental setup

Our environment uses four ALSAT-type low Earth orbit satellites over a 31-day STK training window and an unseen 30-day validation window.

We discretize the Earth into a 36 by 72 grid, so each cell is 5 degrees by 5 degrees. The baselines are Random, Highest Priority, Priority times Coverage, and Centralized Greedy.

The point of using STK is that orbital geometry is not invented by the RL simulator. Access windows, eclipse, pointing angles, and contact information come from orbital simulation, then the learning environment uses those precomputed arrays efficiently.

### Slide 15 - State space

The global grid has four layers.

Base priority is the normal importance of a cell. Current priority can increase temporarily when an emergent event appears. Coverage value represents how useful it is to image a cell now, and decreases after the cell is imaged. Cloud cover reduces imaging value and can block actions when it is too high.

The orange cell represents an emergent event. When an event appears, its priority is elevated for a limited time, so the agents need to react quickly.

### Slide 16 - Observation and action space

Each satellite does not observe the entire global state. Instead, it receives up to 80 candidate cells, each described by 8 features, plus a 9-dimensional own-state vector. This gives an observation dimension of 649.

The action space is fixed: action 0 is idle, and actions 1 to 80 correspond to imaging one candidate cell. A fixed action size is useful for neural training, even though the number of visible cells changes over time.

### Slide 17 - Reward function

The reward is shared across all satellites. It combines several objectives:

Coverage reward encourages imaging useful cells. Priority reward encourages important targets. Redundancy penalty discourages repeated imaging of cells that were already covered. Energy penalty discourages excessive power use. Idle penalty prevents the agents from being too passive. Emergent event bonus heavily rewards fast response to urgent targets.

This is important because coordination is not manually assigned. It emerges because all satellites optimize the same shared mission reward.

### Slide 18 - System overview

The full system has four layers.

First, STK precomputes orbital geometry: footprints, eclipse intervals, contact windows, and pointing information.

Second, the Dec-POMDP environment builds local observations, applies the action mask, updates battery and memory, and computes reward.

Third, the policy layer uses a shared actor, with either IPPO local critics or a MAPPO centralized critic.

Fourth, PPO training collects rollouts, computes advantages, updates the networks, and selects checkpoints.

Transition to Lyne:

Now that the environment is defined, Lyne will walk through how we trained and evaluated the agents.

## Section 4 - Training And Qualitative Behavior

Speaker: Lyne  
Slides: 19-24  
Target time: 3.5 minutes

### Slide 19 - Experimental setup

Each episode has 200 steps, with a 30-second timestep, so it represents around 100 minutes, close to one orbital period.

Both IPPO and MAPPO were trained for 5 million agent-steps, or 6,250 episodes. Each episode randomizes the start offset, priority map, cloud field, and emergent events. On average, about two emergent events appear per episode.

This randomness is important because we do not want a policy that only memorizes one orbit pattern.

### Slide 20 - IPPO results

IPPO learns a solid decentralized policy. On validation, it reaches 27.09 percent coverage, 29.06 percent priority-weighted coverage, around 568 images per episode, and a 47.7 percent event response rate.

This is meaningful because IPPO uses local critics. Even with less global training information, the combination of shared reward and physics-grounded mask lets the satellites learn useful behavior.

### Slide 21 - MAPPO results

MAPPO improves faster early in training. The centralized critic helps the agents understand the effect of their actions at constellation level.

On validation, MAPPO reaches 27.50 percent coverage, 29.50 percent priority-weighted coverage, mean reward 597.33, around 577 images per episode, and a 50 percent event response rate.

The improvement over IPPO is not huge in raw coverage, but it is consistent in reward, imaging activity, and event response.

### Slide 22 - Validation

Validation is done on an unseen 30-day window. The performance drops moderately, but not catastrophically.

That means the agents are not simply memorizing the training window. They generalize reasonably to different access patterns, clouds, and event timings.

### Slide 23 - Energy and constraints

Both learned policies preserve energy well, with a mean final battery of about 88.8 percent.

There is a caveat: some tail cases still stress one satellite, with a minimum final battery case around 5.5 percent. So the aggregate behavior is power-aware, but future work should improve worst-case energy robustness.

For constraints, the key result is that executed actions were feasible. The action mask blocks actions that violate geometry, cloud, eclipse, battery, memory, camera-cycle, or slew feasibility.

### Slide 24 - Emergent division of labor

The heatmaps show that the satellites develop a division of labor without a central assignment step.

This is exactly what we want from cooperative MARL: the agents learn to distribute imaging effort through the shared reward and coverage decay. The coordination is not perfect, but it is visible in the behavior.

Transition to Mey:

Next, Mey will compare the learned policies against classical baselines and show whether this approach is practical for deployment.

## Section 5 - Comparison, Reactivity, And Deployability

Speaker: Mey Aya  
Slides: 25-28  
Target time: 3.5 minutes

### Slide 25 - Learned policies vs baselines

This table compares all methods on the same validation setting.

Random performs poorly, with only 12 percent coverage and almost no event response. Highest Priority and Centralized Greedy image frequently, but they repeat the same valuable cells too often, so redundancy is around 47 percent.

Priority times Coverage is the strongest heuristic. It reaches slightly higher raw coverage than MAPPO: 27.56 percent versus 27.50 percent.

But MAPPO wins the overall mission objective: it has the highest reward, 597.33, and stronger event response: 1.10 events per episode versus 0.75 for Priority times Coverage.

So the message is: the best heuristic is competitive on static coverage, but MAPPO is better when we include reactivity and the full reward.

### Slide 26 - Where MAPPO pulls ahead

MAPPO's advantage is mainly in event response and imaging activity, not in redundancy reduction.

Compared with IPPO, it responds to 50 percent of emergent events instead of 47.7 percent. Compared with the best heuristic, it also reacts better to urgent targets.

Both IPPO and MAPPO recorded zero executed slew-feasibility violations, and both preserve battery on average. So the central critic helps mostly with choosing more useful actions under dynamic conditions.

### Slide 27 - Deterministic deployment

During training, policies sample actions for exploration. But at deployment, we should choose the best action deterministically.

This slide shows why. For MAPPO, stochastic evaluation gave 20.1 percent coverage and 18.2 percent event response, while greedy deployment reached 27.5 percent coverage and 50 percent event response.

Emergent events have narrow response windows. Random exploration wastes time. In operations, the policy should commit to the highest-scoring feasible action.

### Slide 28 - Deployability

A practical question is: can this run onboard?

At execution, each satellite only needs the actor. The centralized critic used by MAPPO during training is discarded.

Our actor has about 108 thousand parameters, around 0.43 MB in fp32 or 0.11 MB if quantized to int8. It needs about 1.1 MFLOPs per decision and around 0.09 milliseconds on the development CPU.

Since decisions happen every 30 seconds, there is a very large timing margin. The policy is lightweight enough for onboard execution compared with embedded platforms and previous AI-in-orbit demonstrations.

Transition to Lyne:

Finally, Lyne will close with what we delivered, what remains limited, and the future directions.

## Section 6 - Contributions, Limits, Future Work, Conclusion

Speaker: Lyne  
Slides: 29-32  
Target time: 2.5 minutes

### Slide 29 - Contributions

To summarize, we delivered five main contributions.

First, a three-axis taxonomy mapping 75 studies. Second, a formal Dec-POMDP model for EO constellation scheduling. Third, a physics-grounded environment using STK geometry and a hard action mask. Fourth, IPPO and MAPPO cooperative schedulers under centralized training and decentralized execution. Fifth, an empirical comparison against four baselines, showing that MAPPO gives the best overall balance.

### Slide 30 - Limitations

There are also clear limitations.

Coverage plateaus around 27 to 31 percent in our current scenario, and the 80 percent milestone was never reached. Scale is still untested because we used four satellites, not dozens or hundreds.

Also, the system is physics-grounded through STK geometry and action masks, but not yet fully physics-informed in the sense of differentiable physical equations inside the learning objective.

Finally, some diagnostics are missing, such as detailed memory use, downlink counts, camera-cycle saturation, and per-constraint binding.

### Slide 31 - Future work

The next steps follow directly from those limits.

We should scale to larger and heterogeneous constellations, introduce stronger coordination or communication, improve worst-case energy robustness, add richer diagnostics, and move from mask-based grounding toward differentiable physics-informed layers.

The long-term goal is hardware-in-the-loop and eventually onboard flight validation.

### Slide 32 - Conclusion

The final conclusion is that cooperative, physics-grounded MARL can schedule an EO constellation in a way that is feasible, reactive, and energy-aware.

MAPPO achieves the best overall balance: it beats the heuristic baselines on reward and reactivity, responds to 50 percent of emergent events, preserves around 88.8 percent mean final battery, and runs with zero executed feasibility violations.

This does not solve full operational constellation autonomy yet. But it provides a validated foundation for decentralized, physics-aware mission planning for future Earth Observation constellations.

Thank you for your attention. We are ready for your questions.

## Timing Cheat Sheet

- Section 1, slides 1-5: 2.5 minutes
- Section 2, slides 6-13: 4 minutes
- Section 3, slides 14-18: 4 minutes
- Section 4, slides 19-24: 3.5 minutes
- Section 5, slides 25-28: 3.5 minutes
- Section 6, slides 29-32: 2.5 minutes
- Total: 20 minutes

## Quick Defense Lines For Questions

Why MAPPO if the coverage improvement is small?

MAPPO's advantage is not only raw coverage. The strongest heuristic slightly matches raw coverage, but MAPPO achieves the highest reward and strongest emergent-event response. Since our objective is reactive, priority-aware, and constrained scheduling, reward and event response are central.

Why use action masking?

Because in satellite scheduling, some actions are physically impossible. Penalizing impossible actions after selection is not enough for operational feasibility. The mask removes infeasible choices before the policy acts.

Why call it physics-grounded instead of fully physics-informed?

Because the physics enters through STK-derived geometry, resource dynamics, and hard feasibility masks. We do not yet embed differentiable physical residuals directly into the learning loss, so fully physics-informed MARL is future work.

Why only four satellites?

Four satellites let us validate the full pipeline: STK geometry, Dec-POMDP environment, action mask, reward, IPPO/MAPPO, and baselines. Scaling to tens or hundreds is one of the main future directions.

What is the practical takeaway?

A lightweight decentralized actor can make feasible onboard decisions every 30 seconds, while the centralized critic is used only during training. This makes the method compatible with the CTDE idea: train with more information, deploy with local autonomy.
