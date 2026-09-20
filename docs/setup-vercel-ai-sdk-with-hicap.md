---
title: "How to Set Up the Vercel AI SDK with Hicap"
date: "2026-01-26"
author: "Hicap Engineering"
description: "A step-by-step guide for Hicap customers to configure the Vercel AI SDK against the Hicap OpenAI-compatible API."
---

# Setting Up the Vercel AI SDK with Hicap

This guide walks you through configuring the [Vercel AI SDK](https://ai-sdk.dev) to work with the Hicap API through its OpenAI-compatible endpoint.

## Prerequisites

- Node 18 or newer
- Your Hicap API key

This guide assumes a **Next.js** project, which loads `.env.local` automatically for server-side code. If you are running plain Node, see [Plain Node](#plain-node) below for how to load the environment variable.

## 1. Set the API Key

The API key should **never** be committed in plain text. Put it in `.env.local` and keep that file out of version control:

```bash
HICAP_API_KEY=your-key-here
```

Add `.env.local` to `.gitignore` if it is not already there.

## 2. Install the Provider

```bash
npm install ai @ai-sdk/openai-compatible
```

## 3. Create a Hicap Provider

Hicap authenticates with a custom `api-key` header rather than the standard `Authorization: Bearer` header. `createOpenAICompatible` lets you set headers directly and, unlike `createOpenAI`, does not require an `apiKey`:

```ts
// lib/hicap.ts
import { createOpenAICompatible } from "@ai-sdk/openai-compatible";

export const hicap = createOpenAICompatible({
  name: "hicap",
  baseURL: "https://api.hicap.ai/v1",
  headers: { "api-key": process.env.HICAP_API_KEY! },
});
```

| Field | Purpose |
|---|---|
| `name` | Provider label used in telemetry and error messages |
| `baseURL` | Hicap API endpoint |
| `headers` | The `api-key` header Hicap actually authenticates on |

Omitting `apiKey` entirely is the cleanest setup: the SDK applies `apiKey` as an `Authorization` header *before* merging `headers`, so there is no conflict either way, but leaving it out avoids sending a credential Hicap ignores.

## 4. Use It

```ts
import { generateText } from "ai";
import { hicap } from "./lib/hicap";

const { text } = await generateText({
  model: hicap("claude-sonnet-4.6"),
  prompt: "Reply with the single word: connected",
});

console.log(text);
```

Streaming works the same way:

```ts
import { streamText } from "ai";
import { hicap } from "./lib/hicap";

const result = streamText({
  model: hicap("claude-sonnet-4.6"),
  prompt: "Summarize the request lifecycle.",
});

for await (const chunk of result.textStream) {
  process.stdout.write(chunk);
}
```

> **Note**: Replace the model with one available from your Hicap account. You can view the full list at [hicap.ai/models](https://hicap.ai/models).

## 5. Verify

A reply confirms the Hicap backend is connected and routing correctly. To confirm the endpoint independently of the SDK, export the key into your shell first — `curl` does not read `.env.local`:

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

## Plain Node

Outside a framework, nothing loads `.env.local` for you, so `process.env.HICAP_API_KEY` is `undefined`. Node 20.6+ can load it natively:

```bash
node --env-file=.env.local your-script.js
```

For TypeScript or older Node, use `dotenv` and point it at the file explicitly:

```bash
npm install dotenv
```

```ts
import "dotenv/config";
```

`dotenv/config` reads `.env`, not `.env.local`. To keep using `.env.local`, configure the path:

```ts
import dotenv from "dotenv";
dotenv.config({ path: ".env.local" });
```

Load the environment **before** importing the provider module, or `process.env.HICAP_API_KEY` will still be `undefined` when `createOpenAICompatible` runs.

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `401` missing subscription key | `headers` omitted | Confirm `headers` is set on `createOpenAICompatible`, spelled `api-key` lowercase |
| `401` invalid subscription key | Env var not loaded | In Next.js, keep the provider in server-only code so `process.env.HICAP_API_KEY` resolves. In plain Node, see [Plain Node](#plain-node) |
| `undefined` API key at build time | Secret referenced in a client component | Move the provider into a server route, action, or server component |
| `undefined` API key in plain Node | `.env.local` never loaded | Run with `node --env-file=.env.local`, or load `dotenv` before importing the provider |
| `model not found` | Model not enabled on your account | Check the available models at [hicap.ai/models](https://hicap.ai/models) |
| `ECONNREFUSED` to `api.hicap.ai` | Outbound networking blocked | Ensure the host allows egress to `https://api.hicap.ai` |

---

For the latest setup instructions, see: https://ai-sdk.dev/providers/openai-compatible-providers

Questions? Reach out to [Hicap support](mailto:support@hicap.ai) or open an issue in the repository.
