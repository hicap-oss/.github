---
title: "How to Set Up Braintrust with Hicap"
date: "2026-11-11"
author: "Hicap Engineering"
description: "Configure Hicap as a custom provider in Braintrust."
---

# Setting Up Braintrust with Hicap

Configure Hicap as a custom OpenAI-compatible provider in Braintrust to use available Hicap models in prompts and evaluations.

## Prerequisites

- A Braintrust project or organization where you can configure AI providers
- A Hicap API key and a model available to your account

## 1. Add a Custom Provider

Open your Braintrust project or organization AI provider settings. Under **AI provider keys**, select **Add custom provider**, name it **Hicap**, and select **API key** authentication. Enter your Hicap API key.

## 2. Configure Requests

Expand **More settings** and set **API base URL** to `https://api.hicap.ai/v1`. Leave the endpoint path at its default for OpenAI-compatible chat completions; the resulting URL should be `https://api.hicap.ai/v1/chat/completions`.

Hicap requires the API key in an `api-key` request header. If the authorization header format offers an `api-key` header option, select it. Otherwise, use **Additional header** to set `api-key` to your Hicap key. Use your Hicap key for this provider, not your Braintrust API key.

## 3. Add Models and Verify

Under **Models**, select the supported registry models you need or add custom models using their exact Hicap model IDs. Save the provider, select it and one of its models in a Braintrust prompt or playground, and send a test message.

If authentication fails, check the `api-key` header and Hicap key. If you receive a not-found error, check the model ID and final endpoint URL. Model availability depends on your Hicap account. See [Hicap models](https://hicap.ai/models) for available models.

See [Braintrust's custom provider overview](https://braintrust.dev/blog/any-framework-any-provider) for more information.

