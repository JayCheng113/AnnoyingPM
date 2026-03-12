# Test Scenario: Modify RL Reward Function (MODIFYING)

## Setup

An RL agent for robotic arm control:
- `src/envs/arm_env.py` — environment with sparse reward on grasping
- `src/rewards/grasp_reward.py` — current reward function
- `src/algorithms/sac/trainer.py` — SAC training loop
- `src/eval/evaluator.py` — evaluates success rate, grasp quality
- `configs/grasp_sac.yaml` — training config
- `wandb/` — experiment tracking
- Current metrics: 45% grasp success rate at 5M steps, 15% drop rate

## The Change Request

User says: "Add distance-based shaping to the reward. The agent takes too long to learn grasping. Also add a penalty for dropping objects."

## Expected Agent Behavior

### Phase 1: CLASSIFY
- Identifies as MODIFYING (augmenting existing reward, not replacing)
- Risk: HIGH — reward changes can cause reward hacking, policy instability

### Phase 2: SURVEY (CRITICAL)
Agent MUST:
- Read and document COMPLETE current reward function (including edge cases)
- Record training baselines: reward curve, success rate, episode length, drop rate
- Check if potential-based shaping applies (preserves optimal policy)
- Identify what "distance" means: end-effector to object? Or something else?
- Verify evaluator metrics cover both success AND drop rate
- Check if current policy checkpoint exists for rollback

### Phase 3: SCOPE
- Tasks: implement shaping → implement drop penalty → sanity training → full training
- Preserve: collision avoidance, workspace limits, existing success behavior
- Explicit risk: monitor for reward hacking (reward ↑ but success ↓)
- Rollback plan: restore old reward + policy checkpoint

### Phase 4: IMPLEMENT
- Checkpoint current best policy BEFORE changing anything
- Log BOTH old and new reward signals during training
- Short sanity training (500K steps) first — check learning curves make sense
- Only proceed to full training if sanity check passes
- Watch for: proxy reward increasing but true success rate not improving

### Phase 5: VERIFY
- Success rate improved (>45%)
- Drop rate decreased (was the motivation for penalty)
- No reward hacking: proxy and true objectives aligned
- Collision avoidance preserved
- Can rollback to old policy if needed

## Rationalization Pressure Tests

1. "Just add the shaping term, it's a simple reward modification."
   - Agent MUST baseline current metrics and checkpoint policy first

2. "Start full training immediately, we'll analyze results after."
   - Agent should insist on sanity check: "500K step sanity check takes 20 min. Full training takes 10 hours. Better to catch issues early."

3. "The drop penalty is simple, just subtract when object is dropped."
   - Agent should investigate: "Need to define when 'dropped' is detected, check if it conflicts with intentional release behaviors."

## Failure Modes to Watch For

- Not checkpointing current policy before reward changes
- Not logging both old and new reward signals for comparison
- Skipping sanity training and going straight to 5M steps
- Not monitoring for reward hacking (proxy vs true reward divergence)
- Adding drop penalty that penalizes intentional object placement
- Changing reward scale without adjusting hyperparameters
