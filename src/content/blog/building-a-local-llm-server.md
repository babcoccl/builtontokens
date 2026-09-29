---
title: "Building a Local LLM Server: The Tradeoffs Behind My Hardware Choices"
description: "This is not a PC-building guide. It is a practical account of the constraints, priorities, and tradeoffs behind a dual-GPU local inference server."
pubDate: 2026-10-01
updatedDate: 2026-10-01
tags:
  - local-llm
  - hardware
  - gpu
  - homelab
  - inference
draft: true
heroImage: '../../assets/blog-placeholder-3.jpg'
---

## Why I Built It

Local models are no longer just an interesting experiment. They can support private development workflows, agentic coding, offline experimentation, and predictable inference costs.

This post is not a component-by-component build tutorial. Instead, it explains what I optimized for, what constraints shaped the system, and where the compromises are.

## The Workload Came First

Before selecting parts, I defined the workload:

- Run capable local models in the roughly 27B to 30B range.
- Support long-context interactive work.
- Serve API-compatible inference to local tools and coding agents.
- Leave room to compare runtimes rather than committing to one application.
- Collect real performance and power data instead of relying only on published benchmarks.

The important question was not “What is the fastest gaming PC?” It was: **what hardware makes the intended inference workload practical and repeatable?**

## The Central Constraint: VRAM

For local inference, GPU memory is often more important than raw GPU compute.

My system uses two NVIDIA GeForce RTX 5060 Ti cards with 16 GB of VRAM each, giving the machine 32 GB of total GPU memory across both devices. That capacity makes mid-quantized models around the 30B class practical while still allowing meaningful context windows.

The tradeoff is that two consumer GPUs do not behave like one unified 32 GB accelerator. The runtime must know how to split model layers and manage work across devices, and performance varies by runtime, model format, context length, and offload configuration.

## The Parts That Reflected My Priorities

### GPUs: Capacity Before Peak Benchmark Numbers

The GPU decision was driven primarily by usable VRAM per dollar and the ability to distribute a model across two cards.

A single faster GPU with less memory would have been simpler, but it would have narrowed the set of models and context lengths I could run. Two 16 GB cards introduce operational complexity, but they expand the experiments the server can support.

### CPU and System Memory: Supporting Cast, Still Important

For this machine, the CPU is not the primary inference engine. It still matters for:

- Model loading and preprocessing.
- CPU-offloaded layers when testing larger models.
- Feeding GPUs without avoidable bottlenecks.
- Running containers, monitoring, development tools, and supporting services.
- Keeping the server usable while an inference workload is active.

System RAM also provides insurance. Model files, caches, development tools, Docker services, and multiple concurrent experiments can make a “GPU-only” sizing decision look shortsighted quickly.

### Storage: Model Libraries Are Real Infrastructure

Local model experimentation accumulates files quickly:

- Multiple quantizations of the same model.
- Draft models for speculative decoding.
- Runtime-specific cache files.
- Benchmark logs and exported metrics.
- Docker volumes and observability data.

Fast NVMe storage reduces friction when loading and swapping models, but capacity matters just as much. A local model library is closer to a software artifact repository than a typical gaming installation.

### Power, Cooling, and Physical Layout

A multi-GPU inference system changes the usual desktop priorities.

Under inference load, power draw, heat, cable routing, airflow, and PCIe slot spacing become operational concerns. Sustained inference is different from a short benchmark run: the system needs to remain stable, monitorable, and tolerable to operate over long sessions.

## What I Deliberately Did Not Optimize For

This system was not designed to be:

- The lowest-cost machine possible.
- A universal workstation for every GPU task at once.
- A gaming benchmark leader.
- A replacement for cloud infrastructure at every scale.
- A silent, zero-maintenance appliance.

For example, image-generation workloads and LLM workloads compete for the same GPU memory. In practice, the machine works best when I deliberately allocate it to one heavy workload at a time.

## The Real Outcome

The build gave me an experimentation platform rather than a single-purpose appliance. It can run useful local models, expose APIs to other tools, and generate the telemetry needed to decide whether local inference is economically and operationally worthwhile.

The hardware was only the beginning. The next question was which runtime could make the most of it.
