---
title: "How to Set Up n8n with Hicap"
date: "2026-01-26"
author: "Hicap Engineering"
description: "A step-by-step guide for Hicap customers to configure n8n AI nodes against the Hicap OpenAI-compatible API."
---

# Setting Up n8n with Hicap

This guide walks you through configuring [n8n](https://n8n.io) to work with the Hicap API through its OpenAI-compatible endpoint.

Use the **OpenAI** credential with its custom-header option. Hicap authenticates on a custom `api-key` header, and this is the only n8n credential that can send one alongside a custom base URL.

> **Do not use the Azure OpenAI credential.** Although its stored credential defines an `api-key` header, the **Azure OpenAI Chat Model** node does not use it for Hicap-style endpoints. On the `Azure AI Foundry` endpoint type the node constructs a plain `ChatOpenAI` client, which sends only `Authorization: Bearer <key>` — Hicap returns `401 missing subscription key`. The `Classic` endpoint type does send `api-key`, but it rewrites requests to `/openai/deployments/<model>/chat/completions?api-version=...`, which Hicap does not serve and which returns `404 Resource not found`.

## Prerequisites

- An n8n instance, cloud or self-hosted
- Your Hicap API key
- **n8n `1.123.64+`, `2.29.8+`, or `2.30.1+`.** Earlier releases write custom-header credential values in plaintext into LLM node execution data, where any user with execution-data access can read your Hicap key. See [GHSA-89gh-3pgc-v5h2](https://github.com/n8n-io/n8n/security/advisories/GHSA-89gh-3pgc-v5h2). If you cannot upgrade, do not use this setup — run a LiteLLM proxy in front of Hicap instead and give n8n the proxy's own key.

## 1. Create the Credential

1. Go to **Credentials**, then **Add credential**, and choose **OpenAI**.
2. Fill in:

| Field | Value |
|---|---|
| API Key | `dummy` |
| Base URL | `https://api.hicap.ai/v1` |
| Add Custom Header | enabled |
| Header Name | `api-key` |
| Header Value | your Hicap API key |

> **Note**: The API Key field is a deliberate placeholder. n8n always sends an `Authorization` header on this credential, but Hicap validates only `api-key`, so the bearer value is inert. Do not put your real key in the API Key field.

3. Save. The credential test calls `/v1/models` on your base URL, so **Test** should pass against Hicap.

## 2. Use It in a Workflow

Add an **OpenAI Chat Model** node, select your Hicap credential, and set the model.

> **Note**: Replace the model with one available from your Hicap account. You can view the full list at [hicap.ai/models](https://hicap.ai/models).

Wire the chat model into an **AI Agent** or **Basic LLM Chain** node and execute the workflow.

## 3. Verify

Execute the node with a short prompt. A reply confirms the Hicap backend is connected and routing correctly.

To confirm the endpoint independently of n8n, substitute your Hicap key for `$HICAP_API_KEY` or export it first:

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
| `401` missing subscription key | Custom header not enabled, or you used the Azure OpenAI credential | On the OpenAI credential, enable **Add Custom Header** and set the name to `api-key` exactly, lowercase |
| `401` invalid subscription key | Invalid or expired API key | Regenerate the key in the Hicap dashboard and update the credential |
| `404` Resource not found | Azure OpenAI credential with the `Classic` endpoint type | Switch to the OpenAI credential with a custom header |
| Credential test fails | Base URL wrong | The test hits `<Base URL>/models`, so the base URL must be `https://api.hicap.ai/v1` with no trailing slash |
| `model not found` | Model not enabled on your account | Check the available models at [hicap.ai/models](https://hicap.ai/models) |
| `ECONNREFUSED` to `api.hicap.ai` | Outbound networking blocked | On self-hosted n8n, ensure the container allows egress to `https://api.hicap.ai` |

---

For the latest setup instructions, see: https://docs.n8n.io/integrations/builtin/credentials/openai/

Questions? Reach out to [Hicap support](mailto:support@hicap.ai) or open an issue in the repository.
