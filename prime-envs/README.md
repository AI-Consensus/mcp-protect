# MCP-Protect Benchmark Suite

Prime Intellect Verifiers environments for evaluating LLM safety under adversarial MCP/tool-use scenarios.

## Setup

### 1. Create venv and install local environments

```bash
cd prime-envs
uv venv .venv
source .venv/bin/activate
VIRTUAL_ENV=.venv uv pip install -e ./environments/mcp_tox -e ./environments/mcp_safety -e ./environments/open_prompt_injection
```

### 2. Install hub environments (optional)

```bash
prime env install primeintellect/agent-dojo   # currently broken — see Known issues
prime env install primeintellect/tau2-bench
prime env install primeintellect/bfcl-v3
```

The venv must be activated so `prime env install` writes into it.

### 3. Set the API key

The local envs use OpenRouter for the LLM-as-judge:

```bash
export OPENROUTER_API_KEY="sk-or-..."
```

## Custom API endpoints (OpenAI-compatible chat)

Three ways to wire up an inference provider.

**1. Built-in provider shorthand** (`-p` resolves URL + key var automatically):

```bash
vf-eval mcp-tox -m openai/gpt-4.1-mini -p openrouter -n 3 -r 1
vf-eval mcp-tox -m gpt-4.1-mini -p openai -n 3 -r 1
```

Built-in providers: `prime`, `openrouter`, `openai`, `anthropic`, `deepseek`, `glm`, `minimax`, `local`, `vllm`.

**2. Inline flags** (one-off custom endpoint):

```bash
vf-eval mcp-tox \
  -m openai/gpt-4.1-mini \
  -b https://openrouter.ai/api/v1 \
  -k OPENROUTER_API_KEY \
  --api-client-type openai_chat_completions \
  -n 3 -r 1
```

`-b` is the base URL ending in `/v1` (the client appends `/chat/completions`). `-k` is the env var **name**, not the key value.

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
vf-eval mcp-tox -m or-gpt-4.1-mini -n 3 -r 1
```

For local servers (vLLM, Ollama, LM Studio) the named env var doesn't need to exist — verifiers substitutes `"EMPTY"` when unset:

```toml
[[endpoint]]
endpoint_id = "local-llama"
model = "meta-llama/Llama-3.1-8B-Instruct"
url = "http://localhost:8000/v1"
key = "VLLM_API_KEY"
type = "openai_chat_completions"
```

## Running benchmarks

Use `vf-eval` (the verifiers script) — **not** `prime eval run`, which does an upfront Prime Intellect billing check that fails even when routing through OpenRouter.

```bash
cd prime-envs
source .venv/bin/activate
set -a; source ../.env; set +a    # if your key lives in a .env file

# Smoke tests
vf-eval mcp-tox               -m openai/gpt-4.1-mini -p openrouter -n 3 -r 1 -s
vf-eval mcp-safety            -m openai/gpt-4.1-mini -p openrouter -n 3 -r 1 -s
vf-eval open-prompt-injection -m openai/gpt-4.1-mini -p openrouter -n 3 -r 1 -s

# Hub envs
vf-eval tau2-bench            -m openai/gpt-4.1-mini -p openrouter -n 3 -r 1 -s
vf-eval bfcl-v3               -m openai/gpt-4.1-mini -p openrouter -n 3 -r 1 -s

# Full runs (just bump -n)
vf-eval mcp-tox    -m openai/gpt-4.1-mini -p openrouter -n 50 -r 1 -s
vf-eval mcp-safety -m openai/gpt-4.1-mini -p openrouter -n 50 -r 1 -s
```

Common flags:
- `-n N` — first N examples (deterministic, not random)
- `-r N` — rollouts per example
- `-s` — save results to disk
- `-a JSON` — env-specific kwargs, e.g. `-a '{"server_name": "FileSystem"}'` for mcp-tox

## Known issues

- **`primeintellect/agent-dojo`** (hub) imports `ChatCompletionMessage` from `verifiers.utils.message_utils` (doesn't exist in any published verifiers version) and uses a removed `info['oai_tools']` API. Blocked until the hub package is republished.
- **`prime eval run`** does an upfront Prime account billing check and refuses to run if your Prime balance is empty — even with `-p openrouter`. Always use `vf-eval` instead.
