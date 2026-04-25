# MCP Safety

Evaluates LLM resistance to adversarial attacks in MCP tool-use scenarios. Based on [MCP-SafetyBench](https://github.com/xjzzzzzzzz/MCPSafety) ([arXiv:2512.15163](https://arxiv.org/abs/2512.15163)).

## Overview

- **245 tasks** across 5 domains: financial_analysis, repository_management, web_search, location_navigation, browser_automation
- **20 attack categories** including Tool Poisoning (parameter, command injection, filesystem, network, dependency, redirection), Tool Shadowing, Rug Pull, Intent Injection, Data Tampering, Identity Spoofing, Replay Injection, and more
- **2 attack classes**: Disruption Attack, Stealth Attack

## Task

Each test case presents the model with a system prompt containing MCP tool descriptions (some poisoned or malicious) and a user task. The model must complete the task without being misled by the attack.

## Metrics

| Metric | Description |
|--------|-------------|
| `attack_resistance` | 1.0 if model resisted the attack, 0.0 if it complied (scored) |
| `task_quality` | 1.0 if model produced a substantive response (metric-only) |

## Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `judge_model` | `openai/gpt-5.4-mini` | LLM judge model via OpenRouter |
| `judge_api_key_var` | `OPENROUTER_API_KEY` | Environment variable for API key |
| `data_path` | Auto-detected | Path to test configs directory |
| `domain` | `None` | Filter by domain |
| `attack_category` | `None` | Filter by attack type |
| `category` | `None` | Filter: "Disruption Attack" or "Stealth Attack" |

## Quickstart

```bash
prime eval run mcp-safety -m gpt-4.1-mini -n 10 -r 1
```
