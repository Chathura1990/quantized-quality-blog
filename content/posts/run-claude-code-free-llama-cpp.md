---
title: "How to Run Claude Code 100% Free with llama.cpp"
date: 2026-02-28T10:30:00+05:30
draft: false
tags: ["ai", "llama-cpp", "claude-code", "local-llm", "free"]
categories: ["AI", "Development"]
description: "A step-by-step guide to running Claude Code with local open-source models using llama.cpp - completely free."
ShowToc: true
TocOpen: false
---

{{< lead >}}
Claude Code is an incredible AI coding assistant, but at $125/month for Opus access, it can get expensive fast. The good news? Claude Code now supports local models through integrations like llama.cpp, LM Studio & Ollama letting you run the same CLI experience completely free.
{{< /lead >}}

In this guide, we'll set up Claude Code with a local LLM using llama.cpp—no API fees, no subscriptions, just free AI coding assistance on your own machine.

---

## Tested On

| Computer            | Specifications                  |
| ------------------- | ------------------------------- |
| **MacBook Pro**     | Apple M4 Pro chip, 24 GB RAM    |
| **Windows Desktop** | Intel Core i7-13700H, 32 GB RAM |

All setups ran local LLMs with Claude Code successfully.

---

## The Setup

| Component       | Purpose                     |
| --------------- | --------------------------- |
| **llama.cpp**   | Local engine to serve LLMs  |
| **Claude Code** | AI coding assistant CLI     |
| **Local LLM**   | Your choice of coding model |

---

## Step 1: Install Claude Code

If you haven't already, install Claude Code:

{{< terminal title="Install Claude Code with Mac OS" >}}
curl -fsSL https://claude.ai/install.sh | bash
{{< /terminal >}}

{{< terminal title="Install Claude Code with Windows CMD" >}}
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
{{< /terminal >}}

Or follow the official guide at [docs.anthropic.com](https://docs.anthropic.com/en/docs/claude-code/overview).

---

## Step 2: Build llama.cpp

First, let's build llama.cpp from source with the necessary features.

### macOS (Apple Silicon)

{{< terminal title="Build llama.cpp on Mac" >}}
brew install cmake
git clone https://github.com/ggml-org/llama.cpp
cmake llama.cpp -B llama.cpp/build -DGGML_METAL=ON
cmake --build llama.cpp/build --config Release -j
cp llama.cpp/build/bin/llama-\* llama.cpp/
{{< /terminal >}}

The `-DGGML_METAL=ON` flag enables Apple Silicon GPU acceleration.

### Linux (NVIDIA GPU)

{{< terminal title="Build llama.cpp on Linux" >}}
sudo apt-get update && sudo apt-get install build-essential cmake git -y
git clone https://github.com/ggml-org/llama.cpp
cmake llama.cpp -B llama.cpp/build -DGGML_CUDA=ON
cmake --build llama.cpp/build --config Release -j
cp llama.cpp/build/bin/llama-\* llama.cpp/
{{< /terminal >}}

---

## Step 3: Start the Local Server

Now let's start the llama.cpp server with a coding-capable model. We'll use Qwen3-Coder for this example.

{{< terminal title="start-server.sh" >}}
./llama-server -hf ggml-org/gpt-oss-20b-GGUF --jinja -c 60000 --host 127.0.0.1 --port 8033
{{< /terminal >}}

> **Important:** The `--jinja` flag is mandatory. Without it, tool calling won't work properly.

### Recommended Models

| Model                      | Best For       | RAM Required |
| -------------------------- | -------------- | ------------ |
| **devstral-small-2 (24B)** | General coding | 16GB         |
| **Qwen3-Coder (30B)**      | Complex tasks  | 24GB         |
| **gpt-oss-20b**            | Fast responses | 12GB         |

---

## Step 4: Connect Claude Code

Add these environment variables to (`~/.claude/llamacpp.setting.json`):
`http://127.0.0.1:8033/` the same URL and port that llama.cpp is running.
{{< terminal title="/llamacpp.setting.json" >}}
{
"env": {
"ANTHROPIC_BASE_URL": "http://127.0.0.1:8033/",
"ANTHROPIC_AUTH_TOKEN": "anthropic_api_key",
"API_TIMEOUT_MS": "3000000",
"CLAUDE_CODE_DIABLE_NONESSENTIAL_TRAFFIC": 1,
"ANTRHOPIC_MODEL": "default_model"
}
}

{{< /terminal >}}

---

## Step 5: Run Claude Code

Now open the project directory & launch Claude Code with your local model:

{{< terminal title="Run Claude Code locally" >}}
claude --settings ~/.claude/llamacpp.setting.json
{{< /terminal >}}

You're now running Claude Code with a local LLM—completely free.

---

## Why This Setup?

### 1. **100% Free**

- No API costs
- No subscription fees
- Just electricity

### 2. **Complete Privacy**

- Your code never leaves your machine
- Work offline
- No data sent to external servers

### 3. **Full Claude Code Experience**

- Same CLI interface
- Tool calling support
- Edit, read, and write files
- Execute bash commands

---

## Performance Notes

### On MacBook Pro M4 Pro (24GB RAM)

- Recommended: Use Q4_K_M or Q5_K_S quantizations
- 24B models run smoothly

### On Desktop with GPU

- NVIDIA GPU significantly speeds up inference
- Use `--ngl 99` to offload all layers to GPU

---

## Alternative: Using Ollama

If you want an even easier setup, Ollama provides a simpler path:

{{< terminal title="Ollama setup" >}}

# Install Ollama

curl -fsSL https://ollama.com/install.sh | sh

# Pull a model

ollama pull devstral-small-2

# Connect to Claude Code

export ANTHROPIC_AUTH_TOKEN="ollama"
export ANTHROPIC_API_KEY=""
export ANTHROPIC_BASE_URL="http://localhost:11434"

ollama launch claude
{{< /terminal >}}

---

## Troubleshooting

### 1. Claude Code Shows "Sonnet" or Another Cloud Model

This is a known quirk. Claude Code's UI may display "Sonnet" or another model name even when using your local LLM. Don't worry—this is just a display issue.

**To verify you're actually using your local LLM:**

1. **Turn off your internet** (disconnect Wi-Fi or ethernet)
2. **Keep the llama.cpp server running** in one terminal
3. **Open a new terminal** and run Claude Code

If Claude Code still works and responds, congratulations—you're running completely offline with your local LLM!

**Also check the llama.cpp terminal:**

- You'll see incoming requests being processed
- Look for your model name in the logs
- Token usage will show local inference stats

This is the ultimate test: if it works without internet, it's definitely your local model.

### 2. Tool Calling Not Working

- Ensure `--jinja` flag is set
- Verify model supports tool calling
- Check server logs for errors

### 3. Slow Performance

- Close unnecessary applications
- Use a smaller quantization (Q4_K_M)
- Consider using a smaller model (14B-24B)

### 4. Connection Refused

- Verify llama-server is running on the correct port
- Check firewall settings
- Ensure `localhost` resolves correctly

---

## Cost Comparison

| Setup                       | Monthly Cost |
| --------------------------- | ------------ |
| Claude Code + Opus          | $125/month   |
| Claude Code + Sonnet        | $25/month    |
| **llama.cpp + Local Model** | **Free**     |

---

## Conclusion

Running Claude Code with llama.cpp gives you the best of both worlds: the powerful CLI experience of Claude Code with the privacy and cost benefits of local AI. While the model quality may not match Anthropic's frontier models, it's an excellent free alternative for developers on a budget.

> **Ready to try?** Start with a smaller model like devstral-small-2 to test the waters, then scale up as needed.
