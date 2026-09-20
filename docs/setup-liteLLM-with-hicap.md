---
title: "How to Set Up LiteLLM with Hicap"
date: "2026-04-06"
author: "Hicap Engineering"
description: "A step-by-step guide for Hicap customers to configure LiteLLM with the Hicap API."
---

# Setting Up LiteLLM with Hicap

This guide walks through configuring LiteLLM to work with the Hicap API. In this setup:

- Docker Compose runs LiteLLM and PostgreSQL locally.
- LiteLLM routes all requests upstream to Hicap's OpenAI-compatible API.
- Models are registered with canonical provider-qualified names such as `anthropic/claude-opus-4.6` or `google/gemini-2.5-pro`.
- A runtime override layer preserves those canonical names across UI, analytics, and spend tracking.

## Prerequisites

- Docker Desktop with `docker compose`
- Node.js 20+
- A valid Hicap API key

---

## 1. Configure Environment Variables

Create a `.env` file at the repo root:

```env
HICAP_API_BASE=https://api.hicap.ai/v1
HICAP_API_KEY=your-hicap-key
LITELLM_MASTER_KEY=sk-local-litellm
DATABASE_URL=postgresql://litellm:litellm@db:5432/litellm
```

| Variable | Purpose |
|---|---|
| `HICAP_API_BASE` | Points LiteLLM at Hicap's OpenAI-compatible endpoint |
| `HICAP_API_KEY` | Forwarded upstream in the `api-key` header Hicap expects |
| `LITELLM_MASTER_KEY` | Used by your app or tests to authenticate to LiteLLM |
| `DATABASE_URL` | PostgreSQL connection for virtual keys, rate limits, and spend tracking |

---

## 2. Docker Compose

The `docker-compose.yml` runs LiteLLM and PostgreSQL and mounts two local files into the container:

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

| Mount | What it does |
|---|---|
| `litellm_config.yaml → /app/config.yaml` | Defines the model routing table and general settings |
| `docker/litellm-overrides → /app/overrides` | Runtime override layer for canonical naming and Hicap transport (see [Section 5](#5-provider-setup-the-runtime-override-layer)) |

`PYTHONPATH: /app/overrides` causes Python to execute `sitecustomize.py` from that directory on every LiteLLM startup, which applies the override patches automatically.

---

## 3. Model Catalog

All models are declared in `litellm_config.yaml` under `model_list`. Each entry maps a canonical alias to an upstream Hicap route.

### Naming convention

Use provider-qualified names for both `model_name` and `litellm_params.model`:

```yaml
model_list:
  - model_name: anthropic/claude-opus-4.6   # what your app and UI use
    litellm_params:
      model: anthropic/claude-opus-4.6       # kept canonical; override layer handles wire rewrite
      api_base: os.environ/HICAP_API_BASE
      api_key: os.environ/HICAP_API_KEY
      extra_headers:
        api-key: os.environ/HICAP_API_KEY    # Hicap expects this header in addition to Authorization
```

The `extra_headers.api-key` field is required for every entry. Hicap's API validates the key from this header.

### OpenAI family models

OpenAI-family models (`openai/gpt-*`) require no `model_info` block. LiteLLM's internal pricing map already covers the canonical `openai/*` names, and Hicap routes these through its OpenAI-compatible path without any wire-name rewriting.

```yaml
  - model_name: openai/gpt-4o
    litellm_params:
      model: openai/gpt-4o
      api_base: os.environ/HICAP_API_BASE
      api_key: os.environ/HICAP_API_KEY
      extra_headers:
        api-key: os.environ/HICAP_API_KEY

  - model_name: openai/gpt-4.1
    litellm_params:
      model: openai/gpt-4.1
      api_base: os.environ/HICAP_API_BASE
      api_key: os.environ/HICAP_API_KEY
      extra_headers:
        api-key: os.environ/HICAP_API_KEY
```

### Non-OpenAI family models

Non-OpenAI families — Anthropic, Google, MiniMax, Moonshot, Zhipu — require an explicit `model_info` block. LiteLLM's internal pricing map does not cover Hicap-backed aliases for these providers. Without `model_info`, cost tracking will be missing or incorrect in the admin UI and spend logs.

Additionally, at transport time, the runtime override layer rewrites non-OpenAI canonical names to `openai/<suffix>` for the actual Hicap upstream call, then restores canonical names in all metadata responses. This is transparent to your app and the LiteLLM UI.

```yaml
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
      input_cost_per_token: 0.000005
      output_cost_per_token: 0.000025
      cache_read_input_token_cost: 0.0000005
      cache_creation_input_token_cost: 0.00000625
```

---

## 4. Cost Layer

Cost metadata lives in the `model_info` block of each non-OpenAI entry. All values are per token in USD.

### Standard cost fields

| Field | What it covers |
|---|---|
| `input_cost_per_token` | Standard prompt token cost |
| `output_cost_per_token` | Standard completion token cost |
| `cache_read_input_token_cost` | Cost for tokens served from the provider's prompt cache |
| `cache_creation_input_token_cost` | Cost for tokens that write to the provider's prompt cache |

### Extended context pricing

Anthropic and Google models have tiered pricing that changes above certain token thresholds. Use the `_above_200k_tokens` variants when the model supports long-context pricing:

| Field | When to use |
|---|---|
| `input_cost_per_token_above_200k_tokens` | Anthropic models with 200k+ context pricing |
| `output_cost_per_token_above_200k_tokens` | Anthropic models with 200k+ context pricing |
| `cache_read_input_token_cost_above_200k_tokens` | Anthropic cache read at 200k+ tier |
| `input_cost_per_token_above_200k_tokens` | Google Gemini Pro models with long-context pricing |
| `output_cost_per_token_above_200k_tokens` | Google Gemini Pro models with long-context pricing |
| `cache_read_input_token_cost_above_200k_tokens` | Google Gemini cache read at long-context tier |

Example for a model with tiered Anthropic pricing:

```yaml
    model_info:
      provider: Anthropic
      display_name: Claude Sonnet 4.5
      max_input_tokens: 200000
      input_cost_per_token: 0.000003
      output_cost_per_token: 0.000015
      cache_read_input_token_cost: 0.0000003
      cache_creation_input_token_cost: 0.00000375
      input_cost_per_token_above_200k_tokens: 0.000006
      output_cost_per_token_above_200k_tokens: 0.0000225
      cache_read_input_token_cost_above_200k_tokens: 0.0000006
```

Example for a Google model with long-context pricing:

```yaml
    model_info:
      provider: Google
      display_name: Gemini 2.5 Pro
      max_input_tokens: 1000000
      input_cost_per_token: 0.00000125
      output_cost_per_token: 0.00001
      cache_read_input_token_cost: 0.000000125
      input_cost_per_token_above_200k_tokens: 0.0000025
      output_cost_per_token_above_200k_tokens: 0.000015
      cache_read_input_token_cost_above_200k_tokens: 0.00000025
```

Hicap's published model catalog is the authoritative source for all pricing. Do not rely on LiteLLM's internal cost map for Hicap-backed non-OpenAI aliases.

---

## 5. Provider Setup: The Runtime Override Layer

Hicap uses an OpenAI-compatible wire protocol, but non-OpenAI model families (Anthropic, Google, etc.) must be sent with `openai/<model-suffix>` wire names in the actual HTTP request. At the same time, you want canonical names like `anthropic/claude-opus-4.6` to appear everywhere — in your app, in the LiteLLM UI, and in spend logs.

The override layer at `docker/litellm-overrides/sitecustomize.py` handles this separation. It runs automatically on LiteLLM startup via `PYTHONPATH` and applies five patches:

### Request routing patch

Intercepts LiteLLM's router at deployment selection time. For non-OpenAI canonical models, it rewrites the outbound `model` parameter to `openai/<suffix>` for the actual Hicap upstream call, leaving all metadata and response handling using the canonical name.

Only models with unambiguous suffix mappings are rewritten — if the same suffix appears under multiple providers, no rewrite is applied.

### Model info display patch

Patches the three LiteLLM proxy endpoints that serve model metadata (`_get_proxy_model_info`, `_enrich_model_info_with_litellm_data`, `_get_model_group_info`). Ensures that model info responses and the admin UI always show canonical provider-qualified names and provider labels, not `openai` placeholders.

### Spend log display patch

Patches the spend log response builder so that the model name shown in the LiteLLM UI's spend log table is the canonical alias, not the wire model name used at request time.

### Provider breakdown patch

Patches `update_breakdown_metrics` in the daily activity endpoints. Without this, the provider breakdown in usage analytics shows `openai` for all non-OpenAI models. The patch derives the canonical provider from the model name and corrects the breakdown record before aggregation.

### Cache header patch

Adds an HTTP middleware that sets `Cache-Control: no-store` on model metadata and spend log endpoints. This prevents stale model names or spend data from being held in the browser after you update the config or clear usage data.

---

## 6. Start the Stack

```bash
npm install
npm run dev:up      # starts LiteLLM + PostgreSQL, waits for proxy health
npm run app:dev     # starts the Next.js workbench
```

This gives you:

- LiteLLM proxy on `http://127.0.0.1:4000`
- LiteLLM admin UI on `http://127.0.0.1:4000/ui`
- Next.js workbench on `http://127.0.0.1:3000`

---

## 7. Verify LiteLLM Can See the Models

```bash
curl -H "Authorization: Bearer sk-local-litellm" http://127.0.0.1:4000/models
```

If this returns no models or errors:

- Confirm `.env` was loaded by the container
- Confirm `HICAP_API_KEY` is valid
- Confirm `litellm_config.yaml` is mounted at `/app/config.yaml`
- Check container logs: `npm run proxy:logs`

---

## 8. Verify a Chat Completion

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

A successful response confirms the full LiteLLM → Hicap path is working, including the canonical-to-wire model name rewrite.

---

## 9. Optional: Use the Local Workbench

The Next.js app at `http://127.0.0.1:3000` provides a useful verification surface:

- Browse and search models from LiteLLM and Hicap
- Send normal or streaming chat requests
- Inspect last request and response payloads
- Confirm model naming, usage metadata, and provider display behavior

---

## Operational Notes

### Auth layers

- Your app authenticates to LiteLLM with `LITELLM_MASTER_KEY`.
- LiteLLM authenticates upstream to Hicap with `HICAP_API_KEY`, forwarded in the `api-key` header.

### Adding a new model

1. Add the entry to `litellm_config.yaml` with a canonical `model_name`.
2. Include `model_info` with cost fields if the model is non-OpenAI family.
3. Restart LiteLLM: `docker compose restart litellm`.
4. Verify with `npm run proxy:models`.

### Updating cost data

Costs change as providers update pricing. To update:

1. Edit the relevant `model_info` cost fields in `litellm_config.yaml`.
2. Restart LiteLLM.
3. Clear any browser-cached model metadata if the admin UI shows stale values.

---

## Troubleshooting

### `/models` is empty or incomplete

- Check `HICAP_API_KEY` in the running container: `docker compose exec litellm env | grep HICAP`.
- Check `docker compose logs litellm` for config parse errors.
- Confirm the config file is mounted: `docker compose exec litellm cat /app/config.yaml`.

### Chat requests fail for non-OpenAI model families

- Confirm the alias exists in `litellm_config.yaml`.
- Confirm the runtime override layer is mounted: `docker compose exec litellm ls /app/overrides/`.
- Check LiteLLM logs for routing or upstream errors.

### Provider column shows "openai" in usage analytics

- Confirm the override layer is mounted and `PYTHONPATH=/app/overrides` is set.
- Check that the provider breakdown patch applied: look for `Applied Hicap provider breakdown patch` in the container startup logs.
- Clear historical usage data if you are validating new config — old records retain their original provider values.

### LiteLLM UI shows stale model names or spend data

- The cache header patch disables browser caching on metadata endpoints, but hard-refresh (`Ctrl+Shift+R`) if stale data persists.
- Different admin UI surfaces consume different backend routes. Inspect LiteLLM API responses directly if the UI does not match.

### Spend log shows wire model names instead of canonical names

- Confirm the spend log alias patch applied: look for `Applied Hicap spend-log alias display patch` in container startup logs.
- The patch applies at response build time, so only new spend log requests benefit — existing cached browser responses may show old names until a hard-refresh.

---

For the latest LiteLLM documentation, see: https://github.com/BerriAI/litellm

Questions? Reach out to [Hicap support](mailto:support@hicap.ai) or open an issue in the repository.
