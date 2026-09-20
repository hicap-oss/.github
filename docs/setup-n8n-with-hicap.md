---
title: "How to Set Up n8n with Hicap"
date: "2026-01-26"
author: "Hicap Engineering"
description: "A step-by-step guide for Hicap customers to configure n8n AI nodes against the Hicap OpenAI-compatible API."
---

# Setting Up n8n with Hicap

This guide walks you through configuring [n8n](https://n8n.io) to work with the Hicap API through its OpenAI-compatible endpoint.

There are two supported credential types. The **Azure OpenAI** credential is the cleanest match for Hicap because it sends only the `api-key` header; the **OpenAI** credential also works via its custom-header option.

## Prerequisites

- An n8n instance, cloud or self-hosted
- Your Hicap API key

## Option A: Azure OpenAI Credential (Recommended)

This credential sends exactly `api-key: <your key>` and nothing else, matching Hicap's auth scheme directly.

1. Go to **Credentials**, then **Add credential**, and choose **Azure OpenAI**.
2. Fill in:

| Field | Value |
|---|---|
| Endpoint Type | `Azure AI Foundry` |
| API Key | your Hicap API key |
| Endpoint | `https://api.hicap.ai/v1` |

3. Save, then select this credential on any **Azure OpenAI Chat Model** node and enter a model available on your account.

## Option B: OpenAI Credential with a Custom Header

Use this if you want to keep the standard OpenAI nodes. Hicap authenticates with a custom `api-key` header rather than the standard `Authorization: Bearer` header, so the base URL alone is not enough.

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

## Use It in a Workflow

Add an **OpenAI Chat Model** (Option B) or **Azure OpenAI Chat Model** (Option A) node, select your Hicap credential, and set the model.

> **Note**: Replace the model with one available from your Hicap account. You can view the full list at [hicap.ai/models](https://hicap.ai/models).

Wire the chat model into an **AI Agent** or **Basic LLM Chain** node and execute the workflow.

## Verify

Execute the node with a short prompt. A reply confirms the Hicap backend is connected and routing correctly.

To confirm the endpoint independently of n8n:

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
| `401` missing subscription key | Custom header not enabled | On the OpenAI credential, enable **Add Custom Header** and set the name to `api-key` exactly, lowercase |
| `401` invalid subscription key | Invalid or expired API key | Regenerate the key in the Hicap dashboard and update the credential |
| Credential test fails | Base URL wrong | The test hits `<Base URL>/models`, so the base URL must be `https://api.hicap.ai/v1` with no trailing slash |
| `model not found` | Model not enabled on your account | Check the available models at [hicap.ai/models](https://hicap.ai/models) |
| `ECONNREFUSED` to `api.hicap.ai` | Outbound networking blocked | On self-hosted n8n, ensure the container allows egress to `https://api.hicap.ai` |

---

For the latest setup instructions, see: https://docs.n8n.io/integrations/builtin/credentials/openai/

Questions? Reach out to [Hicap support](mailto:support@hicap.ai) or open an issue in the repository.
