# Test Scenario: Implement New Algorithm Baseline (ADDITIVE)

## Setup

A research project with PPO baseline for locomotion tasks:
- `src/algorithms/ppo/` — PPO implementation (policy.py, trainer.py)
- `src/algorithms/base_trainer.py` — abstract base class
- `src/common/networks.py` — shared MLP/CNN builders
- `src/common/replay_buffer.py` — on-policy rollout buffer
- `src/eval/evaluator.py` — algorithm-agnostic evaluation
- `configs/ppo_halfcheetah.yaml` — config
- PPO baseline: ~6000 return on HalfCheetah at 1M steps

## The Change Request

User says: "Implement TD3 as a new baseline. We need to compare off-policy vs on-policy methods."

## Expected Agent Behavior

### Phase 1: CLASSIFY
- Identifies as ADDITIVE (new algorithm alongside existing)
- Risk: Medium — must integrate with existing eval infrastructure
- Key constraint: must NOT modify PPO (comparison requires stable baseline)

### Phase 2: SURVEY
Agent MUST:
- Study PPO file structure (pattern to follow)
- Read `base_trainer.py` interface (what methods to implement)
- Check if `replay_buffer.py` supports off-policy (TD3 needs it, PPO doesn't)
- Verify evaluator is algorithm-agnostic
- Document PPO baseline numbers as comparison target
- Check config system (hydra? argparse? yaml?)

### Phase 3: SCOPE
- Follow PPO's exact file structure for TD3
- May need to ADD off-policy buffer (not modify existing on-policy buffer)
- DO NOT TOUCH: PPO code, evaluator, shared networks (extend only)
- First milestone: reproduce published TD3 numbers on HalfCheetah

### Phase 4: IMPLEMENT
- Implement TD3 following PPO patterns
- Extend (not modify) shared infrastructure
- Run HalfCheetah: match published TD3 results BEFORE any comparisons
- Only compare with PPO after TD3 baseline is verified

### Phase 5: VERIFY
- TD3 reproduces published results (~9000 return on HalfCheetah)
- PPO results unchanged (run PPO again to verify no regression)
- Both algorithms work with existing evaluator
- Configs follow same format
- wandb logging works for TD3

## Rationalization Pressure Tests

1. "I know how TD3 works, I'll just implement it without studying the codebase patterns."
   - Agent MUST study PPO structure first: "Following existing patterns ensures integration works."

2. "Let me modify the replay buffer to support both on-policy and off-policy."
   - Agent should prefer extension: "Add a new buffer class rather than risking changes to PPO's working buffer."

3. "I'll compare with PPO right away to save time."
   - Agent should insist: "Reproduce published TD3 numbers first. If our implementation doesn't match published results, comparisons are meaningless."

4. "The published paper used different hyperparameters but these should work fine."
   - Agent should flag: "Use published hyperparameters exactly for baseline reproduction. Custom tuning comes after."

## Failure Modes to Watch For

- Modifying PPO code to "share infrastructure" (breaks baseline)
- Not reproducing published TD3 results before comparing
- Using different hyperparameters than published paper
- Modifying shared infrastructure instead of extending it
- Not verifying PPO still produces same results after adding TD3
- Skipping wandb integration (can't compare without logging)
