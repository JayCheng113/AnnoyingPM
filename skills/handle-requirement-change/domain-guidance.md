# Domain-Specific Guidance

Read ONLY the section relevant to your current change. Don't load the whole file.

---

## LLM / Prompt Engineering

**Survey:** Document current prompt version + eval scores as baseline. Map full prompt chain (system → user → tools → parsing). Identify ALL consumers of LLM output — code doing `response.field` breaks on format changes.

**Implement:** ONE prompt change per task, evaluate between. Model switching: run SAME eval on BOTH models BEFORE migrating. Format changes: update ALL parsers BEFORE changing prompt.

**Verify:** Eval suite vs baselines. Edge cases: adversarial, long, multilingual. Guardrails still effective. Unmodified behaviors haven't regressed.

### Concrete Commands
```bash
# Document baseline before prompt change
python eval.py --prompt-version current --output baselines/pre-change.json

# After change: evaluate with sufficient samples (min 20)
python eval.py --prompt-version new --samples 50 --output baselines/post-change.json

# Compare
python eval.py --compare baselines/pre-change.json baselines/post-change.json
```

### Common Failed Attempts

| Attempt | Why It Fails | Do This Instead |
|---------|-------------|-----------------|
| Change prompt + parser in same commit | Can't isolate which caused regression | One change per commit, eval between |
| Test with 1-2 examples | Non-deterministic: n=2 proves nothing | Min 20 samples across input categories |
| Copy prompt from GPT-4 to Claude verbatim | Models respond differently to identical prompts | Re-evaluate and tune per model |
| Add tool without testing invocation | Model may never call it, or call it incorrectly | Test tool calling in isolation first |
| Change output format without updating parsers | Downstream code breaks on first LLM call | Update parsers BEFORE changing prompt |

---

## RL / Training

**Survey:** Record training metrics (reward curves, success rates, episode lengths). Document reward function completely — including shaping terms. Check env-agent interface (obs/action space, termination). Checkpoint current best policy.

**Implement:** NEVER modify reward during active training without checkpointing. Smooth curriculum transitions. Version old configs. Short sanity training (10% budget) before full runs.

**Verify:** Learning curves: new vs baseline at same steps. Reward hacking check: proxy ↑ but true objective ↓ = ALARM. Watch rollouts qualitatively.

### Concrete Commands
```bash
# Before reward change: checkpoint
python train.py --save-checkpoint checkpoints/pre-reward-change

# Sanity check: 10% of full training budget
python train.py --max-steps 100000 --eval-freq 10000 --log-reward-components \
    --wandb-tag sanity-check

# Compare with baseline
python eval.py --checkpoint checkpoints/pre-reward-change --episodes 100
python eval.py --checkpoint checkpoints/latest --episodes 100
python compare_runs.py --baseline pre-reward-change --current latest

# Check for reward hacking: proxy vs true reward
python plot_rewards.py --metrics proxy_reward,true_objective --run latest
```

### Common Failed Attempts

| Attempt | Why It Fails | Do This Instead |
|---------|-------------|-----------------|
| Modify reward mid-training without checkpoint | Policy collapses, no rollback | Always checkpoint before changes |
| Skip sanity check, go straight to full training | Wastes 10h if reward is broken | 10% budget sanity check = 1h, catches issues |
| Interpret proxy reward increase as success | Agent may be reward hacking | Always plot proxy vs true objective together |
| Change obs space without updating policy input | Dimension mismatch crash or silent misalignment | Verify policy input layer matches new obs space |
| Add reward shaping that violates potential-based conditions | Optimal policy changes, comparison invalid | Use Φ(s)-based shaping to preserve optimal policy |

---

## ML Pipeline / Training

**Survey:** Document ALL model metrics (not just primary). Map full data flow: raw → preprocess → features → model → output → eval. Check train-serve consistency. For architecture changes: verify tensor shapes.

**Implement:** Loss changes: keep old loss as monitoring metric. Schema changes: update ALL downstream stages. New data: verify distribution compatibility. Abbreviated training before full run.

**Verify:** Compare ALL metrics vs baselines. Check data leakage. Verify serving pipeline handles new model. Test edge-case data.

### Concrete Commands
```bash
# Document baseline metrics
python eval.py --model models/current --all-metrics --output baselines/metrics.json

# After feature change: verify train-serve consistency
python features.py --mode train --input sample_data.csv --output features_train.json
python features.py --mode serve --input sample_data.csv --output features_serve.json
diff features_train.json features_serve.json  # must be identical

# Abbreviated training sanity check
python train.py --epochs 5 --eval-every-epoch --compare-baseline baselines/metrics.json

# Check for data leakage
python check_leakage.py --features new_features --target target_column
```

### Common Failed Attempts

| Attempt | Why It Fails | Do This Instead |
|---------|-------------|-----------------|
| Change feature engineering, don't update serving | Train-serve skew, only shows in production | Verify features match in both modes |
| Only check primary metric after change | Secondary metrics may regress badly | Compare ALL metrics vs baseline |
| Add data source without checking distribution | Model trains on biased/incompatible data | Plot distribution comparison before adding |
| Change architecture without gradient check | Vanishing/exploding gradients | Run `torch.autograd.detect_anomaly()` on first batch |
| Full training without abbreviated run | Waste compute on broken config | 5-epoch sanity check catches 90% of issues |

---

## AI Agent Development

**Survey:** Map tool dependencies and shared state. Document agent behavior on standard scenarios. Check context window budget. Multi-agent: map coordination protocols.

**Implement:** New tools: test in isolation BEFORE agent loop. Orchestration changes: verify existing tools still called correctly. Guardrail changes: red-team after EVERY modification.

**Verify:** Standard scenarios vs baselines. Multi-turn conversations. Tool accuracy on edge cases. Guardrail bypass attempts.

### Concrete Commands
```bash
# Before changes: capture baseline behavior
python run_scenarios.py --suite standard --output baselines/agent-behavior.json

# Test new tool in isolation
python test_tool.py --tool new_tool --cases tool_test_cases.json

# After changes: compare behavior
python run_scenarios.py --suite standard --output current/agent-behavior.json
python compare_behavior.py --baseline baselines/ --current current/

# Red-team guardrails
python red_team.py --suite guardrail_bypasses --model current
```

### Common Failed Attempts

| Attempt | Why It Fails | Do This Instead |
|---------|-------------|-----------------|
| Add tool without isolation testing | Tool invocation may fail or return unexpected format | Test tool independently with edge cases first |
| Add tool that overlaps existing tool | Agent calls wrong tool in ambiguous situations | Check for semantic overlap, add disambiguation |
| Change context management without stress test | Works in short convos, breaks in long ones | Test with 20+ turn conversations |
| Update guardrails without red-teaming | New guardrails may have bypass vectors | Run known bypass techniques after every change |

---

## Framework Migration / New Baseline

**Survey:** Map COMPLETE API surface from old framework. Identify implicit defaults (dtype, init, loading order). Document benchmarks: speed, memory, accuracy. Check community migration guides.

**Implement:** One component at a time. New baselines: match published results BEFORE modifications. Keep old code running in parallel. Watch: seeds, defaults, data order.

**Verify:** Reproduce published numbers first. Compare new vs old on identical inputs. Check numerical precision. Training reproducibility (same seed → same results).

### Concrete Commands
```bash
# Map API surface used from old framework
grep -r "old_framework\." src/ | awk -F'.' '{print $2}' | sort -u > api_surface.txt

# Reproduce published baseline
python train.py --config configs/published_baseline.yaml --seed 42
# Compare with published: return should be within 5%

# Compare frameworks on identical input
python compare_frameworks.py --input test_batch.pt --old old_model --new new_model \
    --tolerance 1e-5

# Check reproducibility
python train.py --seed 42 --run 1 && python train.py --seed 42 --run 2
python compare_runs.py --run1 results/run1 --run2 results/run2  # must match
```

### Common Failed Attempts

| Attempt | Why It Fails | Do This Instead |
|---------|-------------|-----------------|
| Assume framework defaults are equivalent | PyTorch/TF differ in init, dtype, loading order | Map and explicitly set ALL defaults |
| Skip reproducing published numbers | Your implementation may have bugs you don't see | Match published results before ANY modifications |
| Delete old framework code during migration | No rollback, can't compare side-by-side | Keep old code until new code is verified |
| Use different hyperparameters than published | Can't tell if differences are code bugs or HP effects | Use EXACT published HPs for baseline reproduction |
| Migrate all components at once | Can't isolate which component broke | One component per migration step |
