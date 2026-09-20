# Hicap OSS

**The control plane for AI.** One API in front of every model and provider, with intelligent routing, full observability, and spend you can govern to the dollar.

[![Website](https://img.shields.io/badge/Website-hicap.ai-0F0F0E?style=flat-square)](https://hicap.ai)
[![Docs](https://img.shields.io/badge/Docs-docs.hicap.ai-0F0F0E?style=flat-square)](https://docs.hicap.ai)
[![Platform](https://img.shields.io/badge/Platform-Sign%20in-0F0F0E?style=flat-square)](https://platform.hicap.ai)

This organization is home to the open-source projects, examples, and tooling built around the [Hicap](https://hicap.ai) platform.

## What Hicap does

Hicap sits in the path of every AI request your company makes, so you can:

- **See it** - every request logged and tagged by team, feature, and key, with the model that actually answered.
- **Govern it** - budgets, caps, and model allow-lists enforced at the gateway, not in your app code.
- **Optimize it** - route to the cheapest model that clears your quality bar, with failover the moment a provider wobbles.
- **Secure it** - scoped, rotatable keys and a full audit trail behind one base URL, with no rewrites.

## Drop-in integration

Hicap is a drop-in replacement for the OpenAI API. Point your existing client at our base URL and reach every major model:

```javascript
import OpenAI from "openai";

// Set once per service so every request carries the same baseline attribution.
const defaultTags = {
  env: process.env.NODE_ENV === "production" ? "production" : "development",
  system: "checkout-api",
  channel: "web",
};

const client = new OpenAI({
  baseURL: "https://api.hicap.ai/v1",
  apiKey: process.env.HICAP_API_KEY,
  defaultHeaders: {
    "api-key": process.env.HICAP_API_KEY,
    "x-hicap-tags": JSON.stringify(defaultTags),
  },
});

// Add per-call dimensions to pinpoint what a specific request was doing.
const response = await client.chat.completions.create(
  {
    model: "gpt-5",
    messages: [{ role: "user", content: "Hello!" }],
  },
  {
    headers: {
      "x-hicap-tags": JSON.stringify({ ...defaultTags, feature: "product-recommendations" }),
    },
  },
);
```

Swap providers with a single parameter change - OpenAI, Anthropic, Google Gemini, Moonshot, Zhipu, and MiniMax all share the same endpoint and SDK.

The `x-hicap-tags` header is how spend and usage get sliced. Set the stable dimensions (`env`, `system`, `channel`) once on the client, then add per-call dimensions like `feature` or `agent` where they matter. Requests that arrive without them land in **Unattributed Traffic**.

See the [Developer Quickstart](https://docs.hicap.ai/quickstart) to make your first request, [Tags, Dimensions & Segments](https://docs.hicap.ai/concepts/tags-dimensions-segments) for the full attribution model, or browse [all supported models](https://hicap.ai/models).

## Projects

| Repository | Description |
| --- | --- |
| [docs](https://github.com/hicap-oss/docs) | Hicap documentation service, published to [docs.hicap.ai](https://docs.hicap.ai) |
| [realtime-voice](https://github.com/hicap-oss/realtime-voice) | Minimal web app exercising GPT Realtime voice through the Hicap OpenAI-compatible API |
| [VercelSDK-chatTest](https://github.com/hicap-oss/VercelSDK-chatTest) | Vercel AI SDK v5 testing application |
| [thinking-test](https://github.com/hicap-oss/thinking-test) | Example of Claude extended thinking over Hicap |
| [brew-status](https://github.com/hicap-oss/brew-status) | Windows tray app for monitoring Claude Code usage, rate limits, and session stats in real time |

We also maintain forks of community tools we build on, including [cline](https://github.com/hicap-oss/cline), [claude-code-router](https://github.com/hicap-oss/claude-code-router), [openclaude](https://github.com/hicap-oss/openclaude), and [paperclip](https://github.com/hicap-oss/paperclip).

[Browse all repositories](https://github.com/orgs/hicap-oss/repositories)

## Learn more

- [LLM Gateway](https://hicap.ai/gateway) - access, routing, and reliability behind one endpoint
- [Voice Gateway](https://hicap.ai/voice) - speech, narration, and production transcription from one route
- [How it works](https://hicap.ai/how-it-works) - the whole integration, end to end
- [Integrations](https://hicap.ai/integrations) and [Enterprise features](https://hicap.ai/enterprise)
- [API reference](https://docs.hicap.ai/api-reference/introduction) and [FAQ](https://docs.hicap.ai/faq)

## Contributing

We welcome contributions. Start with our [Contributing Guidelines](https://github.com/hicap-oss/.github/blob/main/CONTRIBUTING.md) and [Code of Conduct](https://github.com/hicap-oss/.github/blob/main/CODE_OF_CONDUCT.md).

## Get in touch

- Need help? See our [Support guide](https://github.com/hicap-oss/.github/blob/main/SUPPORT.md)
- Found a vulnerability? Follow our [Security Policy](https://github.com/hicap-oss/.github/blob/main/SECURITY.md)
- Questions or partnerships: [contact@hicap.ai](mailto:contact@hicap.ai)
- Want to see it on your own stack? [Book a demo](https://hicap.ai/talk-to-us)

## License

Our projects are released under various open-source licenses. Check the `LICENSE` file in each repository for details.
