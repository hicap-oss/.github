---
title: "How to Set Up OpenHands with Hicap"
date: "2026-01-26"
author: "Hicap Engineering"
description: "A step-by-step guide for Hicap customers to configure OpenHands against the Hicap OpenAI-compatible API."
---

# Setting Up OpenHands with Hicap

This guide walks you through configuring [OpenHands](https://github.com/All-Hands-AI/OpenHands) to work with the Hicap API through its OpenAI-compatible endpoint.

> **Important**: Hicap requires a custom `api-key` header. Only the **v1 / agent SDK** path supports custom headers. The legacy V0 `config.toml` `[llm]` block accepts `model`, `api_key`, and `base_url` but has no header option, so it cannot reach Hicap directly. See Legacy V0 below for the workaround.

## Prerequisites

- Python 3.12 or newer
- The OpenHands agent SDK installed (`pip install openhands-sdk`)
- Your Hicap API key

## 1. Set the API Key

The API key should **never** be committed in plain text. Supply it as an environment variable read at runtime.

- **Windows (PowerShell)**: `$env:HICAP_API_KEY = "your-key-here"`
- **Mac/Linux**: `export HICAP_API_KEY="your-key-here"`

## 2. Configure the LLM

Hicap authenticates with a custom `api-key` header rather than the standard `Authorization: Bearer` header, so set `extra_headers` alongside `base_url`:

```python
import os
from pydantic import SecretStr
from openhands.sdk import LLM

llm = LLM(
    model="openai/claude-sonnet-4.6",
    base_url="https://api.hicap.ai/v1",
    api_key=SecretStr("dummy"),
    extra_headers={"api-key": os.environ["HICAP_API_KEY"]},
    usage_id="hicap",
)
```

| Field | Purpose |
|---|---|
| `model` | Prefix with `openai/` so the OpenAI-compatible protocol is used against `base_url` |
| `base_url` | Hicap API endpoint |
| `api_key` | Placeholder, required by the client and ignored by Hicap |
| `extra_headers` | The `api-key` header Hicap actually authenticates on |
| `usage_id` | Label for per-LLM usage accounting |

> **Note**: `api_key` is a deliberate placeholder. Hicap validates only the `api-key` header, so the value here is never used. Do not put your real key in this field.

> **Note**: Replace the model with one available from your Hicap account. You can view the full list at [hicap.ai/models](https://hicap.ai/models).

## 3. Run an Agent

Pass the configured `llm` into your agent as usual:

```python
from openhands.sdk import Agent, Conversation

agent = Agent(llm=llm, tools=[])
conversation = Conversation(agent=agent)
conversation.send_message("Reply with the single word: connected")
conversation.run()
```

## 4. Verify

A reply confirms the Hicap backend is connected and routing correctly. To confirm the endpoint independently of OpenHands:

```bash
curl https://api.hicap.ai/v1/chat/completions \
  -H "api-key: $HICAP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4.6",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

## Legacy V0 config.toml

If you are pinned to the legacy V0 release, the `[llm]` block cannot send custom headers. Run a LiteLLM proxy in front of Hicap that accepts a standard bearer token and re-issues requests with the `api-key` header, then point OpenHands at the proxy:

```toml
[llm]
model = "openai/claude-sonnet-4.6"
base_url = "http://localhost:4000"
api_key = "your-litellm-master-key"
```

See the LiteLLM setup guide for the proxy configuration.

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `401` missing subscription key | `extra_headers` omitted, or you are on legacy V0 | Confirm `extra_headers` is set on the `LLM`; on V0 use the LiteLLM proxy above |
| `401` invalid subscription key | Invalid or expired API key | Regenerate the key in the Hicap dashboard and re-export `HICAP_API_KEY` |
| Requests go to `api.openai.com` | `base_url` omitted or model missing the `openai/` prefix | Set both `base_url` and an `openai/`-prefixed model |
| `model not found` | Model not enabled on your account | Check the available models at [hicap.ai/models](https://hicap.ai/models) |
| `ECONNREFUSED` to `api.hicap.ai` | Outbound networking blocked | Ensure the host allows egress to `https://api.hicap.ai` |

---

For the latest setup instructions, see: https://docs.all-hands.dev

Questions? Reach out to [Hicap support](mailto:support@hicap.ai) or open an issue in the repository.
