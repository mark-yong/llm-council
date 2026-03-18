# llm-council
A Karpathy-inspired LLM council using Langgraph

This folder contains a LangGraph-based “LLM council” implementation inspired by Andrej Karpathy’s `llm-council` (3-stage pipeline: generate candidates, have a panel rank anonymously, then have a chairman synthesize).

## Departures

### 1) Stage-1 label assignment is randomized

- **Difference**: we call `random.shuffle(answers)` before assigning `Response A/B/C/...` labels.
- **Karpathy behavior**: Karpathy labels in the original stage-1 order (no shuffle).
- **Reason**: reduces positional bias and label-to-model association.
- **Tradeoff**: run-to-run label-to-model mapping changes, but we persist `metadata["shuffle_seed"]` so the shuffle is reproducible for debugging.

### 2) Aggregate scoring excludes self-votes

- **Difference**: when computing average ranks, we exclude votes where `model == evaluator`.
- **Karpathy behavior**: the example implementation does not explicitly remove self-votes.
- **Why exclude self-votes**: reduces conflict-of-interest bias in peer aggregation.
- **Tradeoff**: behavioral deviation from Karpathy (but often makes “council ranking” semantics cleaner).

### 3) Stage-3 inputs are “text bundles”, and the chair prompt discourages provenance leakage

- **Difference**: the chairman sees:
  - stage-1 model names + answers
  - stage-2 evaluator writeups (including evaluator names)
- **Difference**: the chairman prompt instructs it not to mention internal labels/provenance in the user-facing final answer.
- **Why**: keeps the user-visible output product-like while still letting the chair use evaluator reasoning text.
- **Tradeoff**: prompt contract differs slightly from Karpathy.

## UV Setup

Create/refresh the virtual environment and install pinned dependencies:
- `uv sync --locked`