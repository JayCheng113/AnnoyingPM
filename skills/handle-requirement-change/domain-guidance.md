# Domain-Specific Guidance

Read the section relevant to your current change. Don't load the whole file.

## LLM / Prompt Engineering

**Unique risks:** Non-deterministic output, cascading failures through prompt chains, silent quality degradation.

**Survey essentials:**
- Document current prompt version + evaluation scores as baseline
- Map the full prompt chain: system prompt → user prompt → tool calls → output parsing
- Identify ALL consumers of LLM output (code that parses `response.field` will break on format changes)
- Check if prompt changes affect other prompts in a chain

**Implementation rules:**
- ONE prompt change per task, evaluate between changes
- Model switching (GPT-4 → Claude): run SAME eval suite on BOTH models BEFORE migrating
- Output format changes: update ALL parsers BEFORE changing the prompt
- New tools/functions: verify model actually calls them correctly with test cases

**Verification:**
- Eval suite comparison: new metrics vs Phase 2 baselines
- Edge cases: adversarial, long, multilingual inputs
- Guardrails still effective after changes
- Regressions in UNMODIFIED prompt behaviors

**Top failure mode:** Changing system prompt, assuming downstream parsing still works. Non-deterministic output means "worked in testing" proves nothing without systematic evaluation.

---

## RL / Training

**Unique risks:** Reward hacking, policy collapse from mid-training changes, silent degradation.

**Survey essentials:**
- Record training metrics: reward curves, episode lengths, success rates
- Document reward function completely — including shaping terms and rationale
- Check env-agent interface: observation space, action space, termination conditions
- For reward changes: understand potential-based shaping guarantees (or lack)

**Implementation rules:**
- Reward changes: NEVER modify during active training without checkpointing current policy
- Environment changes: verify obs/action space compatibility with existing policy
- Curriculum changes: smooth transitions between stages (no sudden difficulty spikes)
- Version old reward/env configs — you may need rollback
- Short sanity training (e.g., 10% of full budget) before committing to full runs

**Verification:**
- Learning curves: new vs baseline at same step counts
- Reward hacking check: proxy reward ↑ but true objective ↓ = ALARM
- Qualitative check: watch rollouts, don't just read numbers
- Test in held-out environment configurations

**Top failure mode:** Modifying reward mid-training, interpreting increased proxy reward as improvement, when agent is exploiting a flaw.

---

## ML Pipeline / Training

**Unique risks:** Data-model version mismatch, silent schema drift, train-serve skew.

**Survey essentials:**
- Document current model performance (ALL metrics, not just primary)
- Map full data flow: raw → preprocessing → features → model → output → eval
- Check feature store consistency between training and serving
- For architecture changes: verify tensor shapes at each layer

**Implementation rules:**
- Loss function changes: keep old loss as monitoring metric even if not optimizing
- Architecture changes: verify gradient flow (no vanishing/exploding)
- Schema changes: update ALL downstream pipeline stages
- New data sources: verify distribution compatibility
- Abbreviated training (fewer epochs) as sanity check before full run

**Verification:**
- Compare ALL metrics vs baselines (not just the one you optimized)
- Check for data leakage in new features
- Serving pipeline handles new model correctly (preprocessing + postprocessing)
- Edge-case data: missing values, outliers, adversarial examples

**Top failure mode:** Changing feature engineering without updating serving pipeline → train-serve skew that only appears in production.

---

## AI Agent Development

**Unique risks:** Tool interaction side-effects, context window overflow, coordination breakdowns.

**Survey essentials:**
- Map tool dependencies: which tools call which, what state they share
- Document current agent behavior on standard test scenarios
- Context window budget: will this change push usage over limits?
- Multi-agent: map coordination protocols and message formats

**Implementation rules:**
- New tools: test invocation in isolation BEFORE agent loop integration
- Orchestration changes: verify all existing tools still called correctly
- Memory/context changes: test with long conversations that stress limits
- Guardrail changes: red-team after EVERY modification
- Keep behavior logs for before/after comparison

**Verification:**
- Standard scenarios: compare with Phase 2 baselines
- Multi-turn conversations (not just single-turn)
- Tool calling accuracy on edge cases
- Guardrail bypass attempts

**Top failure mode:** New tool competes with existing tools for invocation → wrong tool called in ambiguous situations.

---

## Framework Migration / New Baseline

**Unique risks:** API surface mismatch, implicit default differences, performance regression.

**Survey essentials:**
- Map COMPLETE API surface used from old framework (not just main functions)
- Identify implicit defaults (PyTorch dtype vs TensorFlow, data loading order)
- Document performance benchmarks: speed, memory, accuracy
- Check community migration guides

**Implementation rules:**
- Migrate one component at a time, verify each
- New baselines: simplest version first, match published results BEFORE modifications
- Keep old code running in parallel until new code verified
- Watch: random seeds, default hyperparameters, data loading order

**Verification:**
- Reproduce published numbers BEFORE custom modifications
- Performance: new vs old on identical inputs
- Numerical precision: FP differences between frameworks accumulate
- Training reproducibility (same seed → same results)

**Top failure mode:** Assuming framework defaults are equivalent. PyTorch and TensorFlow differ in initialization, loading order, precision — "invisible" differences cause hard-to-debug discrepancies.
