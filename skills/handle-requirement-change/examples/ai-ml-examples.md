# AI/ML Domain Examples

## Example 1: MODIFYING — Changing LLM System Prompt in RAG Pipeline

### Scenario

A RAG-based Q&A app uses Claude to answer questions from company docs. The PM requests: "Make the responses shorter and add source citations in a structured format."

**Current system prompt:**
```
You are a helpful assistant. Answer questions based on the provided context. If the context doesn't contain the answer, say so.
```

**Current output:** Free-form text paragraphs.
**Desired output:** Concise answer + structured JSON citations.

### Step 1: CLASSIFY
- **Type:** MODIFYING (changing existing LLM behavior)
- **Risk:** HIGH — output format change breaks all consumers; non-deterministic component
- **Business context:** Users want quicker answers with verifiable sources

### Step 2: SURVEY

**Investigation:**
```bash
# Find all code that consumes LLM output
Grep: "response" / "completion" / "answer" in src/**/*.{ts,py}
# Result:
#   src/api/chat.py (streams response to frontend)
#   src/parsers/answer_parser.py (extracts answer text)
#   src/analytics/usage_tracker.py (logs response length)
#   frontend/src/components/Answer.tsx (renders markdown)

# Find evaluation setup
Glob: **/eval/**
# Result: eval/test_cases.json (50 test Q&A pairs with expected answers)

# Find prompt files
Glob: **/*prompt*
# Result: src/prompts/system.txt, src/prompts/user_template.txt
```

**Impact Report:**
- `src/parsers/answer_parser.py` — assumes plain text output, will BREAK with structured citations
- `frontend/src/components/Answer.tsx` — renders markdown, needs to handle citation blocks
- `eval/test_cases.json` — expected answers are paragraph-format, need updating
- `src/analytics/usage_tracker.py` — response length tracking still works but metrics change significantly

**Current behavior baseline:**
- Average response: 150-200 words, paragraph format
- Evaluation score: 0.82 accuracy on 50 test cases
- No structured citations (just inline "according to...")

### Step 3: SCOPE

**Tasks:**
1. Update `src/prompts/system.txt` — add citation format instructions — S
2. Update `src/parsers/answer_parser.py` — parse structured citations from output — M
3. Update `frontend/src/components/Answer.tsx` — render citation blocks — M
4. Update `eval/test_cases.json` — adjust expected format — M
5. Run evaluation — compare accuracy and format compliance — S

**Non-Goals:**
- NOT changing the RAG retrieval pipeline
- NOT upgrading the embedding model
- NOT adding multi-language support

**Behaviors to preserve:**
- "I don't know" responses when context lacks answer
- Context-grounded answers (no hallucination)
- Streaming works correctly

**DO NOT TOUCH:**
- RAG retrieval pipeline (embedding, search, reranking)
- User prompt template
- Authentication/rate limiting

### Step 4: IMPLEMENT

**Key rule: update parsers BEFORE changing prompt.**
1. First: make parser handle BOTH old and new format (backwards compatible)
2. Then: change the prompt
3. Then: verify output matches new format across 50+ samples (not just 1)
4. Then: update frontend rendering
5. Then: update evaluation test cases

### Step 5: VERIFY
- [x] Evaluation: accuracy ≥ 0.80 (baseline 0.82) — within acceptable range
- [x] Citation format compliance: 95%+ responses include structured citations
- [x] "I don't know" behavior preserved
- [x] Streaming still works
- [x] 50 test cases pass with updated expected format
- [x] Response length reduced: avg 80-100 words (from 150-200)
- [x] RAG pipeline untouched (retrieval metrics identical)

### LEARN
- **Surprise:** analytics `usage_tracker.py` also consumed response length — metrics dashboard showed anomaly after change.
- **Failed attempt:** changed prompt before updating parser, broke 3 API calls before catching it.
- **Would-do-differently:** always update parsers BEFORE changing prompts.

---

## Example 2: REPLACING — Switching RL Reward Function

### Scenario

An RL agent navigates a warehouse. Current reward: sparse (only on task completion). Research lead says: "Switch to a dense reward with potential-based shaping. The agent takes too long to learn."

### Step 1: CLASSIFY
- **Type:** REPLACING (replacing reward function entirely)
- **Risk:** HIGHEST — reward changes can cause policy collapse or reward hacking

### Step 2: SURVEY

**Old reward inventory:**
```python
# Current (sparse)
def reward(state, action, next_state):
    if next_state.goal_reached:
        return 100.0
    if next_state.collision:
        return -10.0
    return -0.1  # step penalty
```

**Implicit behaviors from this reward:**
1. Agent learns to avoid walls (collision penalty)
2. Agent learns shortest paths (step penalty)
3. No reward hacking possible (sparse + simple)
4. Training converges after ~2M steps
5. Success rate: 60% at convergence

**Current baselines documented:**
- Convergence: 2M steps
- Success rate: 60%
- Average episode length: 120 steps
- Collision rate: 5%

### Step 3: SCOPE

**Tasks:**
1. Implement potential function Φ(s) based on distance-to-goal — M
2. Implement dense reward: R'(s,a,s') = R(s,a,s') + γΦ(s') - Φ(s) — S
3. Add reward component logging (sparse vs shaping terms) — S
4. Run short sanity training (100K steps) — compare with baseline learning curve — M
5. Run full training — compare with baseline — L

**Non-Goals:**
- NOT changing the policy architecture
- NOT modifying the environment dynamics
- NOT tuning hyperparameters beyond reward function

**Behaviors to preserve:**
- Collision avoidance
- Goal-seeking behavior
- No reward hacking (monitor proxy vs true reward separately!)

**DO NOT TOUCH:**
- Environment dynamics
- Observation space
- Action space
- Policy network architecture

### Step 4: IMPLEMENT

**Key rule: checkpoint current policy BEFORE any reward changes.**
1. Save current best policy as rollback target
2. Implement new reward alongside old (both compute, only one used)
3. Log BOTH reward signals during training
4. Sanity check at 100K steps: is learning curve reasonable?
5. Full training only after sanity check passes
6. Watch for: proxy reward ↑ but success rate ↓ (reward hacking signal!)

### Step 5: VERIFY
- [x] Convergence faster: 800K steps (from 2M) — 2.5x improvement
- [x] Success rate: 75% (from 60%) — improved
- [x] No reward hacking: true objective tracks proxy reward
- [x] Collision rate: 4% (from 5%) — preserved/improved
- [x] Average episode length: 80 steps (from 120) — improved
- [x] Old sparse reward still logged for comparison

### LEARN
- **Surprise:** step penalty (-0.1) interacted with distance shaping in unexpected ways — agent briefly learned to hover near goal without reaching it.
- **Would-do-differently:** analyze interaction between existing reward terms and new shaping before training.

---

## Example 3: ADDITIVE — Implementing a New Baseline Algorithm

### Scenario

Research project has PPO baseline for continuous control. Lead says: "Implement SAC as a new baseline for comparison."

### Step 1: CLASSIFY
- **Type:** ADDITIVE (new algorithm alongside existing)
- **Risk:** Medium — new code, but must integrate with existing evaluation infrastructure

### Step 2: SURVEY

**Investigation:**
```bash
# Understand existing algorithm structure
Glob: src/algorithms/**
# Result: src/algorithms/ppo/ (policy.py, trainer.py, config.yaml)

# Find shared infrastructure
Grep: "class.*Trainer\|class.*Policy\|class.*Buffer" in src/**/*.py
# Result:
#   src/algorithms/base_trainer.py (abstract base class)
#   src/algorithms/ppo/trainer.py (PPO implementation)
#   src/common/replay_buffer.py (on-policy buffer)
#   src/common/networks.py (shared MLP/CNN builders)

# Find evaluation pipeline
Glob: src/eval/**
# Result: src/eval/evaluator.py (runs policy, records metrics)

# Check config system
Glob: configs/**
# Result: configs/ppo_halfcheetah.yaml, configs/ppo_walker.yaml
```

**Existing patterns:**
- All algorithms inherit from `BaseTrainer`
- Configs in YAML with hydra-style overrides
- Shared network builders in `src/common/networks.py`
- Evaluation via `src/eval/evaluator.py` (algorithm-agnostic)
- Results logged to wandb

**PPO baseline metrics (on HalfCheetah):**
- Final return: ~6000 (1M steps)
- Training time: ~2 hours

### Step 3: SCOPE

**Tasks:**
1. Create `src/algorithms/sac/` following PPO structure — M
2. Add off-policy replay buffer to `src/common/replay_buffer.py` (or new file) — M
3. Implement SAC policy (actor-critic with entropy) — L
4. Implement SAC trainer inheriting `BaseTrainer` — L
5. Add configs: `configs/sac_halfcheetah.yaml`, `configs/sac_walker.yaml` — S
6. Run HalfCheetah: verify reproduction of published SAC results — M
7. Add to evaluation comparison scripts — S

**Non-Goals:**
- NOT modifying PPO implementation
- NOT adding new environments
- NOT tuning SAC beyond reproducing published results

**DO NOT TOUCH:**
- PPO implementation (don't modify existing algorithm)
- Evaluation pipeline (SAC should work with existing evaluator)
- Shared network builders (extend, don't modify)

**Key principle: reproduce published numbers BEFORE any modifications.**

### Step 4: IMPLEMENT

1. Follow PPO's file structure exactly (pattern matching)
2. Use shared `networks.py` builders — extend if needed, don't modify existing
3. After implementing: run HalfCheetah, compare with published SAC results
4. Only proceed to comparisons after baseline reproduction verified

### Step 5: VERIFY
- [x] SAC reproduces published results on HalfCheetah (~8000 return at 1M steps)
- [x] SAC works with existing evaluator (no changes needed)
- [x] SAC configs follow same format as PPO configs
- [x] PPO results unchanged (no regression from adding SAC)
- [x] wandb logging works for SAC
- [x] Comparison scripts include both algorithms

### LEARN
- **Surprise:** published SAC paper used different network sizes than our PPO — had to add larger network configs to shared builders.
- **Would-do-differently:** check if shared infrastructure needs extension before estimating scope.

---

## Example 4: MODIFYING — Changing ML Feature Engineering

### Scenario

An ML fraud detection model uses manual features. Data scientist says: "Add transaction velocity features (transactions per hour) and change the time window from 24h to 72h."

### Step 1: CLASSIFY
- **Type:** MODIFYING (changing existing feature pipeline)
- **Risk:** HIGH — feature changes affect model AND serving pipeline

### Step 2: SURVEY

**Critical discovery:**
```bash
# Find feature computation
Grep: "time_window\|24.*hour\|feature" in src/features/**
# Result:
#   src/features/transaction_features.py (TIME_WINDOW = 24)
#   src/features/feature_store.py (caches features with 24h key)
#   src/serving/predict.py (recomputes features at inference time!)

# Find model training
Grep: "feature_columns\|input_dim" in src/model/**
# Result:
#   src/model/config.py (FEATURE_DIM = 45)
#   src/model/fraud_model.py (input_dim from config)
```

**Impact Report:**
- `src/features/transaction_features.py` — directly changed
- `src/features/feature_store.py` — cache key includes time window, must update
- `src/serving/predict.py` — CRITICAL: recomputes features live, must use SAME logic as training
- `src/model/config.py` — FEATURE_DIM changes from 45 to 48 (3 new velocity features)
- Model checkpoint — incompatible with new features, MUST retrain

**Train-serve skew risk:** Feature store computes offline (training), predict.py computes online (serving). Both MUST use identical logic.

### Step 3: SCOPE

**Tasks:**
1. Add velocity features to `src/features/transaction_features.py` — M
2. Update TIME_WINDOW from 24 to 72 — S
3. Update feature store cache key scheme — S
4. Update `src/serving/predict.py` to match new features — M
5. Update FEATURE_DIM in config — S
6. Retrain model with new features — L
7. Compare metrics: new model vs baseline — M

**Non-Goals:**
- NOT retraining with new architecture
- NOT changing the serving infrastructure
- NOT adding real-time feature computation

**Behaviors to preserve:**
- All existing 45 features compute identically
- Feature store caching works correctly
- Serving latency within SLA (< 100ms)

**CRITICAL: train-serve consistency**
- Feature computation in `transaction_features.py` and `predict.py` MUST be identical

### Step 5: VERIFY
- [x] Existing 45 features produce identical values (regression test)
- [x] 3 new velocity features compute correctly (unit tests)
- [x] Time window correctly set to 72h in both training and serving
- [x] Feature store cache invalidation works
- [x] Train-serve skew test: features from offline pipeline match online computation
- [x] Model performance: AUC 0.94 (from 0.91) — improved
- [x] Serving latency: 85ms (from 70ms) — within SLA
- [x] No data leakage from velocity features

### LEARN
- **Surprise:** feature store cache key included time window — changing from 24h to 72h invalidated all cached features.
- **Failed attempt:** forgot to update serving pipeline, caught by train-serve consistency test.
- **Would-do-differently:** always run train-serve skew test as first verification step.
