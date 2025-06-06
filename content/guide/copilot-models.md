---
title: Copilot Available Models
draft: false
date: 2025-05-14
tags:
  - ai
  - copilot
  - ai-models
---

I need to know which Copilot models are available in my current environment.
I found that
[this link](https://main.vscode-cdn.net/extensions/copilotChat.json) provides
the list of available models:

As of 2025-05-14, these models are available:

```json
{
  "version": 1,
  "modelInfo": {
    "OpenAI": {
      "gpt-4.1": {
        "name": "GPT-4.1",
        "toolCalling": true,
        "vision": true,
        "maxInputTokens": 1014808,
        "maxOutputTokens": 32768
      },
      "gpt-4.1-mini": {
        "name": "GPT-4.1 mini",
        "toolCalling": true,
        "vision": true,
        "maxInputTokens": 1014808,
        "maxOutputTokens": 32768
      },
      "gpt-4.1-nano": {
        "name": "GPT-4.1 nano",
        "toolCalling": true,
        "vision": true,
        "maxInputTokens": 1014808,
        "maxOutputTokens": 32768
      },
      "gpt-4.5-preview": {
        "name": "GPT-4.5 Preview",
        "toolCalling": true,
        "vision": true,
        "maxInputTokens": 111616,
        "maxOutputTokens": 16384
      },
      "gpt-4o-2024-11-20": {
        "name": "GPT-4o (2024-11-20)",
        "toolCalling": true,
        "vision": true,
        "maxInputTokens": 111616,
        "maxOutputTokens": 16384
      }
    },
    "Anthropic": {
      "claude-3-7-sonnet-20250219": {
        "name": "Claude 3.7 Sonnet",
        "vision": true,
        "toolCalling": true,
        "maxInputTokens": 64000,
        "maxOutputTokens": 16384
      },
      "claude-3-5-sonnet-20241022": {
        "name": "Claude 3.5 Sonnet",
        "vision": true,
        "toolCalling": true,
        "maxInputTokens": 64000,
        "maxOutputTokens": 8192
      },
      "claude-3-5-haiku-20241022": {
        "name": "Claude 3.5 Haiku",
        "vision": true,
        "toolCalling": true,
        "maxInputTokens": 64000,
        "maxOutputTokens": 8192
      }
    },
    "Gemini": {
      "models/gemini-2.5-pro-preview-03-25": {
        "name": "Gemini 2.5 Pro Preview",
        "vision": true,
        "toolCalling": true,
        "maxInputTokens": 819600,
        "maxOutputTokens": 65536
      },
      "models/gemini-2.5-pro-exp-03-25": {
        "name": "Gemini 2.5 Pro Exp",
        "vision": true,
        "toolCalling": true,
        "maxInputTokens": 819600,
        "maxOutputTokens": 65536
      },
      "models/gemma-3-27b-it": {
        "name": "Gemma 3 27b",
        "vision": true,
        "toolCalling": false,
        "maxInputTokens": 64000,
        "maxOutputTokens": 8192
      },
      "models/gemini-2.5-flash-preview-04-17": {
        "name": "Gemini 2.5 Flash Preview",
        "vision": true,
        "toolCalling": true,
        "maxInputTokens": 819600,
        "maxOutputTokens": 65536
      }
    },
    "Groq": {
      "qwen-qwq-32b": {
        "name": "Qwen QWQ 32B",
        "toolCalling": true,
        "vision": false,
        "maxInputTokens": 131072,
        "maxOutputTokens": 8192
      },
      "qwen-2.5-coder-32b": {
        "name": "Qwen 2.5 Coder 32B",
        "toolCalling": true,
        "vision": false,
        "maxInputTokens": 131072,
        "maxOutputTokens": 8192
      },
      "qwen-2.5-32b": {
        "name": "Qwen 2.5 32B",
        "toolCalling": true,
        "vision": false,
        "maxInputTokens": 131072,
        "maxOutputTokens": 8192
      },
      "deepseek-r1-distill-qwen-32b": {
        "name": "DeepSeek R1 Distill Qwen 32B",
        "toolCalling": true,
        "vision": false,
        "maxInputTokens": 131072,
        "maxOutputTokens": 16384
      },
      "deepseek-r1-distill-llama-70b": {
        "name": "DeepSeek R1 Distill Llama 70B",
        "toolCalling": true,
        "vision": false,
        "maxInputTokens": 131072,
        "maxOutputTokens": 8192
      },
      "llama-3.3-70b-versatile": {
        "name": "Llama 3.3 70B Versatile",
        "toolCalling": true,
        "vision": false,
        "maxInputTokens": 131072,
        "maxOutputTokens": 32768
      },
      "llama-3.1-8b-instant": {
        "name": "Llama 3.1 8B Instant",
        "toolCalling": true,
        "vision": false,
        "maxInputTokens": 131072,
        "maxOutputTokens": 8192
      },
      "mixtral-8x7b-32768": {
        "name": "Mixtral 8x7B 32768",
        "toolCalling": true,
        "vision": false,
        "maxInputTokens": 32768,
        "maxOutputTokens": 8192
      },
      "gemma2-9b-it": {
        "name": "Gemma2 9B IT",
        "toolCalling": true,
        "vision": false,
        "maxInputTokens": 8192,
        "maxOutputTokens": 8192
      }
    }
  }
```
