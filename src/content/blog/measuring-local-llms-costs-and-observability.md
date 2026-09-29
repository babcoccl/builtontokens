---
title: "What Does a Local LLM Token Cost? Measuring Qwen-Class Inference with Prometheus and Grafana"
description: "How I turned local LLM power draw, token throughput, and electricity prices into a practical cost-per-token dashboard."
pubDate: 2026-10-15
updatedDate: 2026-10-15
tags:
  - local-llm
  - qwen
  - observability
  - grafana
  - prometheus
  - cost-analysis
draft: true
heroImage: '../../assets/blog-placeholder-2.jpg'
---

## Local Is Not Free

Running a model locally avoids per-token API billing, but it does not make inference free.

The cost moves into hardware, electricity, cooling, maintenance, and the opportunity cost of tying up GPUs. I wanted a more useful answer than “local inference is cheap,” so I built a way to measure it.

The goal was not to manufacture a universal number. It was to calculate a number that reflected my hardware, electricity rate, runtime settings, and actual usage pattern.

## The Question I Wanted to Answer

For a Qwen-class model in the 27B range, I wanted to estimate:

- Cost per generated token.
- Cost per million generated tokens.
- Difference between prompt processing and token generation.
- Cost at idle versus sustained inference.
- Whether speculative decoding or runtime changes improved the economics.
- How local cost compares with an API only after accounting for realistic utilization.

A token-cost estimate without throughput and power data is mostly a guess.

## The Core Calculation

The basic relationship is straightforward:

\[
\text{Cost per hour} =
\frac{\text{Watts}}{1000}
\times
\text{Electricity price per kWh}
\]

Then convert hourly cost into token cost:

\[
\text{Cost per generated token} =
\frac{\text{Cost per hour}}{\text{Generated tokens per hour}}
\]

For a system generating at an average of approximately 31 tokens per second, the hourly output is:

\[
31
\times
3600
=
111{,}600
\text{ tokens per hour}
\]

That calculation becomes more realistic when the inputs are measured separately for prompt prefill, token generation, and idle time.

## What I Measured

My dashboard pulls together two sides of the workload:

- **LLM runtime metrics:** request activity, prompt-processing throughput, generated-token throughput, context behavior, and runtime events.
- **GPU and host metrics:** GPU power draw, utilization, memory use, temperatures, and system state.

I used Prometheus as the time-series store and Grafana as the visualization layer. The goal was to make cost and performance visible in the same view instead of manually matching terminal logs to a power meter.

## Why Input and Output Tokens Need Separation

Prompt processing and token generation stress the system differently.

Prompt prefill can process large quantities of input tokens quickly while drawing a different power profile than steady autoregressive generation. Output generation is slower, more interactive, and often the metric that determines whether a coding or chat workflow feels usable.

A dashboard that merges them carelessly can produce misleading “average cost per token” numbers. It can also show zero or near-zero values during idle periods even though the machine still consumes power.

## The Dashboard Views That Matter

### Throughput

Track input tokens per second and output tokens per second separately.

A fast prefill rate does not guarantee responsive generation. For agentic coding, the output rate often has a greater impact on perceived responsiveness.

### Power

Track GPU power by device and the total inference system draw where possible.

Power should be viewed alongside request activity. A flat average over long idle periods hides what an active inference run actually costs.

### Cost Per Token

Calculate cost per generated token and cost per million generated tokens over a meaningful time window.

The most useful view preserves or averages completed runs rather than instantly collapsing after a short burst of activity. Otherwise, transient workloads make the cost panel look artificially low or empty.

### GPU Memory and Context

Track memory utilization alongside configured context length and model configuration.

This connects a practical question—“Can I increase context?”—to the resource constraint that actually determines the answer.

## Why This Is More Than a Dashboard Project

Instrumentation changes runtime decisions.

Once throughput, power, and cost are visible together, it becomes easier to evaluate:

- Whether a larger model earns its slower output rate.
- Whether a quantization change is worth the quality tradeoff.
- Whether speculative decoding improves real efficiency.
- Whether a runtime configuration is merely fast in a benchmark or efficient over a full workload.
- Which local workloads are economically sensible compared with hosted inference.

The result is a repeatable method, not a fixed claim that local inference is always cheaper.

## Next Experiments

This first dashboard is a foundation. Future work includes:

- Comparing model quality, latency, and cost across several Qwen quantizations.
- Measuring complete agent tasks rather than only token streams.
- Separating server idle power from incremental inference power.
- Tracking cost by application or workflow.
- Comparing local results with specific hosted-model pricing at comparable quality levels.
