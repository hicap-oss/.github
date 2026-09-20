---
title: "How to Set Up LangChain with Hicap"
date: "2026-01-26"
author: "Hicap Engineering"
description: "A step-by-step guide for Hicap customers to configure LangChain in Python or JavaScript against the Hicap OpenAI-compatible API."
---

# Setting Up LangChain with Hicap

This guide walks you through configuring [LangChain](https://langchain.com) to work with the Hicap API through its OpenAI-compatible endpoint, in both Python and JavaScript.

## Prerequisites

- Python 3.9+ with `langchain-openai`, or Node 18+ with `@langchain/openai`
- Your Hicap API key

## 1. Set the API Key

The API key should **never** be committed in plain text. Supply it as an environment variable read at runtime.

- **Windows (PowerShell)**: `$env:HICAP_API_KEY = "your-key-here"`
- **Mac/Linux**: `export HICAP_API_KEY="your-key-here"`

## 2. Configure the Chat Model

Hicap authenticates with a custom `api-key` header rather than the standard `Authorization: Bearer` header, so you must set the header explicitly alongside the base URL.

### Python

```bash
pip install langchain-openai
```

```python
import os
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="claude-sonnet-4.6",
    base_url="https://api.hicap.ai/v1",
    api_key="dummy",
    default_headers={"api-key": os.environ["HICAP_API_KEY"]},
    use_responses_api=False,
)
```

| Field | Purpose |
|---|---|
| `model` | The model identifier sent to the Hicap API |
| `base_url` | Hicap API endpoint |
| `api_key` | Placeholder, required by the client and ignored by Hicap |
| `default_headers` | The `api-key` header Hicap actually authenticates on |
| `use_responses_api` | Pins the client to `/v1/chat/completions` |

> **Note**: Keep `use_responses_api=False`. When it is unset, Python `ChatOpenAI` can infer the Responses API from the model name and call `/v1/responses`, which Hicap serves only as a developer preview. This guide is validated against the production `/v1/chat/completions` path.

### JavaScript / TypeScript

```bash
npm install @langchain/openai
```

```ts
import { ChatOpenAI } from "@langchain/openai";

const llm = new ChatOpenAI({
  model: "claude-sonnet-4.6",
  apiKey: "dummy",
  configuration: {
    baseURL: "https://api.hicap.ai/v1",
    defaultHeaders: { "api-key": process.env.HICAP_API_KEY! },
  },
});
```

In the JS package, `baseURL` and `defaultHeaders` are **not** top-level fields. They belong inside `configuration`, which is passed through to the underlying OpenAI client.

> **Note**: `api_key` / `apiKey` is a deliberate placeholder. Hicap validates only the `api-key` header, so the value here is never used. Do not put your real key in this field.

> **Note**: Replace the model with one available from your Hicap account. You can view the full list at [hicap.ai/models](https://hicap.ai/models).

## 3. Verify

### Python

```python
print(llm.invoke("Reply with the single word: connected").content)
```

### JavaScript / TypeScript

```ts
const res = await llm.invoke("Reply with the single word: connected");
console.log(res.content);
```

A reply confirms the Hicap backend is connected and routing correctly. To confirm the endpoint independently of LangChain:

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
| `401` missing subscription key | Header not sent | Python: use `default_headers`. JS: nest `defaultHeaders` inside `configuration`, not at the top level |
| `401` invalid subscription key | Invalid or expired API key | Regenerate the key in the Hicap dashboard and re-export `HICAP_API_KEY` |
| Requests go to `api.openai.com` | Base URL not applied | Python: `base_url`. JS: `configuration.baseURL` |
| `model not found` | Model not enabled on your account | Check the available models at [hicap.ai/models](https://hicap.ai/models) |
| Unsupported-parameter errors | `ChatOpenAI` targets official OpenAI specs | Remove OpenAI-only parameters that the routed model does not accept |
| `ECONNREFUSED` to `api.hicap.ai` | Outbound networking blocked | Ensure the host allows egress to `https://api.hicap.ai` |

---

For the latest setup instructions, see: https://python.langchain.com/docs/integrations/chat/openai/

Questions? Reach out to [Hicap support](mailto:support@hicap.ai) or open an issue in the repository.
