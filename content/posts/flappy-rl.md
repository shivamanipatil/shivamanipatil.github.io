---
title: "Teaching an agent to play Flappy Bird using reinforcement learning"
date : "2026-10-06"
---

In this post, I will document a small reinforcement learning experiment where I trained an agent to play a Flappy Bird-like game. The main purpose is to record the experiments, failures, observations, and concepts so I can revisit and continue the project later.

### Environment
* The game was implemented using `pygame` and wrapped as a Gymnasium environment.
* The bird has:
  * Fixed horizontal position.
  * Variable vertical position.
  * Vertical velocity.
  * Radius of `18px`.
* Pipes have:
  * Horizontal position.
  * Gap center (`gap_y`).
  * Width of `70px`.
  * Gap size of `180px`.

Physics:

```python
gravity = 0.5
flap_strength = -8
pipe_speed = 4
```

Actions:

```text
0 -> do nothing
1 -> flap
```

The bird is approximately treated as a circle and the pipes as rectangular boundaries.

For a pipe:

```python
upper_edge = gap_y - pipe_gap / 2
lower_edge = gap_y + pipe_gap / 2
```

A collision happens when the bird horizontally overlaps the pipe and:

```text
bird_top <= upper_edge
```

or:

```text
bird_bottom >= lower_edge
```

### Observation space
The agent receives only 4 values:

```python
[
    bird_y,
    velocity_y,
    pipe_x - bird_x,
    gap_y - bird_y
]
```

These represent:

1. Bird vertical position.
2. Bird vertical velocity.
3. Horizontal distance to the pipe.
4. Vertical distance from the bird to the gap center.

The agent is not explicitly told:
* Bird radius.
* Pipe boundaries.
* Safe vertical range.
* When to flap.
* Whether it is too high or too low.

### PPO
I used PPO from Stable Baselines3:

```python
from stable_baselines3 import PPO
```

with:

```python
PPO("MlpPolicy", env)
```

The basic RL loop is:

```text
state
  ↓
policy
  ↓
action
  ↓
environment
  ↓
reward + next state
```

### Random-agent baseline
Before training, I tested a random-action agent.

```text
Average score: ~0
Best score:    0
Average reward: ~-5.9
```

Random flapping was essentially useless.

### Initial PPO with reward shaping
The first PPO version used shaped rewards.

Example:

```python
reward = 0.01

distance_from_gap = abs(bird_y - gap_y)

reward += 0.05 * (
    1.0 - min(distance_from_gap / 350.0, 1.0)
)
```

Passing a pipe:

```python
reward += 10
```

Dying:

```python
reward = -10
```

After around `100,000` training steps:

```text
Average score: ~5.8
Best score:    36
```

This worked, but the reward directly encoded useful behavior such as staying near the center of the gap.

I wanted to see how far the agent could get with much less domain knowledge.

### Sparse reward
The reward was changed to:

```text
pass pipe -> +1
die       -> -1
otherwise -> 0
```

There was no reward for:
* Staying alive.
* Staying near the center.
* Good velocity.
* Moving toward the opening.

This made the problem much harder because the positive reward was delayed.

### Sparse PPO failure
A sparse PPO agent trained for `100,000` steps gave roughly:

```text
Average score: 0.02
Best score:    1
```

I then trained using 8 parallel environments:

```python
env = make_vec_env(FlappyEnv, n_envs=8)
```

for:

```text
1,000,000 steps
```

Result:

```text
Average score: 0.03
Best score:    3
Average flaps: ~0.5
```

The policy had effectively learned to almost never flap.

More training alone did not solve the problem.

### Delayed reward problem
The first pipe was around `480px` ahead of the bird.

At:

```text
pipe_speed = 4px/frame
```

the bird needs to survive approximately:

```text
480 / 4 = 120 frames
```

before it can receive its first positive reward.

This makes exploration difficult because the agent needs to accidentally survive long enough before learning anything useful.

### Curriculum learning
Instead of modifying the reward, I made the environment easier initially.

The first curriculum environment used roughly:

```text
pipe gap      = 400px
pipe distance = 100px
```

The reward stayed sparse.

At `100px` distance:

```text
Average score: 1.06
Best score:    4
```

The pipe distance was then increased gradually:

```text
100
200
250
275
...
425
450
480
```

One interesting observation was that learning was not always gradual.

At distance `425`, performance stayed near zero for multiple training rounds and then suddenly reached:

```text
Average score: 5.64
Best score:    47
```

Sparse reward learning can therefore behave more like a sudden discovery than a smooth improvement curve.

Eventually the agent reached the original:

```text
pipe distance = 480
```

### Gap curriculum
After reaching the normal pipe distance, I gradually reduced the gap:

```text
400
380
360
...
200
180
```

Reward still remained:

```text
+1 pass
-1 death
0 otherwise
```

At:

```text
pipe distance = 480
pipe gap      = 180
```

the agent reached approximately:

```text
Average score: 2.37
Best score:    19
```

At this point the agent was playing the original environment using only sparse reward.

### More training is not always better
I trained the policy for another `500,000` steps on the normal environment.

Result:

```text
Average score: 2.92
Best score:    18
```

However, continuing for another `1,000,000` steps caused complete collapse:

```text
Average score: 0
Best score:    0
Average flaps: 0
```

The policy had again converged toward doing nothing.

An important observation from this experiment was:

> PPO performance is not necessarily monotonic. A later checkpoint can be significantly worse than an earlier one.

### Best-checkpoint training
Instead of assuming the last model was the best, I trained in rounds of:

```text
50,000 steps
```

After every round, the model was evaluated.

If the candidate was better than the previous best, it was saved.

This produced:

```text
Average score: 8.28
Best score:    45
```

However, the live model could still degrade after a good checkpoint because training continued from its current weights.

### Rollback training
The next strategy was:

```text
current champion
      ↓
train 50k steps
      ↓
evaluate candidate
      ↓
better? ---- yes ---> replace champion
   |
   no
   ↓
reload champion
```

So bad candidates were discarded completely.

This produced:

```text
Average score: 17.88
Median score:  11
Best score:    133
```

A later `5000` episode evaluation gave approximately:

```text
Average score: 17.34
Best score:    161
```

This confirmed that the improvement was not just evaluation noise.

### Failure analysis
Instead of continuing to train blindly, I started looking at how the agent was dying.

One earlier model showed:

```text
Upper pipe deaths: 96.30%
Lower pipe deaths: 3.64%
Floor deaths:      0.06%
```

For upper-pipe deaths:

```text
Average velocity:   +2.12
Average relative Y: -79.54px
Falling:            75.05%
Rising:              8.03%
```

Positive velocity means the bird was falling.

The bird was usually not actively flapping into the pipe. It was often already too high and started correcting too late.

### Pipe encounter analysis
I then analyzed every pipe encounter rather than only final deaths.

Over `5000` episodes:

```text
Pipe encounters: 92,736
Average score:   17.55
Best score:      181

Passes:          87,748
Failures:         4,988

Overall pipe pass rate: 94.62%
```

Pass rate by transition:

```text
FIRST PIPE             91.86%

SMALL CHANGE <= 75px   94.65%

MEDIUM UP 76-150px     96.40%
MEDIUM DOWN 76-150px   94.54%

LARGE UP > 150px       96.14%
LARGE DOWN > 150px     92.68%
```

This showed that large upward transitions were not the main problem.

The weakest normal transition was:

```text
LARGE DOWN > 150px
```

with:

```text
92.68% pass rate
```

The first pipe was also unusually difficult:

```text
91.86% pass rate
```

All classified first-pipe deaths were upper-pipe collisions.

### Entry position
The bird's position relative to the gap center when entering a pipe was analyzed.

```text
very high (< -60px)      70.66%
high (-60 to -30px)      96.66%
center (-30 to +30px)    95.26%
low (+30 to +60px)       93.39%
very low (> +60px)       78.46%
```

The policy was quite reliable when entering roughly within:

```text
-60px to +60px
```

of the gap center.

Outside this range the pass rate dropped significantly.

### Entry velocity
Pass rates grouped by entry velocity were:

```text
rising fast (< -4)       94.58%
rising (-4 to -1)        94.94%
nearly level             94.65%
falling (+1 to +4)       93.96%
falling fast (> +4)      94.82%
```

Velocity by itself was not strongly predictive of failure.

The weakness was more likely a combination of:

```text
pipe geometry
+
vertical position
+
available time to correct
```

### Weakness-focused training
Based on the analysis, I created a training environment that generated more of the situations the agent struggled with:

```text
difficult first pipes
large downward transitions
```

The reward was still unchanged:

```text
pass -> +1
death -> -1
otherwise -> 0
```

The training distribution was modified, not the reward.

The environment still generated normal random pipes as well, so the agent did not train only on the weak cases.

The important setup was:

```text
TRAIN:
weakness-focused environment

EVALUATE:
normal environment
```

Every candidate was evaluated on the original game. This prevented selecting a model that only performed well on the biased training environment.

### Weakness-focused result
The weakness-focused rollback training produced:

```text
Average score: 48.12
Median score:  34
Best score:    245
```

I then evaluated the model over `5000` normal games.

```text
Episodes:      5000
Average:       48.72
Median:        34
90th pct:      110
95th pct:      149
Best:          493
Score 0 rate:  2.12%
```

Compared with the previous champion:

```text
Average score:

~17.5 -> 48.72
```

This was close to a `3x` improvement in average score.

The median score of `34` is also important because the improvement was not only due to rare long runs.

The final distribution approximately means:

```text
50% of games score >= 34
10% of games score >= 110
5% of games score >= 149
```

The highest observed run was:

```text
493
```

### Progress summary

```text
Random agent
    avg ~0

Sparse PPO
    avg 0.03
    best 3

Curriculum on final environment
    avg 2.37
    best 19

Normal practice
    avg 2.92

Blind additional training
    avg 0
    policy collapsed

Best-checkpoint training
    avg 8.28
    best 45

Rollback training
    avg 17.88
    median 11
    best 133

5000 episode evaluation
    avg ~17.5
    best 181

Weakness-focused training
    avg 48.12
    median 34
    best 245

Final 5000 episode evaluation
    avg 48.72
    median 34
    p90 110
    p95 149
    best 493
```

### Concepts used
* **Sparse rewards**
  * Reward only on passing a pipe or dying.
  * Reduced encoded domain knowledge but made exploration harder.

* **Delayed rewards**
  * Initially the agent had to survive around 120 frames before receiving positive reward.

* **Exploration**
  * The agent needs to discover successful trajectories before PPO can reinforce them.

* **Curriculum learning**
  * The environment was gradually made harder while keeping the reward unchanged.

* **Policy degradation**
  * Additional PPO training sometimes destroyed previously useful behavior.

* **Checkpoint selection**
  * Intermediate models were evaluated instead of assuming the final checkpoint was best.

* **Rollback training**
  * Worse candidate models were discarded and training resumed from the current champion.

* **Failure analysis**
  * Pipe encounters and death states were measured instead of blindly adding more training.

* **Targeted experience distribution**
  * Situations the policy struggled with were generated more frequently during training.

### Current state
Current best model:

```text
flappy_weakness_best.zip
```

Normal environment:

```python
FlappyEnv(
    pipe_distance=480,
    pipe_gap=180
)
```

Current measured performance over `5000` episodes:

```text
Average:         48.72
Median:          34
90th percentile: 110
95th percentile: 149
Best:            493
Score 0 rate:    2.12%
```

The reward remains:

```text
+1 -> pass pipe
-1 -> death
 0 -> everything else
```

### Where to continue
The next experiment is to rerun the pipe-encounter analysis using:

```python
model = PPO.load("flappy_weakness_best")
```

and compare the original weak areas:

```text
FIRST PIPE:
91.86% -> ?

LARGE DOWN > 150px:
92.68% -> ?
```

This should show whether the targeted training directly improved the failure modes it was designed for.

Other possible experiments:
* Analyze combinations of position and velocity.
* Normalize observations.
* Tune PPO hyperparameters.
* Try a larger policy network.
* Add additional state while keeping rewards sparse.
* Compare PPO with another RL algorithm.
* Test generalization with different gravity, gap sizes, pipe speed, or bird physics.

The main practical lesson from this experiment was that simply increasing training time was not the most effective approach.

The largest improvement came from:

```text
observe failures
    ↓
identify weak situations
    ↓
train more frequently on those situations
    ↓
evaluate on the real environment
```

The agent went from barely discovering sparse rewards to averaging almost `49` pipes and occasionally reaching close to `500`.
