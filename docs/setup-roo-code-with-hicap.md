---
title: "How to Set Up Roo Code with Hicap"
date: "2026-01-26"
author: "Hicap Engineering"
description: "A step-by-step guide for Hicap customers to configure the Roo Code VS Code extension against the Hicap OpenAI-compatible API."
---

# Setting Up Roo Code with Hicap

This guide walks you through configuring [Roo Code](https://roocode.com) to work with the Hicap API through its OpenAI-compatible endpoint.

Roo Code exposes a **Custom Headers** field on its OpenAI Compatible provider, which is what lets it send the `api-key` header Hicap requires.

## Prerequisites

- VS Code with the Roo Code extension installed
- Your Hicap API key

## 1. Open Provider Settings

1. Click the Roo Code icon in the VS Code activity bar.
2. Open the settings gear, then **Providers**.
3. Create a new configuration profile so your Hicap setup does not overwrite an existing provider.

## 2. Select the OpenAI Compatible Provider

Set **API Provider** to **OpenAI Compatible**, then fill in:

| Field | Value |
|---|---|
| Base URL | `https://api.hicap.ai/v1` |
| API Key | `dummy` |
| Model | A model available on your account, e.g. `claude-sonnet-4.6` |

> **Note**: The API Key field is a deliberate placeholder. Hicap authenticates with a custom `api-key` header rather than the standard `Authorization: Bearer` header, so the value here is never used. Do not put your real key in this field.

## 3. Add the Hicap Auth Header

Expand **Custom Headers** and add a single entry:

| Header Name | Header Value |
|---|---|
| `api-key` | your Hicap API key |

This is the header Hicap actually authenticates on. Without it, every request returns `401`.

The equivalent stored profile looks like this:

```json
{
  "apiProvider": "openai",
  "openAiBaseUrl": "https://api.hicap.ai/v1",
  "openAiApiKey": "dummy",
  "openAiModelId": "claude-sonnet-4.6",
  "openAiHeaders": { "api-key": "<HICAP_API_KEY>" }
}
```

> **Note**: Replace the model with one available from your Hicap account. You can view the full list at [hicap.ai/models](https://hicap.ai/models).

## 4. Verify

Save the profile and send a short prompt in the Roo Code chat panel. A reply confirms the Hicap backend is connected and routing correctly.

To confirm the endpoint independently of Roo Code, substitute your Hicap key for `$HICAP_API_KEY` or export it first:

- **Mac/Linux**:

  ```bash
  export HICAP_API_KEY="your-key-here"
  curl https://api.hicap.ai/v1/chat/completions \
    -H "api-key: $HICAP_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "model": "claude-sonnet-4.6",
      "messages": [{"role": "user", "content": "Hello"}]
    }'
  ```

- **Windows (PowerShell)**:

  ```powershell
  $env:HICAP_API_KEY = "your-key-here"
  curl.exe https://api.hicap.ai/v1/chat/completions `
    -H "api-key: $env:HICAP_API_KEY" `
    -H "Content-Type: application/json" `
    -d '{\"model\": \"claude-sonnet-4.6\", \"messages\": [{\"role\": \"user\", \"content\": \"Hello\"}]}'
  ```

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `401` missing subscription key | Custom header not saved | Re-open **Custom Headers** and confirm the row reads `api-key` exactly, lowercase |
| `401` invalid subscription key | Invalid or expired API key | Regenerate the key in the Hicap dashboard and paste it into the header value |
| `model not found` | Model not enabled on your account | Check the available models at [hicap.ai/models](https://hicap.ai/models) |
| Streaming stalls | Model does not support streaming on your plan | Disable streaming in the provider's advanced settings |
| `ECONNREFUSED` to `api.hicap.ai` | Outbound networking blocked | Ensure the host allows egress to `https://api.hicap.ai` |

---

For the latest setup instructions, see: https://docs.roocode.com/providers/openai-compatible

Questions? Reach out to [Hicap support](mailto:support@hicap.ai) or open an issue in the repository.
