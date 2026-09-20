---
title: "How to Set Up Aider with Hicap"
date: "2026-01-26"
author: "Hicap Engineering"
description: "A step-by-step guide for Hicap customers to configure the Aider CLI against the Hicap OpenAI-compatible API."
---

# Setting Up Aider with Hicap

This guide walks you through configuring [Aider](https://aider.chat) to work with the Hicap API through its OpenAI-compatible endpoint.

Aider routes model traffic through LiteLLM internally, which is what lets it send the custom `api-key` header Hicap requires.

## Prerequisites

- Python 3.10 or newer
- Aider installed (`python -m pip install aider-install && aider-install`)
- Your Hicap API key (set as an environment variable):
  - **Windows (PowerShell)**: `$env:HICAP_API_KEY = "your-key-here"`
  - **Mac/Linux**: `export HICAP_API_KEY="your-key-here"`

## 1. Set the API Key

The API key should **never** be committed in plain text. Supply it as an environment variable that Aider will read at runtime.

- **Windows (PowerShell)**: `$env:HICAP_API_KEY = "your-key-here"`
- **Mac/Linux**: `export HICAP_API_KEY="your-key-here"`

## 2. Point Aider at Hicap

Hicap authenticates with a custom `api-key` header rather than the standard `Authorization: Bearer` header, so the base URL alone is not enough — you must also declare the header.

Create `.aider.model.settings.yml` in your home directory or repository root:

```yaml
- name: aider/extra_params
  extra_params:
    api_base: https://api.hicap.ai/v1
    api_key: dummy
    extra_headers:
      api-key: ${HICAP_API_KEY}
```

| Field | Purpose |
|---|---|
| `aider/extra_params` | Special model name that applies these settings to **every** model |
| `api_base` | Hicap API endpoint |
| `api_key` | Placeholder — the underlying client requires a non-empty value, but Hicap ignores it |
| `extra_headers.api-key` | The header Hicap actually authenticates on |

> **Note**: `api_key` is a deliberate placeholder. Hicap validates only the `api-key` header, so the value here is never used. Do not put your real key in this field.

## 3. Launch Aider

Prefix the model with `openai/` so Aider uses the OpenAI-compatible protocol against your `api_base`:

```bash
aider --model openai/claude-sonnet-4.6
```

To avoid passing the flag every time, add it to `.aider.conf.yml`:

```yaml
model: openai/claude-sonnet-4.6
```

> **Note**: Replace the model with one available from your Hicap account. You can view the full list at [hicap.ai/models](https://hicap.ai/models).

## 4. Verify

Aider prints the resolved model on startup. Ask it a question that does not touch your repository:

```bash
aider --model openai/claude-sonnet-4.6 --message "Reply with the single word: connected"
```

A reply confirms the Hicap backend is connected and routing correctly.

To confirm the endpoint independently of Aider:

```bash
curl https://api.hicap.ai/v1/chat/completions \
  -H "api-key: $HICAP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4.6",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `401` missing subscription key | `extra_headers` not applied | Confirm the settings file is named `.aider.model.settings.yml` and sits in your home directory or repo root |
| `401` invalid subscription key | Invalid or expired API key | Regenerate the key in the Hicap dashboard and re-export `HICAP_API_KEY` |
| Requests go to `api.openai.com` | `api_base` not picked up | Use the `aider/extra_params` model name exactly, and prefix your model with `openai/` |
| `model not found` | Model not enabled on your account | Check the available models at [hicap.ai/models](https://hicap.ai/models) |
| `ECONNREFUSED` to `api.hicap.ai` | Outbound networking blocked | Ensure the host allows egress to `https://api.hicap.ai` |

---

For the latest setup instructions, see: https://aider.chat/docs/config/adv-model-settings.html

Questions? Reach out to [Hicap support](mailto:support@hicap.ai) or open an issue in the repository.
