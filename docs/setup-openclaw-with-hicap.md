---
title: "How to Set Up OpenClaw with Hicap"
date: "2026-01-26"
author: "Hicap Engineering"
description: "A step-by-step guide for Hicap customers to configure OpenClaw with the Hicap API for multi-model, multi-provider AI development."
---

# Setting Up OpenClaw with Hicap

This guide walks you through configuring [OpenClaw](https://openclaw.ai) to work with the Hicap API for multi-model, multi-provider AI development.

## Prerequisites

- A running OpenClaw gateway instance
- Access to the `openclaw.json` configuration file
- Your Hicap API key (set as an environment variable):
  - **Windows (PowerShell)**: `$env:HICAP_API_KEY = "your-key-here"`
  - **Mac/Linux**: `export HICAP_API_KEY="your-key-here"`

## 1. Set the API Key

The API key should **never** be committed in plain text. Supply it as an environment variable that OpenClaw will interpolate at runtime.

Set the variable in your shell, a `.env` file, or through whatever secrets mechanism your deployment platform provides:

- **Windows (PowerShell)**: `$env:HICAP_API_KEY = "your-key-here"`
- **Mac/Linux**: `export HICAP_API_KEY="your-key-here"`

Make sure the `HICAP_API_KEY` environment variable is available to the OpenClaw process when it starts.

## 2. Register the Hicap Provider

In `openclaw.json`, add a `hicap` entry under `models.providers`. The provider uses the OpenAI-compatible completions API:

```json
{
  "models": {
    "providers": {
      "hicap": {
        "baseUrl": "https://api.hicap.ai/v1",
        "apiKey": "${HICAP_API_KEY}",
        "authHeader": true,
        "headers": {
          "api-key": "${HICAP_API_KEY}"
        },
        "api": "openai-completions",
        "models": []
      }
    }
  }
}
```

| Field | Purpose |
|---|---|
| `baseUrl` | Hicap API endpoint |
| `apiKey` | Interpolated from the `HICAP_API_KEY` environment variable |
| `authHeader` | Sends the key as a `Bearer` token in the `Authorization` header |
| `headers.api-key` | Also sends the key in a custom `api-key` header (required by Hicap) |
| `api` | Protocol to use — `openai-completions` for the chat completions interface |

## 3. Add Models

Inside the provider's `models` array, define each model you want to expose. Every entry needs at minimum an `id` and `name`:

```json
{
  "id": "<model-id>",
  "name": "<model-name>",
  "reasoning": false,
  "input": ["text"],
  "contextWindow": 128000,
  "maxTokens": 32000
}
```

> **Note**: Replace the model placeholders with models available from your Hicap account. You can view the full list at [hicap.ai/models](https://hicap.ai/models).

| Field | Purpose |
|---|---|
| `id` | The model identifier sent to the Hicap API |
| `name` | Display name in the Control UI |
| `reasoning` | Set to `true` for chain-of-thought models |
| `input` | Supported input modalities (`text`, `image`, etc.) |
| `contextWindow` | Maximum context length in tokens |
| `maxTokens` | Maximum output tokens per request |

## 4. Configure Agent Defaults

Under `agents.defaults.models`, map each provider model to an agent-accessible reference. Optionally assign short aliases:

```json
{
  "agents": {
    "defaults": {
      "models": {
        "hicap/claude-opus-4.6": { "alias": "opus" },
        "hicap/claude-sonnet-4.5": { "alias": "sonnet" },
        "hicap/claude-haiku-4.5": { "alias": "haiku" },
        "hicap/gpt-5.2": { "alias": "gpt" },
        "hicap/gpt-5-mini": { "alias": "gpt-mini" },
        "hicap/gemini-2.5-pro": { "alias": "gemini" },
        "hicap/gemini-2.5-flash": { "alias": "gemini-flash" }
      },
      "model": {
        "primary": "hicap/claude-opus-4.6",
        "fallbacks": ["hicap/claude-sonnet-4.5", "hicap/gpt-5.2"]
      }
    }
  }
}
```

- **`primary`** — the default model used for all agent interactions.
- **`fallbacks`** — ordered list of models to try if the primary is unavailable or rate-limited.
- **`alias`** — short names users can type in the Control UI instead of the full `provider/model` path.

## 5. Restart the Gateway

After updating `openclaw.json`, restart your OpenClaw gateway for the changes to take effect. The exact command depends on how you run the gateway (e.g., restarting a container, systemd service, or process).

## 6. Verify

Open the Control UI at your gateway endpoint (e.g., `https://your-gateway-host/overview`). You should see all the Hicap models listed and available for selection.

To test from the command line:

```bash
curl -X POST https://your-gateway-host/v1/chat/completions \
  -H "Authorization: Bearer $MOLTBOT_GATEWAY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "hicap/claude-opus-4.6",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

A successful response confirms the Hicap backend is connected and routing correctly.

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `401 Unauthorized` from Hicap | Invalid or expired API key | Regenerate the key in the Hicap dashboard and redeploy |
| Model not listed in Control UI | Missing from `models` array or `agents.defaults.models` | Add the model entry to both sections in `openclaw.json` |
| `ECONNREFUSED` to `api.hicap.ai` | Outbound networking blocked | Ensure the host allows egress to `https://api.hicap.ai` |
| Fallback model used unexpectedly | Primary model rate-limited | Check Hicap usage limits; consider upgrading your plan |

---

For the latest setup instructions, see: https://openclaw.ai

Questions? Reach out to [Hicap support](mailto:support@hicap.ai) or open an issue in the repository.
