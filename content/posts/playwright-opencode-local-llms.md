---
title: "The Privacy-First QA: 100% Local AI-Powered Playwright Setup"
date: 2026-01-20T10:30:00+05:30
draft: false
tags: ["playwright", "testing", "ai", "llama-cpp", "automation"]
categories: ["Quality Assurance", "AI"]
description: "A step-by-step guide for Automation QAs to build a fully local AI testing stack using Playwright, OpenCode, and llama.cpp for maximum data privacy."
ShowToc: true
TocOpen: false
---

{{< lead >}}
In modern software testing, AI agents are revolutionizing how we write, run, and heal automated tests. However, most enterprise projects forbid using cloud-based AI models (like ChatGPT or Claude) due to strict data privacy and security policies.
{{< /lead >}}

What if you could have the power of a Senior QA Agent living entirely on your local machine? In this guide, we will build a **100% local AI testing stack** that requires no internet connection and keeps your proprietary code private.

---

## The Tech Stack

| Component | Purpose |
|-----------|---------|
| **llama.cpp** | Local engine to serve LLMs |
| **GPT-OSS-20B** | High-performance coding model with tool-calling |
| **OpenCode** | Terminal-based agent orchestrator |
| **Playwright MCP Agents** | Bridge for AI to control browsers and run tests |

---

## Step 1: Setting up the Brain (llama.cpp)

We need an API that "talks" to our AI model. We'll use the **GPT-OSS-20B** model because it supports the **Harmony** architecture, allowing the AI to "think" before it acts—crucial for complex testing tasks.

### 1. Download the model

llama.cpp can download models directly from Hugging Face:

### 2. Launch the local server

Create a script named `start-ai.sh`.

{{< terminal title="start-ai.sh" >}}
./llama-server \
-hf ggml-org/gpt-oss-20b-GGUF \
-ngl 99 \
-c 32768 \
--jinja \
--chat-template harmony \
--reasoning-format none \
--temp 0.0 \
--top-k 1 \
--port 8033
{{< /terminal >}}

> **Important:** Setting `--temp 0.0` is mandatory. It ensures the model outputs valid JSON for our automation tools without getting "creative."

---

## Step 2: Preparing the Hands (Playwright MCP)

The **Model Context Protocol (MCP)** is what allows the LLM to actually "see" and "touch" your local environment.

### 1. Install Browser Binaries

{{< terminal title="Terminal" >}}
npx playwright install chromium
{{< /terminal >}}

### 2. Initialize the Project Bridge

In your project root, run the following to prepare the agent instructions:

{{< terminal title="Terminal" >}}
npx playwright init-agents --loop=opencode
{{< /terminal >}}

---

## Step 3: Configuring the Orchestrator (OpenCode)

OpenCode is the "command center." We need to point it to our local llama.cpp server and enable the Playwright tools. Edit your `opencode.json`:

{{< terminal title="opencode.json" >}}
{
  "$schema": "https://opencode.ai/config.json",
  "model": "llama-cpp-local/gpt-oss-20b",
  "tools": {
    "read": true,
    "edit": true,
    "bash": true,
    "list": true,
    "grep": true
  },
  "agent": {
    "general": {
      "mode": "subagent",
      "model": "llama-cpp-local/gpt-oss-20b"
    }
  },
  "mcp": {
    "playwright-test": {
      "type": "local",
      "command": [
        "npx",
        "playwright",
        "run-test-mcp-server",
        "--headed"
      ],
      "enabled": true,
      "timeout": 300000
    }
  },
  "provider": {
    "llama-cpp-local": {
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "baseURL": "http://127.0.0.1:8033/v1",
        "apiKey": "not-needed"
      },
      "models": {
        "gpt-oss-20b": {
          "name": "Local GPT-OSS",
          "tool_call": true
        }
      }
    }
  }
}
{{< /terminal >}}

---

## Step 4: Running Your First AI-Powered Test

Launch the agent by typing `opencode` in your terminal. Now, you can give it high-level instructions:

### Example Prompt

{{< terminal title="OpenCode Agent" >}}
"Run the login tests in the /tests folder. If the 'Forgot Password' test fails, take a screenshot and tell me if the CSS selector for the email input has changed."
{{< /terminal >}}

Because we added `--headed` in the config, you will see a browser window pop up and watch the AI navigate your app in real-time.

---

## Performance Optimization Tips

Running a 20B parameter model locally requires resources. Here are tips to squeeze out maximum performance:

### 1. Close Unnecessary Applications

Free up RAM for the LLM by closing:
- Browser tabs (keep only what's needed for testing)
- Music/video streaming apps
- Cloud sync services (Dropbox, OneDrive, iCloud)
- Background apps via Task Manager

### 2. Allocate More VRAM (GPU Users)

If you have a dedicated GPU, maximize VRAM usage:
- Use `--n-gpu-layers 99` (or all available layers)
- Update your GPU drivers to the latest version
- For NVIDIA: Use CUDA toolkit for better performance

### 3. Allocate More RAM (CPU Users)

For CPU-only inference, increase context allocation:
- Use `-c 32768` or higher for larger context
- Set `--threads` to your CPU core count
- Consider setting `--mlock` to lock model in memory

### 4. Use Quantized Models

The MXFP4 quantization you chose is already optimized. For even faster performance:
- Try Q4_K_M or Q5_K_S quantizations
- Smaller quantizations = faster inference but lower quality

### 5. Optimize Your System

- **Disable hyperthreading** if experiencing slowdowns
- **Use SSD** for model loading (faster startup)
- **Close virtual machines** or Docker containers
- **Set high performance power plan** in Windows

### 6. Monitor Resources

Keep an eye on resource usage:
- **Task Manager** → Monitor CPU/RAM usage
- **GPU-Z** (NVIDIA) or **Radeon Software** (AMD) → Monitor VRAM
- **llama-server logs** → Check for memory warnings

---

## Troubleshooting

### 1. "Unknown agent type" Error

If your local model tries to start a sub-task and fails, it's likely a hallucination in the `subagent_type` field. Ensure your `llama-server` is running at `temp 0.0` and that you have defined the `agent` block in your `opencode.json`.

### 2. Timeout -32001

Local models can be slow on the first "thought." We set the timeout to `300000` (5 minutes) in the JSON to ensure OpenCode doesn't give up while your CPU/GPU is crunching the data.

---

## Conclusion

Data privacy shouldn't stop you from using the latest AI advancements. By running **GPT-OSS** via **llama.cpp**, you get the reasoning power of a large model with the security of an air-gapped environment.

> **Happy (and private) testing!**