---
title: "How to Set Up Claude Code Router with Hicap"
date: "2026-01-26"
author: "Hicap Engineering"
description: "A step-by-step guide for Hicap customers to integrate Claude Code Router with the Hicap API."
---

# Setting Up Claude Code Router with Hicap

This guide walks you through configuring [Claude Code Router](https://github.com/musistudio/claude-code-router) to work with the Hicap API for multi-model, multi-provider AI development.

## Prerequisites

- Node.js (v20+) installed
- Your Hicap API key (set as an environment variable):
  - **Windows (PowerShell)**: `$env:HICAP_API_KEY = "your-key-here"`
  - **Mac/Linux**: `export HICAP_API_KEY="your-key-here"`

## Installation

```shell
npm install -g @anthropic-ai/claude-code
npm install -g @musistudio/claude-code-router
```

This installs both `claude` and `ccr` CLI tools globally.

## Configuration

Create or edit `C:\Users\<username>\.claude-code-router\config.json`:

> **Mac**: Use `~/.claude-code-router/config.json`

```json
{
  "LOG": true,
  "LOG_LEVEL": "debug",
  "HOST": "127.0.0.1",
  "PORT": 3456,
  "APIKEY": "$HICAP_API_KEY",
  "API_TIMEOUT_MS": "600000",
  "transformers": [
    {
      "path": "C:\\Users\\<username>\\.claude-code-router\\hicap-transformer.js"
    }
  ],
  "Providers": [
    {
      "name": "hicap",
      "api_base_url": "https://api.hicap.ai/v1/chat/completions",
      "api_key": "$HICAP_API_KEY",
      "models": ["<model-1>", "<model-2>", "<model-3>"],
      "transformer": { "use": ["hicap"] }
    }
  ],
  "Router": {
    "default": "hicap,<default-model>",
    "background": "hicap,<background-model>",
    "think": "hicap,<think-model>",
    "longContext": "hicap,<long-context-model>",
    "longContextThreshold": 60000,
    "webSearch": "hicap,<web-search-model>",
    "image": ""
  }
}
```

> **Note**: Replace the model placeholders with models available from your Hicap account. You can view the full list at [hicap.ai/models](https://hicap.ai/models).

### Configuration Options

| Option           | Description                                             |
| ---------------- | ------------------------------------------------------- |
| `APIKEY`         | Your Hicap API key (supports `$ENV_VAR` interpolation)  |
| `API_TIMEOUT_MS` | Request timeout in milliseconds                         |
| `transformers`   | Array of custom transformer paths                       |
| `Providers`      | Provider configurations with name, URL, key, and models |
| `Router`         | Model routing rules for different task types            |

### Router Options

| Route                  | Description                                       |
| ---------------------- | ------------------------------------------------- |
| `default`              | Model used for general tasks                      |
| `background`           | Model used for background/smaller tasks           |
| `think`                | Model used for reasoning-heavy tasks              |
| `longContext`          | Model used for large context handling             |
| `longContextThreshold` | Token count that triggers `longContext` (default: 60000) |
| `webSearch`            | Model used for web search tasks                   |
| `image`                | Model used for image-related tasks                |

## Hicap Transformer

Create `C:\Users\<username>\.claude-code-router\hicap-transformer.js`:

> **Mac**: Use `~/.claude-code-router/hicap-transformer.js`

```js
module.exports = class HicapTransformer {
  static name = "hicap";
  name = "hicap";

  async transformRequestIn(request, provider, context) {
    return {
      body: request,
      config: {
        headers: {
          "api-key": provider.apiKey,
          Authorization: undefined,
          authorization: undefined,
        },
      },
    };
  }

  async auth(request, provider, context) {
    return {
      body: request,
      config: {
        headers: {
          "api-key": provider.apiKey,
          Authorization: undefined,
          authorization: undefined,
        },
      },
    };
  }

  async transformResponseOut(response) {
    return response;
  }
};
```

This transformer replaces the default `Authorization` header with Hicap's required `api-key` header.

## Usage

### Start the Router

```shell
ccr start
```

### Run Claude Code

```shell
ccr code "Write a Hello World in Python"
```

### Use Direct Claude Command (Mac/Linux)

```shell
eval "$(ccr activate)"
claude "Your prompt here"
```

> **Windows (PowerShell)**: Run `ccr activate` and follow the displayed instructions to set the environment variables manually.

## Commands

| Command           | Description                    |
| ----------------- | ------------------------------ |
| `ccr start`       | Start the router server        |
| `ccr stop`        | Stop the router server         |
| `ccr restart`     | Restart after config changes   |
| `ccr status`      | Check server status            |
| `ccr code`        | Start Claude Code with router  |
| `ccr ui`          | Open web-based configuration   |
| `ccr model`       | Interactive CLI model selector |
| `ccr preset list` | List available presets         |

## Troubleshooting

1. **API Key Issues**: Ensure the `HICAP_API_KEY` environment variable is set, or replace `$HICAP_API_KEY` with your actual key in the config file.
2. **Transformer Not Found**: Verify that the path in the `transformers` array points to a valid file.
3. **Connection Errors**: Confirm that `ccr start` is running and that port 3456 is not in use by another process.

---

For the latest setup instructions, see: https://github.com/musistudio/claude-code-router

Questions? Reach out to [Hicap support](mailto:support@hicap.ai) or open an issue in the repository.
