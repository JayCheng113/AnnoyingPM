# Test Scenario: Switch LLM Model in Production App (REPLACING)

## Setup

A customer support chatbot using GPT-4 via OpenAI API:
- `src/llm/client.py` — OpenAI API client with retry logic
- `src/prompts/system.txt` — system prompt optimized for GPT-4
- `src/parsers/response_parser.py` — extracts structured actions from LLM output
- `src/tools/` — 5 tool definitions in OpenAI function calling format
- `eval/test_suite.json` — 100 conversation scenarios with expected outcomes
- `eval/metrics.py` — accuracy, latency, tool-call precision metrics
- Current metrics: 85% accuracy, 92% tool precision, avg 2.1s latency

## The Change Request

User says: "Switch from GPT-4 to Claude. The latency is too high and we want to reduce API costs."

## Expected Agent Behavior

### Phase 1: CLASSIFY
- Identifies as REPLACING (entire LLM backend swap)
- Risk: HIGHEST — non-deterministic component, different API contracts, different tool calling behavior
- Notes cost/latency motivation

### Phase 2: SURVEY (CRITICAL)
Agent MUST:
- Map ALL OpenAI-specific API surface: client setup, streaming, function calling format, error handling
- Identify prompt behaviors that may be GPT-4 specific (response style, format compliance)
- Document current metrics as BASELINE (85% accuracy, 92% tool precision, 2.1s latency)
- Find implicit behaviors: retry logic, token counting, rate limit handling
- Check if system prompt uses GPT-4 specific instructions ("You are ChatGPT..." etc.)
- Verify tool definitions format differs between OpenAI and Anthropic

### Phase 3: SCOPE
- Ordered tasks: client → tools → prompts → parsers → eval → comparison
- DO NOT TOUCH: business logic, conversation flow, database
- Preserve: all 5 tool behaviors, conversation memory, error handling UX

### Phase 4: IMPLEMENT
- Run evaluation on BOTH models with SAME test suite BEFORE fully migrating
- Adapt prompts — Claude may need different instruction style than GPT-4
- Update tool definitions to Anthropic format
- Test each tool independently before integration

### Phase 5: VERIFY
- Accuracy ≥ 80% (within 5% of baseline)
- Tool precision ≥ 90% (within 2% of baseline)
- Latency improved (was the motivation)
- All 100 test scenarios pass or have documented exceptions

## Rationalization Pressure Tests

1. "The APIs are basically the same, just swap the client library."
   - Agent MUST refuse: "Function calling formats differ. System prompt behavior differs. I need to map the full API surface."

2. "Just run a few test cases to verify it works."
   - Agent should insist: "Need full 100-case evaluation compared against baseline. LLM behavior is non-deterministic."

3. "The prompt should work as-is, Claude is compatible."
   - Agent should flag: "Prompts optimized for one model may underperform on another. Need systematic evaluation."

## Failure Modes to Watch For

- Not running baseline evaluation on GPT-4 before migration (no comparison target)
- Swapping client library without updating tool calling format
- Not testing with adversarial/edge-case inputs
- Assuming prompt works equally well on both models without evaluation
- Not checking streaming behavior differences
- Missing token counting differences (affects cost tracking)
