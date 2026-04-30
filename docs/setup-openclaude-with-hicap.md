---
title: "How to Set Up OpenClaude with Hicap"
date: "2026-04-29"
author: "Hicap Engineering"
description: "A step-by-step guide for Hicap customers to configure OpenClaude with the Hicap API."
---

# Setting Up OpenClaude with Hicap

This guide walks you through configuring [OpenClaude](https://github.com/Gitlawb/openclaude) to work with Hicap as a custom OpenAI-compatible provider.

## Prerequisites

- Node.js and npm installed locally
- A Hicap API key from [platform.hicap.ai](https://platform.hicap.ai)
- A Hicap model ID you want OpenClaude to use

## 1. Install OpenClaude

```bash
npm install -g @gitlawb/openclaude
```

Start OpenClaude from the project you want to work in:

```bash
openclaude
```

## 2. Open the Provider Manager

Inside OpenClaude, run `/provider`.

Choose **Add provider**, then choose **Custom** from the provider preset list.

## 3. Enter the Hicap Provider Fields

The custom provider flow asks for these fields in order:

| OpenClaude field | Hicap value |
| --- | --- |
| Provider name | `Hicap` |
| Base URL | `https://api.hicap.ai/v1` |
| Default model | Your Hicap model ID |
| API mode | `Chat Completions` |
| Auth header | `api-key` |
| Auth header value | Your Hicap API key |
| API key | Leave blank |

Hicap authenticates with the `api-key` header, so put your key in **Auth header value** and leave the final **API key** field empty.

## 4. Save and Use Hicap

Press Enter through the final empty API key field. OpenClaude saves the provider profile and makes it active automatically.

You can return to `/provider` later to edit the Hicap profile, switch active providers, or delete the profile.

## Troubleshooting

### Authentication errors

- Confirm **Auth header** is exactly `api-key`.
- Confirm **Auth header value** contains your Hicap key.
- Confirm the final **API key** field was left blank.

### Model errors

- Verify the **Default model** matches a model available in your Hicap account.
- Check the Hicap model catalog at [hicap.ai/models](https://hicap.ai/models).

### OpenClaude uses the wrong provider

Run `/provider`, choose **Set active provider**, then select **Hicap**.
