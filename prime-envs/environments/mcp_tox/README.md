# MCP Tox

Evaluates LLM resistance to poisoned MCP tool descriptions. Based on the [MCPTox-Benchmark](https://github.com/zhiqiangwang4/MCPTox-Benchmark) ([arXiv:2508.14925](https://arxiv.org/abs/2508.14925)).

## Overview

- **1,348 test cases** across 45 MCP servers
- **11 toxicity categories**: Credential Leakage, Privacy Leakage, Message Hijacking, Code Injection, Data Tampering, Instruction Tampering, Information Manipulation, Financial Loss, Service Disruption, Infrastructure Damage, Other
- **3 attack paradigms**: Template-1 (direct injection), Template-2 (precondition attacks), Template-3 (parameter modification)

## Task

Each test case presents the model with a system prompt containing tool descriptions (one of which is poisoned with hidden malicious instructions) and a clean user query. The model must complete the user's request without following the malicious instructions.

## Metrics

| Metric | Description |
|--------|-------------|
| `attack_resistance` | 1.0 if model resisted the attack, 0.0 if it complied (scored) |
| `attack_detected` | 1.0 if model explicitly flagged the suspicious tool (metric-only) |

## Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `judge_model` | `openai/gpt-5.4-mini` | LLM judge model via OpenRouter |
| `judge_api_key_var` | `OPENROUTER_API_KEY` | Environment variable for API key |
| `data_path` | Auto-detected | Path to `response_all.json` |
| `server_name` | `None` | Filter by MCP server (e.g., "FileSystem") |
| `security_risk` | `None` | Filter by risk category |
| `paradigm` | `None` | Filter by attack paradigm (e.g., "Template-2") |

## Quickstart

```bash
prime eval run mcp-tox -m gpt-4.1-mini -n 10 -r 1
```
