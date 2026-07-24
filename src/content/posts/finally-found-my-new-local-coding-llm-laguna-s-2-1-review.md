---
title: "This 118B Model Just Replaced Qwen for My Coding Setup"
description: "I spent months hunting for a reliable local coding LLM. Then I found Laguna S 2.1 - the 118B MoE model that's faster, smarter, and actually thinks before it acts. Here's my complete setup and why Qwen just got replaced."
pubDatetime: 2026-07-24T00:00:00Z
modDatetime: 2026-07-24T00:00:00Z
author: "Sh1d0w"
tags: ["llms", "local-ai", "coding", "benchmarking", "ollama", "laguna"]
draft: false
---

## The Search for a Better Local Coding LLM

If you've been following my experiments with local AI models, you know I've been hunting for the perfect coding assistant. Someone who can actually understand complex tasks, doesn't hallucinate answers, and most importantly - thinks before acting.

After months of frustration with Qwen 3.6 27B (which hallucinates constantly and rarely uses search tools unless I nag it), I think I finally found my replacement: Laguna S 2.1.

## What is Laguna S 2.1?

For those who haven't heard of it, Laguna S 2.1 is a **118B parameter Mixture-of-Experts (MoE)** model from Poolside AI with **~8B activated parameters** per token. It's specifically designed for **agentic coding and long-horizon work**.

Key specs:
- **Total Parameters:** 118B (but only ~8B active at a time)
- **Context Window:** Up to 1 million tokens
- **Architecture:** MoE with 256 experts, top-10 routing across 48 layers
- **Purpose:** Built for terminal tool use, code synthesis, and research workflows
- **License:** OpenMDW-1.1 (with quantizations available)

The MoE architecture is why I can run this beast locally without needing a data center. Only the relevant experts fire up for each task, keeping memory usage manageable.

## Why Laguna S 2.1? The Real Difference

Let me be clear about what I mean by "proactive":

With **Qwen 3.6 27B**, the problem is that it never checks its work. I give it a task to implement something, and it just replays what's in its training memory - no research, no library doc lookups, no checking for current versions. It hallucinates constantly because it assumes everything it "remembers" is accurate, even when it's outdated or wrong.

**Laguna S 2.1 is different.** I give it a task and some tools available (search, context7, etc.), and without me explicitly asking or putting instructions in AGENTS.md, it automatically does extensive research. It looks up current library docs, checks versions, examines the codebase, and only then proposes changes. This happens by default - no nagging required.

I'm currently running the **Q4_K_M GGUF** quantization, which is surprisingly good for a 118B model. (For reference, I run Qwen Q6_K which is more resource-intensive.)

## My Hardware Setup

Here's what I'm running Laguna on:

- **GPU:** RTX 5090 (32GB VRAM)
- **System RAM:** 64 GB DDR4
- **CPU:** Ryzen 7 5700X
- **Context:** 100k tokens (can go higher with config changes)
- **Performance:** ~19 tokens/sec, ~700 tokens prefill speed

That performance is actually **very usable** for interactive coding work. Not instant, but fast enough that you don't feel the wait when thinking through problems.

## Configuration Deep Dive

Here's my exact launch configuration and what each parameter does:

```bash
"$SERVER_BIN" \
    -m "$MODEL_FILE" \
    --chat-template-file "$CHAT_TEMPLATE_FILE" \
    -c 100000 \
    --flash-attn on \
    --threads 8 \
    --n-gpu-layers 999 \
    --n-cpu-moe 32 \
    --parallel 1 \
    -b 4096 \
    -ub 4096 \
    -ctk q8_0 \
    -ctv q8_0 \
    --temp 1.0 \
    --top-p 1.0 \
    --top-k 20 \
    --port 8081
```

Let me break down what each flag does:

### Core Parameters

**`-m "$MODEL_FILE"`** - Path to the model weights file (my Q4_K_M GGUF)

**`--chat-template-file "$CHAT_TEMPLATE_FILE"`** - The template for formatting conversations. This ensures the model receives prompts in the correct format it was trained on.

### Performance Tuning

**`-c 100000`** - Context window size in tokens. I'm using 100k, though the model supports up to 1M with rope-scaling overrides. I can bump this by adjusting `--n-cpu-moe`.

**`--flash-attn on`** - Enables Flash Attention for faster attention computations. This is crucial for long context performance and memory efficiency.

**`--threads 8`** - Number of CPU threads to use. I have 8 because the rest can handle GPU operations. More threads could help with preprocessing but might bottleneck elsewhere.

**`--n-gpu-layers 999`** - Max layers to offload to GPU. Setting this high ensures almost everything runs on the GPU, which is faster than CPU. My RTX 5090 has enough VRAM to handle most of the model.

**`--n-cpu-moe 32`** - Number of MoE layers whose expert weights get offloaded to CPU rather than GPU. This is the secret sauce for running huge models efficiently — each layer still routes across all 256 experts (with top-10 active per token), but keeping more layers' experts on CPU frees up VRAM. With 32, I get a good balance between speed and memory usage. Bump to 40 to free more VRAM for larger contexts.

**`--parallel 1`** - Parallel generation workers. Single worker avoids race conditions in tool use workflows.

### Memory Management

**`-b 4096`** - Logical batch size for processing. 4096 is a good balance between throughput and memory usage.

**`-ub 4096`** - Micro-batch (physical batch) size — the actual chunk size used for GPU compute steps under the hood. It's distinct from `-b`; keeping it in line with your batch size here maximizes throughput at the cost of some peak memory usage.

### Quantization Settings

**`-ctk q8_0`** - K-cache (key cache) quantization type. q8_0 (8-bit) provides a good quality-to-size ratio for context storage.

**`-ctv q8_0`** - V-cache (value cache) quantization type, the counterpart to `-ctk`. Also 8-bit for maintaining quality while reducing memory pressure.

### Generation Parameters

**`--temp 1.0`** - Temperature for generation. 1.0 is balanced - creative enough for problem-solving but not so random that it starts hallucinating.

**`--top-p 1.0`** - Nucleus sampling parameter. 1.0 means consider all tokens (with temperature, this is fine).

**`--top-k 20`** - Top-K sampling. From the top 20 most likely tokens, sample randomly. This adds just enough variety without losing coherence.

**`--port 8081`** - API port for connecting to the model server.

## Benchmarking & Real-World Performance

I've been testing Laguna S 2.1 against Qwen 3.6 27B for local development tasks over the past week. Here's what I've found:

### Short Tests (First Impressions)

✅ **No loops** - Unlike some models that get stuck in repetitive patterns, Laguna stays focused on the task.

✅ **Speed is excellent** - At ~19 t/s with ~700 t/s prefill, it feels responsive. Not chatbot-fast, but productive-fast.

✅ **On par or better than Qwen** - In my initial comparisons, Laguna matches or exceeds Qwen's coding abilities while being much more reliable.

### What I've Tested

- File modifications across multiple files
- Debugging complex errors
- Writing new features from scratch
- Researching unfamiliar libraries
- Explaining code changes

The proactive behavior shows up even in simple tasks. Ask it to refactor something, and it will show you what it plans to change, why, and ask for confirmation. It's like having a pair programmer who actually reads the code before touching it.

## The Verdict (So Far)

I'm still benchmarking this more thoroughly, but **Laguna S 2.1 is looking like the replacement for my local coding setup**.

The combination of:
- **Reliability** (no hallucinations in my tests)
- **Proactive reasoning** (thinks before acting)
- **Good performance** (usable speed on consumer hardware)
- **Long context** (actually handles large codebases)

...makes it uniquely suited for day-to-day development work.

I'll continue testing and comparing with Qwen, but I'm optimistic. This might be the model that makes local AI truly useful for professional coding work.

## Want to Try It?

If you have the hardware (118B parameters is heavy), Laguna S 2.1 is available in multiple GGUF quantizations on Poolside's Hugging Face page:

👉 [huggingface.co/poolside/Laguna-S-2.1-GGUF](https://huggingface.co/poolside/Laguna-S-2.1-GGUF)

A few things worth knowing before you dive in:

**File sizes:** Q4_K_M (the quant I'm running) is ~75GB, Q8_0 is ~128GB, and full-precision F16 is ~235GB. There's also a small ~2.2GB DFlash-BF16 draft model for speculative decoding.

**Default context:** the published GGUFs ship configured for 256K tokens. The weights support up to the full 1M context, but that requires overriding the rope-scaling settings at load time and comes with a quality trade-off (Poolside recommends `--temp 0.7 --top-p 0.95` if you go that route).

**Serving:** for full Laguna support (including DFlash speculative decoding), Poolside recommends their llama.cpp fork's laguna branch rather than mainline llama.cpp, since upstream support is still in PR review at the time of writing.

### Getting Started

1. **Clone Poolside's llama.cpp fork and build it:**
```bash
git clone --branch laguna https://github.com/poolsideai/llama.cpp
cd llama.cpp && cmake -B build && cmake --build build -j
```

2. **Download the quant you want (via the Hugging Face CLI, git-lfs, or the -hf flag shown below), then start the server:**
```bash
./build/bin/llama-server -m laguna-s-2.1-Q4_K_M.gguf --jinja --port 8000
```

3. **Or, if you want to squeeze out extra tokens/sec, enable DFlash speculative decoding using the small draft model:**
```bash
./build/bin/llama-server -m laguna-s-2.1-Q4_K_M.gguf \
  -md laguna-s-2.1-DFlash-BF16.gguf \
  --spec-type draft-dflash --spec-draft-n-max 15 -fa on --jinja --port 8000
```

4. **For low-VRAM setups, add `--n-cpu-moe` (as in my config above) to offload MoE expert layers to system RAM — this is how I run a 75GB quant on a 32GB card.**

Since llama.cpp also supports pulling straight from Hugging Face by repo ID, you can skip the manual download entirely:

```bash
llama-server -hf poolside/Laguna-S-2.1-GGUF:Q4_K_M --jinja --port 8000
```

The key is finding the right balance between model size, quantization, and your hardware. For me, Laguna S 2.1 hit that sweet spot where the intelligence matches the performance cost.

---

*What local models are you using for coding? Have you tried Laguna S 2.1? Drop a comment or reach out on Twitter - I'm always happy to discuss AI model performance and configurations.*
