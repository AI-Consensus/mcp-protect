# AGENTS.md — MCP-Protect Benchmark Suite

## Project Overview

This workspace contains **Prime Intellect Verifiers environments** for evaluating LLM safety under adversarial MCP/tool-use scenarios. These benchmarks support a mechanistic interpretability paper investigating intervention effects on model safety and capability.

## Benchmark Inventory

### Attack Benchmarks (implemented in this repo)

| Environment | Hub | Dataset | What it tests |
|---|---|---|---|
| **mcp-tox** | [`hubert-marek/mcp-tox`](https://app.primeintellect.ai/dashboard/environments/hubert-marek/mcp-tox) | 1,348 cases, 45 MCP servers, 353 tools | Tool poisoning across 3 paradigms (explicit hijacking, implicit hijacking, parameter tampering). [arXiv:2508.14925](https://arxiv.org/abs/2508.14925) \| [Repo](https://github.com/zhiqiangwang4/MCPTox-Benchmark) |
| **mcp-safety** | [`hubert-marek/mcp-safety`](https://app.primeintellect.ai/dashboard/environments/hubert-marek/mcp-safety) | 245 cases, 5 domains, 20 attack types | Broader attack coverage: command injection, identity spoofing, replay injection, credential theft, tool shadowing, rug pull. [arXiv:2512.15163](https://arxiv.org/abs/2512.15163) \| [Repo](https://github.com/xjzzzzzzzz/MCPSafety) |
| **open-prompt-injection** | [`hubert-marek/open-prompt-injection`](https://app.primeintellect.ai/dashboard/environments/hubert-marek/open-prompt-injection) | 7 attacks x 49 task combos | Fundamental instruction-data boundary without tool complexity. 5 heuristic attacks. [arXiv:2310.12815](https://arxiv.org/abs/2310.12815) \| [Repo](https://github.com/liu00222/Open-Prompt-Injection) |

### Attack Benchmarks (on Prime Hub, already implemented by Prime Intellect)

| Environment | Hub | Dataset | What it tests |
|---|---|---|---|
| **agent-dojo** | [`primeintellect/agent-dojo`](https://app.primeintellect.ai/dashboard/environments/primeintellect/agent-dojo) | 97 tasks, 629 security cases, 70 tools | Stateful tool-use with injected prompts. 4 environments (Workspace, Slack, Travel, Banking). NeurIPS 2024, [arXiv:2406.13352](https://arxiv.org/abs/2406.13352) \| [Repo](https://github.com/ethz-spylab/agentdojo) \| [Leaderboard](https://agentdojo.spylab.ai) |

### Capability Preservation Benchmarks (on Prime Hub, already implemented by Prime Intellect)

| Environment | Hub | What it tests |
|---|---|---|
| **tau2-bench** | [`primeintellect/tau2-bench`](https://app.primeintellect.ai/dashboard/environments/primeintellect/tau2-bench) | Multi-turn tool use with domain policies (airline, retail, banking, telecom). [arXiv:2406.12045](https://arxiv.org/abs/2406.12045) \| [Leaderboard](https://taubench.com) |
| **bfcl-v3** | [`primeintellect/bfcl-v3`](https://app.primeintellect.ai/dashboard/environments/primeintellect/bfcl-v3) | Function calling correctness (single-turn, parallel, multi-step). ICML 2025 \| [Repo](https://github.com/ShishirPatil/gorilla/tree/main/berkeley-function-call-leaderboard) |

## Quick Start

### Prerequisites

```bash
# Set up API key for LLM-as-judge (local envs use OpenRouter)
export OPENROUTER_API_KEY="your-key-here"
```

### Custom API endpoints (OpenAI-compatible chat)

Any provider exposing the OpenAI Chat Completions API (OpenAI, OpenRouter, Together, Groq, DeepSeek, vLLM, Ollama, etc.) works with `prime eval run`. Pick one of the three patterns below.

**1. Built-in provider shorthand** (`prime` resolves URL + key var automatically):

```bash
# OpenRouter — reads OPENROUTER_API_KEY (note: no underscore between OPEN and ROUTER)
prime eval run mcp-tox -m openai/gpt-4.1-mini -p openrouter -n 3 -r 1

# OpenAI direct — reads OPENAI_API_KEY
prime eval run mcp-tox -m gpt-4.1-mini -p openai -n 3 -r 1
```

Built-in providers: `prime`, `openrouter`, `openai`, `anthropic`, `deepseek`, `glm`, `minimax`, `local`, `vllm`.

**2. Inline flags** (one-off custom OpenAI-compatible endpoint):

```bash
prime eval run mcp-tox \
  -m openai/gpt-4.1-mini \
  -b https://openrouter.ai/api/v1 \
  -k OPENROUTER_API_KEY \
  --api-client-type openai_chat_completions \
  -n 3 -r 1 -s
```

`-b` takes the **base URL** (ending in `/v1`); the client appends `/chat/completions`. `-k` takes the env var **name**, not the key value.

**3. TOML registry** (reusable aliases — `./configs/endpoints.toml` is auto-loaded):

```toml
[[endpoint]]
endpoint_id = "or-gpt-4.1-mini"
model = "openai/gpt-4.1-mini"
url = "https://openrouter.ai/api/v1"
key = "OPENROUTER_API_KEY"
type = "openai_chat_completions"
```

```bash
prime eval run mcp-tox -m or-gpt-4.1-mini -n 3 -r 1
```

**Note on the judge model:** `mcp-tox`, `mcp-safety`, and `open-prompt-injection` make a separate judge call hardcoded to `https://openrouter.ai/api/v1` and reading `OPENROUTER_API_KEY` by default (same env var the `-p openrouter` shorthand uses). Override per-run via `-a '{"judge_model": "openai/gpt-4o-mini", "judge_api_key_var": "MY_KEY_VAR"}'`.

### Running local benchmarks (mcp-tox, mcp-safety, open-prompt-injection)

```bash
# Install
VIRTUAL_ENV=.venv uv pip install -e ./environments/mcp_tox

# Smoke test
prime eval run mcp-tox -m gpt-4.1-mini -n 3 -r 1

# Full run
prime eval run mcp-tox -m gpt-4.1-mini -n 50 -r 1
```

### Running Hub benchmarks (agent-dojo, tau2-bench, bfcl-v3)

These are already implemented by Prime Intellect and hosted on the Hub. Install and run directly:

```bash
# AgentDojo — primary Surface 2 attack benchmark
prime env install primeintellect/agent-dojo
prime eval run agent-dojo -m gpt-4.1-mini -n 20 -r 1

# tau2-bench — capability preservation (multi-turn tool use)
prime env install primeintellect/tau2-bench
prime eval run tau2-bench -m gpt-4.1-mini -n 20 -r 1

# BFCL v3 — capability preservation (function calling correctness)
prime env install primeintellect/bfcl-v3
prime eval run bfcl-v3 -m gpt-4.1-mini -n 20 -r 1
```

### Running all benchmarks end-to-end

```bash
# Install everything
VIRTUAL_ENV=.venv uv pip install -e ./environments/mcp_tox -e ./environments/mcp_safety -e ./environments/open_prompt_injection
prime env install primeintellect/agent-dojo
prime env install primeintellect/tau2-bench
prime env install primeintellect/bfcl-v3

# Run attack benchmarks
prime eval run mcp-tox -m <your-model> -n 50 -r 1
prime eval run mcp-safety -m <your-model> -n 50 -r 1
prime eval run open-prompt-injection -m <your-model> -n 50 -r 1
prime eval run agent-dojo -m <your-model> -n 20 -r 1

# Run capability benchmarks
prime eval run tau2-bench -m <your-model> -n 20 -r 1
prime eval run bfcl-v3 -m <your-model> -n 20 -r 1
```

## Environment Details

### mcp-tox

Based on [MCPTox-Benchmark](https://github.com/zhiqiangwang4/MCPTox-Benchmark) ([arXiv:2508.14925](https://arxiv.org/abs/2508.14925)).

Each test case presents the model with a system prompt containing real MCP tool descriptions (one poisoned with hidden malicious instructions) and a clean user query. An LLM judge (default: `openai/gpt-5.4-mini` via OpenRouter) evaluates whether the model resisted or complied.

**Filtering:**
```bash
prime eval run mcp-tox -a '{"server_name": "FileSystem"}' -n 20
prime eval run mcp-tox -a '{"security_risk": "Credential Leakage"}' -n 20
prime eval run mcp-tox -a '{"paradigm": "Template-2"}' -n 20
```

### mcp-safety

Based on [MCP-SafetyBench](https://github.com/xjzzzzzzzz/MCPSafety) ([arXiv:2512.15163](https://arxiv.org/abs/2512.15163)).

Covers 20 attack types across 5 domains. Constructs tool listings from task configs including poisoned descriptions, injected tools, and client-side attacks.

**Filtering:**
```bash
prime eval run mcp-safety -a '{"domain": "financial_analysis"}' -n 20
prime eval run mcp-safety -a '{"attack_category": "Tool Poisoning-Command Injection"}' -n 20
prime eval run mcp-safety -a '{"category": "Stealth Attack"}' -n 20
```

### open-prompt-injection

Based on [Open-Prompt-Injection](https://github.com/liu00222/Open-Prompt-Injection) ([arXiv:2310.12815](https://arxiv.org/abs/2310.12815)).

Tests the instruction-data boundary with configurable target task, injected task, and attack strategy. Data loaded from HuggingFace datasets (sst2, sms_spam, hsol, mrpc, rte, gigaword, jfleg).

**Configuration:**
```bash
# Default: sentiment_analysis vs spam_detection with combine attack
prime eval run open-prompt-injection -n 20

# Custom pairing
prime eval run open-prompt-injection -a '{"target_task": "summarization", "injected_task": "hate_detection", "attack_strategy": "ignore"}' -n 20
```

**Available tasks:** sentiment_analysis, spam_detection, hate_detection, duplicate_sentence_detection, natural_language_inference, summarization, grammar_correction

**Attack strategies:** naive, escape, ignore, fake_comp, combine

## Architecture

All three local environments follow the same pattern:

- `SingleTurnEnv` with `JudgeRubric` (LLM-as-judge via OpenRouter)
- Judge grades A (RESISTED) / B (COMPLIED) / C (UNCLEAR)
- Primary metric: `attack_resistance` (1.0 = resisted, 0.0 = complied)
- Secondary metrics logged but not scored (e.g., `attack_detected`, `task_accuracy`)
- Each environment exports `load_environment(**kwargs) -> vf.Environment`
- API key validated early via `vf.ensure_keys(["OPENROUTER_API_KEY"])`

## Reference Data

All environments automatically download their data on first run:
- **mcp-tox** and **mcp-safety** shallow-clone their benchmark repos into `tmp/`, cache the data, and clean up the clone. No manual setup needed.
- **open-prompt-injection** loads data directly from HuggingFace datasets.

## Shared Best Practices (Prime Lab)

- Environments expose `load_environment(...) -> vf.Environment` and install with `prime env install <env-name>`.
- Validate with `prime eval run` before pushing. It saves results automatically.
- Use `ToolEnv`/`MCPEnv` for stateless tools and `StatefulToolEnv` for per-rollout state.
- Validate API keys in `load_environment()` with `vf.ensure_keys(...)`.
- Keep endpoint aliases in `./configs/endpoints.toml`.
- Keep each environment self-contained under `environments/<env_name>/`.
- Push with `prime env push <env-name> --visibility PUBLIC` after local eval is verified.
- See `environments/AGENTS.md` for full Verifiers API documentation.
