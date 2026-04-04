---
title: "How to Set Up LiteLLM with Hicap"
date: "2026-04-04"
author: "Hicap Engineering"
description: "A step-by-step guide for Hicap customers to configure LiteLLM with the Hicap API."
---

# Setting Up LiteLLM with Hicap

This guide walks you through configuring LiteLLM to work with the Hicap API. In this setup, Docker Compose runs LiteLLM and PostgreSQL locally, LiteLLM routes upstream to Hicap's OpenAI-compatible API, and the app uses canonical provider-qualified aliases such as `anthropic/claude-opus-4.6`.

## Prerequisites

- Docker Desktop with `docker compose`
- Node.js 20+
- A valid Hicap API key

## 1. Configure Environment Variables

Create a `.env` file at the repo root and set the required Hicap and LiteLLM values:

```env
HICAP_API_BASE=https://api.hicap.ai/v1
HICAP_API_KEY=your-hicap-key
LITELLM_MASTER_KEY=sk-local-litellm
DATABASE_URL=postgresql://litellm:litellm@db:5432/litellm
```

Notes:

- `HICAP_API_BASE` points LiteLLM at Hicap's OpenAI-compatible endpoint.
- `HICAP_API_KEY` is forwarded upstream in the `api-key` header.
- `LITELLM_MASTER_KEY` is what your app or local tests use to authenticate to LiteLLM.

## 2. Mount LiteLLM Through Docker Compose

This repo's `docker-compose.yml` mounts the LiteLLM config and override layer into the container:

```yaml
services:
  litellm:
    image: ghcr.io/berriai/litellm-database:main-latest
    env_file:
      - .env
    environment:
      DATABASE_URL: ${DATABASE_URL:-postgresql://litellm:litellm@db:5432/litellm}
      HICAP_API_BASE: ${HICAP_API_BASE:-https://api.hicap.ai/v1}
      HICAP_API_KEY: ${HICAP_API_KEY:-replace-me}
      LITELLM_MASTER_KEY: ${LITELLM_MASTER_KEY:-sk-local-litellm}
      PYTHONPATH: /app/overrides
    command: ["--config", "/app/config.yaml", "--port", "4000", "--detailed_debug"]
    volumes:
      - ./litellm_config.yaml:/app/config.yaml:ro
      - ./docker/litellm-overrides:/app/overrides:ro
```

Why this matters:

- `litellm_config.yaml` defines the model routing table.
- `docker/litellm-overrides` contains the runtime override layer used in this repo for canonical naming and Hicap transport fixes.

## 3. Define Explicit Hicap Model Aliases

Use explicit provider-qualified aliases in `litellm_config.yaml` instead of relying on wildcard passthrough if you want stable naming, provider metadata, and cleaner UI behavior.

Example:

```yaml
model_list:
  - model_name: anthropic/claude-opus-4.6
    litellm_params:
      model: anthropic/claude-opus-4.6
      api_base: os.environ/HICAP_API_BASE
      api_key: os.environ/HICAP_API_KEY
      extra_headers:
        api-key: os.environ/HICAP_API_KEY
    model_info:
      provider: Anthropic
      display_name: Claude Opus 4.6
      max_input_tokens: 200000
```

Recommended pattern:

- Keep `model_name` canonical and provider-qualified.
- Keep `litellm_params.model` canonical as well.
- Add `model_info` where useful for provider labels, display names, token limits, and pricing.

## 4. Understand the Hicap Transport Detail

One important detail in this repo is that canonical aliases are preserved for UI and local app behavior, but some Hicap-backed non-OpenAI families still need OpenAI-style wire model names at transport time.

In this repository, that is handled by the override layer mounted from `docker/litellm-overrides/sitecustomize.py`.

That override does two separate jobs:

- It preserves canonical names in UI-facing and app-facing metadata.
- It rewrites the temporary outbound LiteLLM deployment to the routed upstream model only when making the actual Hicap request.

That separation is the reason canonical names work locally without breaking Hicap's OpenAI-compatible upstream call path.

## 5. Start the Stack

Install dependencies and start the local services:

```bash
npm install
npm run dev:up
npm run app:dev
```

That gives you:

- LiteLLM on `http://127.0.0.1:4000`
- The Next.js workbench on `http://127.0.0.1:3000`

## 6. Verify LiteLLM Can See the Models

Check that LiteLLM is up and serving the aliases you configured:

```bash
curl -H "Authorization: Bearer sk-local-litellm" http://127.0.0.1:4000/models
```

If this returns no models or errors:

- confirm `.env` was loaded
- confirm `HICAP_API_KEY` is valid
- confirm `litellm_config.yaml` is mounted into `/app/config.yaml`

## 7. Verify a Chat Completion Through LiteLLM

Run a chat completion through LiteLLM using one of the configured aliases:

```bash
curl -X POST http://127.0.0.1:4000/v1/chat/completions \
  -H "Authorization: Bearer sk-local-litellm" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "anthropic/claude-opus-4.6",
    "messages": [{"role": "user", "content": "Reply with ok."}],
    "max_tokens": 16
  }'
```

If the request succeeds, the LiteLLM-to-Hicap path is working.

## 8. Optional: Use the Local Workbench

The sample app in this repo provides a useful verification surface:

- browse models from LiteLLM and Hicap
- send normal or streaming chat requests
- inspect the last request and response payloads
- confirm model naming, usage metadata, and provider display behavior

Open:

- `http://127.0.0.1:3000`
- `http://127.0.0.1:3000/docs`

## Operational Notes

### Auth layers

- Your local app authenticates to LiteLLM with `LITELLM_MASTER_KEY`.
- LiteLLM authenticates upstream to Hicap with `HICAP_API_KEY`.

### Canonical naming

- Public model names, local app model names, and LiteLLM-facing model names are best kept canonical and provider-qualified.
- In this repo, those names are intentionally preserved for UI correctness.

### Metadata

- Hicap's published model catalog is the authoritative source for display metadata such as provider, context window, and pricing.
- LiteLLM's own internal pricing/model-cost map is not sufficient for all Hicap-backed aliases.

### Usage analytics

- LiteLLM admin surfaces do not all use the same backend fields.
- Model info, request details, spend logs, and daily activity may each need separate normalization if you want canonical provider names everywhere.

## Troubleshooting

### `/models` is empty or incomplete

- Check `HICAP_API_KEY`.
- Check that the container has the right `.env` values.
- Check `docker compose logs litellm`.

### Chat requests fail for non-OpenAI model families

- Check that the alias exists in `litellm_config.yaml`.
- Check that the runtime override layer is mounted through `docker-compose.yml`.
- Check whether the upstream call path still needs routed OpenAI-style wire model names.

### LiteLLM UI shows stale provider or spend data

- Clear historical usage data if you are validating new naming behavior.
- Disable caching on the affected endpoints if the browser is holding stale responses.

### The sample app looks correct but LiteLLM admin UI does not

- Inspect the LiteLLM endpoints directly.
- Different admin UI surfaces can consume different backend routes and may need separate fixes.

---

For the latest setup instructions, see: https://github.com/BerriAI/litellm

Questions? Reach out to [Hicap support](mailto:support@hicap.ai) or open an issue in the repository.