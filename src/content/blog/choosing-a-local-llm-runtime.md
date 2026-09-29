---
title: "Choosing a Local LLM Runtime: From LM Studio to vLLM, llama.cpp, and Unsloth Desktop"
description: "A hands-on look at the tradeoffs between local LLM runtimes: ease of use, API serving, multi-GPU behavior, observability, and experimentation."
pubDate: 2026-10-08
updatedDate: 2026-10-08
tags:
  - local-llm
  - llm-runtime
  - lm-studio
  - vllm
  - llama-cpp
  - unsloth
draft: true
heroImage: '../../assets/blog-placeholder-4.jpg'
---

## The Runtime Is Part of the System

Buying GPUs does not create a useful LLM server by itself.

The runtime determines how models load, how memory is allocated, how requests are served, which quantization formats work, what metrics are exposed, and whether an experiment can become a reliable workflow. I started with the most approachable option and gradually moved toward tools that offered more operational control.

This is not a ranking. Each runtime solved a different problem at a different stage of the journey.

## Stage One: LM Studio Made Local Inference Concrete

LM Studio was the fastest path from downloaded model to a working local chat and API server.

It made several early tasks simple:

- Downloading and organizing models.
- Testing GGUF quantizations.
- Adjusting context and GPU-offload settings.
- Exploring a model interactively before integrating it into software.
- Enabling a local OpenAI-compatible API for applications and agents.

For learning, that low-friction experience mattered more than maximum configurability. It allowed me to focus on practical questions: Does this model fit? Is the quality useful? How does context length affect memory? Is the output fast enough for interactive work?

## Stage Two: vLLM Raised the Bar for Serving

As my focus shifted from desktop experimentation toward service-style inference, vLLM became compelling.

The appeal was not simply raw speed. It was the runtime’s orientation toward serving: batching, throughput, API compatibility, and the kinds of operational behavior that matter when more than one request or tool may be using the model.

The questions I explored with vLLM included:

- Which models and formats fit its supported path well?
- How does its memory-management approach change usable context?
- How does it behave with my multi-GPU configuration?
- When does a higher-throughput serving runtime justify its added setup complexity?
- Which observability signals are available without building custom instrumentation?

vLLM was an important reminder that “works on my desktop” and “serves applications well” are related but different goals.

## Stage Three: llama.cpp Offered Control and Visibility

llama.cpp became central when I wanted a lightweight, explicit, highly configurable local serving path for GGUF models.

Its value for my setup was control:

- Clear model and context configuration.
- Practical multi-GPU layer splitting.
- Broad support for quantized GGUF files.
- A server mode suitable for application integration.
- Logs and metrics that can be incorporated into monitoring.

With a Qwen-class 27B model, the details become meaningful: quantization choice, context length, KV-cache precision, speculative decoding, prompt processing rate, generation rate, and GPU utilization all affect the usable experience.

That precision is also the cost. llama.cpp exposes choices that a desktop application can hide. It rewards understanding the system rather than treating local inference as a single-button feature.

## Stage Four: Testing Unsloth Desktop

Unsloth Desktop entered the evaluation as another attempt to improve the local model workflow.

I tested it with the same mindset used for the other runtimes: not “is this universally best?” but “what workflow does this improve, and what does it make harder?”

The evaluation criteria were consistent:

- How quickly can I get a model running?
- Which model formats and quantizations are practical?
- How well does it use two consumer GPUs?
- Does it expose enough configuration for long-context work?
- Can it serve my applications reliably?
- What performance and telemetry can I observe?
- What additional operational dependency does it introduce?

## How I Now Think About Runtime Choice

A local runtime should be selected by workload, not ideology.

| Need | Runtime direction to evaluate |
| --- | --- |
| Fastest route to interactive testing | LM Studio |
| Service-oriented throughput experiments | vLLM |
| GGUF flexibility, low-level control, and instrumentation | llama.cpp |
| A simplified or specialized local workflow | Unsloth Desktop |

The important part is establishing a repeatable test harness. Use the same model family, context target, prompts, API client, and measurement method. Otherwise, a runtime comparison becomes a comparison of unrelated configurations.

## What Comes Next

Once I had a working server and several runtime options, the next problem changed from “Can I run this model?” to “What does it cost, how does it perform over time, and which model is worth running?”
