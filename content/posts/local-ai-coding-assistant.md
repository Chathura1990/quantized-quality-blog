---
title: "The Privacy-First Developer: 100% Local AI Coding Assistant"
date: 2026-01-15T10:30:00+05:30
draft: false
tags: ["ai", "llama-cpp", "coding", "local-llm", "development"]
categories: ["AI", "Development"]
description: "A step-by-step guide for developers to build a fully local AI coding assistant using OpenCode and llama.cpp for maximum data privacy."
ShowToc: true
TocOpen: false
---

{{< lead >}}
Every line of code you write is intellectual property. When you use cloud-based AI coding assistants, your code gets sent to external servers—often stored, analyzed, and potentially exposed. For developers working on proprietary projects, this creates a significant security and compliance challenge.
{{< /lead >}}

In this guide, we'll build a **100% local AI coding assistant** that runs entirely on your machine. No internet required, no data leaves your computer—just you and your AI pair programmer.

---

## The Tech Stack

| Component | Purpose |
|-----------|---------|
| **llama.cpp** | Local engine to serve LLMs |
| **GPT-OSS-20B** | High-performance coding model with tool-calling |
| **OpenCode** | Terminal-based AI coding assistant |
| **Your IDE** | Continue using your favorite editor (VS Code, JetBrains, etc.) |

---

## Step 1: Setting up the Brain (llama.cpp)

We need an API that "talks" to our AI model. We'll use the **GPT-OSS-20B** model because it supports the **Harmony** architecture, allowing the AI to "think" before it acts—crucial for complex coding tasks.

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

> **Important:** Setting `--temp 0.0` is mandatory. It ensures the model outputs valid code without getting "creative."

---

## Step 2: Configuring OpenCode for Development

OpenCode is your "AI pair programmer." We need to point it to our local llama.cpp server and enable developer tools. Edit your `opencode.json`:

{{< terminal title="opencode.json" >}}
{
  "$schema": "https://opencode.ai/config.json",
  "model": "llama-cpp-local/gpt-oss-20b",
  "tools": {
    "read": true,
    "write": true,
    "edit": true,
    "bash": true,
    "list": true,
    "grep": true,
    "search": true,
    "glob": true,
    "task": true
  },
  "agent": {
    "general": {
      "mode": "subagent",
      "model": "llama-cpp-local/gpt-oss-20b"
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

## Step 3: Using OpenCode for Development

Launch OpenCode by typing `opencode` in your terminal. Now you can give it coding instructions:

### Example Tasks

**Refactor a function:**
{{< terminal title="OpenCode Agent" >}}
"Find the function 'processUserData' in src/utils.ts and refactor it to use async/await properly. Also add proper error handling."
{{< /terminal >}}

**Explain complex code:**
{{< terminal title="OpenCode Agent" >}}
"Read the file src/auth/middleware.ts and explain what the authentication flow does in simple terms."
{{< /terminal >}}

**Write tests:**
{{< terminal title="OpenCode Agent" >}}
"Create unit tests for the calculateTotal function in src/cart.ts using Jest."
{{< /terminal >}}

**Find bugs:**
{{< terminal title="OpenCode Agent" >}}
"Search for all TODO comments in the codebase and help me implement them."
{{< /terminal >}}

---

## Why Local AI for Development?

### 1. **Data Privacy**
- Your code never leaves your machine
- No risk of IP leakage to cloud providers
- Comply with strict enterprise security policies

### 2. **Offline Development**
- Work on planes, trains, or remote locations
- No dependency on internet connectivity
- Consistent AI assistance anywhere

### 3. **Cost Efficiency**
- No subscription fees
- One-time cost of hardware
- Unlimited usage after setup

### 4. **Customization**
- Fine-tune models on your codebase
- Create custom prompts for your stack
- Full control over model behavior

---

## Performance Optimization Tips

Running a 20B parameter model locally requires resources. Here are tips to squeeze out maximum performance:

### 1. Close Unnecessary Applications

Free up RAM for the LLM by closing:
- Browser tabs (keep only what's needed)
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

### 3. Slow Code Generation

If the model generates code slowly:
- Reduce context size (`-c 8192` instead of `32768`)
- Close other applications
- Consider using a smaller quantization (Q4_K_M)

---

## Conclusion

Data privacy shouldn't stop you from using AI coding assistants. By running **GPT-OSS** via **llama.cpp**, you get the power of an AI pair programmer with the security of an air-gapped environment.

> **Happy (and private) coding!**
