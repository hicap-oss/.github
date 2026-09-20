# Repository Instructions

## Pricing Source Of Truth

When updating base pricing in `src/features/models/config/models-catalog.json`, use these upstream pricing pages as the canonical sources for base list pricing:

- OpenAI: https://developers.openai.com/api/docs/pricing
- Anthropic: https://platform.claude.com/docs/en/about-claude/pricing
- Gemini: https://ai.google.dev/gemini-api/docs/pricing

Rules:

- Update base pricing in `models-catalog.json`, not discounted pricing copied from downstream systems. The canonical source lives in `hicap-oss/.github` at `data/models-catalog.json`; the WIT copy is a synced build artifact for the site.
- When pricing changes are made in `models-catalog.json`, regenerate `src/features/models/config/models-pricing.json` so automated agents and the `/models.json` route stay in sync with the shared catalog source of truth.
- Use `pnpm generate:models-pricing` to regenerate `src/features/models/config/models-pricing.json` after pricing-related edits.
- Treat `promotions.ts` as the place where WIT-specific discounts are applied. Do not bake promotional discounts directly into `models-catalog.json`.
- If a downstream system such as TAB shows discounted pricing, verify that `base price * active promotion` matches the downstream value before changing the base price.
- Keep model IDs aligned with the object key or canonical model identifier being compared.
- If an upstream page does not list an optional field such as `cacheWrite`, `longContextInput`, `longContextOutput`, or `longContextCache`, remove or leave absent that optional field instead of inventing a value.
- For Gemini pricing, use the Gemini Developer API pricing page above, not Vertex AI pricing, unless a task explicitly asks for Vertex pricing.

## Pricing Extraction And Matching Rules

- Always match token pricing for the exact model ID or its explicit canonical alias. Do not substitute a newer model, successor model, or similarly named model unless the task explicitly asks for migration guidance rather than pricing verification.
- Use base list pricing only. Ignore downstream discounts, enterprise discounts, promotional discounts, regional uplifts, batch discounts, flex pricing, priority pricing, and storage-hour pricing unless the task explicitly asks for those values.
- Prefer text token pricing for `models-catalog.json`. Do not accidentally use audio, speech, image-per-image, video, tool-call, web-search, storage, or container pricing when the catalog entry is a text-token-priced model.
- For multimodal models that list separate text, image, video, and audio token prices, use the text token rate unless the WIT model entry is clearly an audio-only, speech-only, embedding-only, or image-priced model.
- Ignore `Free`, `Free of charge`, `Not available`, and zero-priced starter-tier values when selecting canonical paid base pricing.
- When a page includes multiple billing modes, use the standard on-demand paid list rate for the provider's primary API unless the task explicitly asks for batch, flex, priority, regional, or enterprise pricing.
- Match only values that are expressed in the same unit as WIT expects for token-priced models: dollars per 1M tokens or the provider's equivalent MTok notation.
- If a page section is noisy or the extracted text is ambiguous, verify the model heading first, then match only the pricing row that belongs to that section. Do not combine values from neighboring model sections.

## Field Mapping Rules

- `input` means the normal paid input token price for prompts at or below the provider's standard threshold.
- `output` means the normal paid output token price for responses at or below the provider's standard threshold.
- `cache` means cached-input or cache-read pricing, not cache-write pricing.
- `cacheWrite` means cache-write pricing only when the upstream provider explicitly publishes it.
- `longContextThreshold` should be set only when the provider explicitly publishes a higher long-context threshold such as `> 200k` or `> 272k` tokens.
- `longContextInput`, `longContextOutput`, and `longContextCache` should only be set when the provider explicitly publishes higher token pricing above that threshold.
- If a provider supports a large context window but charges standard rates across the full window, do not add long-context premium fields.
- Never infer missing optional fields from multipliers, related tables, or neighboring models unless the upstream page explicitly states the applicable value for that exact model.

## Provider-Specific Matching Rules

### OpenAI

- Use the OpenAI pricing page's standard text-token pricing row for the exact model.
- OpenAI text pricing tables are ordered as `Input`, `Cached Input`, then `Output` for the standard tier. Map them exactly to `input`, `cache`, and `output`.
- Ignore Batch, Flex, and Priority columns unless the task explicitly asks for them.
- For models that publish a higher-rate long-context tier, keep the normal below-threshold values in `input` and `output`, and map the above-threshold values to `longContextInput` and `longContextOutput`.
- Set `longContextThreshold` only when the page explicitly gives the threshold, such as `272K input tokens`.
- Do not confuse built-in tool pricing, audio pricing, image pricing, or fine-tuning pricing with text-token inference pricing.

### Anthropic

- Use the Anthropic Claude API pricing table, not Bedrock, Vertex AI, Foundry, regional endpoint, batch, fast mode, or data residency pricing unless the task explicitly asks for those variants.
- Anthropic model pricing rows should be mapped as: base input price -> `input`, cache read / cache hit price -> `cache`, output price -> `output`.
- When Anthropic lists both 5-minute and 1-hour cache writes, map `cacheWrite` to the 5-minute cache write value unless the repository is explicitly changed to support separate cache-write durations.
- Do not derive `cacheWrite` from the multiplier table if the model row already provides an explicit cache-write value.
- For Claude Sonnet models that explicitly publish premium long-context pricing above 200k input tokens, map the normal row to `input` / `output` / `cache`, then map the premium row to `longContextInput` / `longContextOutput` / `longContextCache` and set `longContextThreshold` to `200000`.
- For Claude models that explicitly state the full 1M context window is billed at standard rates, do not add long-context premium fields.

### Gemini

- Use the Gemini Developer API pricing page, not Vertex AI pricing, unless the task explicitly asks for Vertex.
- Use the paid Gemini API token rates, not free-tier values, not batch-discount values, and not enterprise / Vertex references.
- Gemini pages often show separate prices for text/image/video versus audio. For WIT token-priced Gemini catalog entries, use the text/image/video token rate and ignore the audio rate unless the model entry is explicitly audio-only.
- When Gemini shows `Input price`, `Output price`, and `Context caching price`, map them to `input`, `output`, and `cache` respectively.
- Ignore `Context caching (storage)` or per-hour storage rows because `models-catalog.json` does not track storage pricing.
- When Gemini shows two token tiers such as `<= 200k tokens` and `> 200k tokens`, map the lower tier to `input` / `output` / `cache`, map the higher tier to `longContextInput` / `longContextOutput` / `longContextCache`, and set `longContextThreshold` to `200000`.
- When a Gemini model is described as image-capable but the WIT entry is still token-priced in `per 1M tokens`, use the token-priced text output rate, not per-image equivalent pricing.

## Verification Checklist

- Confirm the matched upstream section belongs to the exact model ID being updated.
- Confirm the selected values are paid text-token list prices, not audio or image-equivalent prices unless the model entry requires those.
- Confirm `input`, `cache`, `cacheWrite`, and `output` are mapped to the correct columns and not shifted one column to the left or right.
- Confirm long-context fields are only present when the upstream page explicitly publishes a higher-rate threshold for that exact model.
- Confirm any downstream comparison against TAB or another system uses `base price * active promotion` before concluding that WIT base pricing is wrong.

## Pricing Audit Workflow

The pricing audit workflow must use the same pricing pages above when checking `models-catalog.json`.

## Agent Pricing Snapshot

- `src/features/models/config/models-pricing.json` is the machine-readable pricing snapshot that backs the public `/models.json` feed for external agents visiting the site.
- `src/features/models/config/models-catalog.json` is the WIT-local synced copy of the shared source catalog. The canonical source is `hicap-oss/.github:data/models-catalog.json`.
- `src/features/models/config/models-data.ts` only provides TypeScript types and a typed export for the synced catalog JSON.
- `/models.json` is the public machine-readable contract for external agents and must serve the contents of `src/features/models/config/models-pricing.json` without applying promotions.
- Keep `/models.json` stable and agent-friendly: use JSON, preserve model IDs, avoid presentation-only fields, avoid internal source-file metadata, and expose base pricing rather than promo-adjusted pricing.
- If pricing fields, model IDs, or provider mappings change in `models-catalog.json`, update the generated pricing snapshot in the same change.
