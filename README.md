# mcp-protect

This repo has two connected modules:

- `axbench/` trains and serves a **local steered policy** (Gemma + HyperSteer).
- `prime-envs/` runs **MCP safety benchmarks** with `vf-eval` against any OpenAI-compatible endpoint.

## What each module does

- **`axbench`**: merges steering concepts (Neuronpedia + `mcpattack.jsonl`), generates training data, trains HyperSteer, and serves `/v1/chat/completions` via `serve_mcp_hypersteer.py`.
- **`prime-envs`**: runs benchmark environments (`mcp-tox`, etc.) and sends HTTP requests to your chosen policy endpoint.

## Setup by module

### 1) Setup `axbench` (training + local policy)

Use this when you want a **local steered model**:

- Full instructions: [axbench/axbench/mcp-protect/README.md](axbench/axbench/mcp-protect/README.md)
- Typical outputs:
  - training artifacts under `<dump-dir>/train/`
  - metadata under `<dump-dir>/generate/`
- Serve command (from AxBench project root):

```bash
python axbench/mcp-protect/serve_mcp_hypersteer.py --dump-dir <your-run-dir> --port 8000
```

### 2) Setup `prime-envs` (benchmark runner)

Use this for evaluation:

- Full setup and providers: [prime-envs/README.md](prime-envs/README.md)
- Benchmark conventions: [prime-envs/AGENTS.md](prime-envs/AGENTS.md)

## How to link them together

1. Start your local policy server in `axbench`. It prints a base URL like:
   - `http://127.0.0.1:8000/v1`
2. Activate `prime-envs` venv and run `vf-eval` against that URL:

```bash
cd prime-envs && source .venv/bin/activate
export OPENROUTER_API_KEY="sk-or-..."   # if the environment uses a remote judge

vf-eval mcp-tox \
  -b http://127.0.0.1:8000/v1 \
  -k EMPTY \
  --api-client-type openai_chat_completions \
  -m local-hypersteer \
  -n 3 -r 1 -s
```

You can also define `local-hypersteer` in [prime-envs/configs/endpoints.toml](prime-envs/configs/endpoints.toml) and run with that endpoint id/model alias.

## Keys and what they are for

- `OPENAI_API_KEY`: used by `axbench` only when generating training data with a remote LM.
- `OPENROUTER_API_KEY`: often used by `prime-envs` judge models or remote baselines.
- `HYPERSTEER_CONCEPT_ID`, `HYPERSTEER_FACTOR`: optional runtime steering controls for `serve_mcp_hypersteer.py`.

## Important constraint

For steered-policy evaluation, the policy must run on your machine (or your own server) where HyperSteer is applied. OpenRouter can be used for judges/baselines, but not as the in-process steered policy.

