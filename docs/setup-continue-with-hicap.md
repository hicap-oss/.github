---
title: "How to Set Up Continue with Hicap"
date: "2026-01-26"
author: "Hicap Engineering"
description: "A step-by-step guide for Hicap customers to configure the Continue extension for VS Code and JetBrains against the Hicap OpenAI-compatible API."
---

# Setting Up Continue with Hicap

This guide walks you through configuring [Continue](https://continue.dev) to work with the Hicap API through its OpenAI-compatible endpoint.

## Prerequisites

- The Continue extension installed in VS Code or a JetBrains IDE
- Access to your Continue configuration at `~/.continue/config.yaml`
- Your Hicap API key

## 1. Store the API Key

Continue resolves secrets through its own secret store rather than shell environment variables, referenced with `${{ secrets.NAME }}`.

Add your key to `~/.continue/.env`:

```bash
HICAP_API_KEY=your-key-here
```

The API key should **never** be committed in plain text. Keep `.env` out of version control.

## 2. Add Hicap as a Model Provider

Hicap authenticates with a custom `api-key` header rather than the standard `Authorization: Bearer` header. Continue supports this through `requestOptions.headers`.

In `~/.continue/config.yaml`:

```yaml
name: hicap-assistant
version: 0.0.1
schema: v1
models:
  - name: Claude Sonnet 4.6 (Hicap)
    provider: openai
    model: claude-sonnet-4.6
    apiBase: https://api.hicap.ai/v1
    apiKey: dummy
    roles:
      - chat
      - edit
      - apply
    requestOptions:
      headers:
        api-key: ${{ secrets.HICAP_API_KEY }}
```

| Field | Purpose |
|---|---|
| `provider` | Must be `openai` so Continue speaks the OpenAI-compatible protocol |
| `model` | The model identifier sent to the Hicap API |
| `apiBase` | Hicap API endpoint |
| `apiKey` | Placeholder — required by the provider, ignored by Hicap |
| `requestOptions.headers.api-key` | The header Hicap actually authenticates on |
| `roles` | Which Continue features this model serves |

> **Note**: `apiKey` is a deliberate placeholder. Hicap validates only the `api-key` header, so the value here is never used. Do not put your real key in this field.

## 3. Add More Models

Each model is its own entry in the `models` array. Repeat the block and change `name` and `model`:

```yaml
  - name: GPT-5 (Hicap)
    provider: openai
    model: gpt-5
    apiBase: https://api.hicap.ai/v1
    apiKey: dummy
    roles:
      - chat
    requestOptions:
      headers:
        api-key: ${{ secrets.HICAP_API_KEY }}
```

> **Note**: Replace the model placeholders with models available from your Hicap account. You can view the full list at [hicap.ai/models](https://hicap.ai/models).

## 4. Reload and Verify

Reload your IDE window so Continue re-reads the configuration, then open the Continue sidebar. Your Hicap models should appear in the model dropdown.

Send a short prompt to confirm routing. To confirm the endpoint independently of Continue:

```bash
curl https://api.hicap.ai/v1/chat/completions \
  -H "api-key: $HICAP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4.6",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

A successful response confirms the Hicap backend is connected and routing correctly.

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `401` missing subscription key | `requestOptions.headers` omitted or misindented | Confirm `headers` nests under `requestOptions` on the same model entry |
| `401` invalid subscription key | Secret not resolved | Confirm `HICAP_API_KEY` is in `~/.continue/.env` and reload the window |
| Model missing from dropdown | YAML parse error | Check indentation; Continue silently skips malformed entries |
| Requests go to `api.openai.com` | `apiBase` omitted | Add `apiBase` to each model entry — it is not inherited |
| `ECONNREFUSED` to `api.hicap.ai` | Outbound networking blocked | Ensure the host allows egress to `https://api.hicap.ai` |

---

For the latest setup instructions, see: https://docs.continue.dev/reference

Questions? Reach out to [Hicap support](mailto:support@hicap.ai) or open an issue in the repository.
