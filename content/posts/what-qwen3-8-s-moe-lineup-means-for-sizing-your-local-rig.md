---
title: "What Qwen3.8's MoE Lineup Means for Sizing Your Local Rig"
date: 2026-08-09
description: "Qwen3.8-27B and 3.8-Max landed this week. r/LocalLLaMA's reaction says more about active-parameter counts and offload rigs than about the models themselves."
tags: ["qwen", "moe", "vram-sizing", "local-inference"]
---

<!-- SOURCE_THREAD: https://old.reddit.com/r/LocalLLaMA/comments/1ve0psn/qwen3827b_announced_alongside_qwen38max/ -->

Qwen released 3.8-27B alongside a much larger 3.8-Max, and the r/LocalLLaMA thread reacting to it turned into a decent snapshot of what people are actually planning to run on their own hardware. Buried under the celebration ("still waiting for 3.8 35b a3b, but excited for the 27b version," one top comment put it) is a pretty clear map of which model shapes this community wants and why.

## The sizes people are actually asking for

Scan the thread and a pattern shows up fast. Nobody's asking for bigger dense models. They're asking for specific MoE (Mixture of Experts) configurations:

- A 35B-A3B, which several commenters were holding out for over the 27B dense release
- A5B, called out as the sweet spot for 6GB VRAM cards
- 35B-A7B, described by one commenter as "the sweet spot of speed and smart"
- An 80B-A3B floated as a wishlist item, with replies pointing to "Coder Next" as the closest existing analog

The naming convention (total params-A-active params) matters more to local hosters than raw parameter count. A 35B-A3B model only computes 3B parameters per forward pass even though 35B worth of weights exist somewhere on disk or in RAM. That's the entire appeal for anyone without a rack of A100s.

## Why active params beat total params

One reply in the thread lays out the tradeoff directly: too few active parameters and the model stops being much smarter than a dense model a fraction of its size, no matter how many total parameters it has. The same commenter pointed to Qwen 3.5 27B beating the 122B-A10B Qwen 3.5 release on coding tasks specifically, despite the 122B model having far more total weights. Total size buys you breadth of knowledge stored across experts. Active size buys you reasoning quality on the tokens you're generating right now. For local hosting, active parameter count is close to what determines your tokens/sec, while total parameter count determines how much RAM or VRAM you need to hold the whole thing.

That split is exactly what makes MoE offloading work. You don't need all the experts in VRAM simultaneously, only the ones that get activated for the current token.

## A concrete offload setup from the thread

One commenter described running a 122B-A10B model (a different release, same Qwen 3.5 generation referenced above) on a 3090 with 24GB VRAM plus roughly 50GB more sitting in dual-channel DDR4 on a Ryzen 9 system. The active experts get computed on the GPU, the rest of the weight matrix sits in system RAM and gets touched only when a token routes to it. Their point was that this isn't exotic workstation hardware. It's a five-year-old gaming PC with a lot of system RAM added. If Qwen 3.8 ships an A3B or A5B variant at a similar total size, that same rig profile becomes the target build for a lot of people in the thread.

## Planner and executor as two separate models

The other setup worth noting: one commenter runs DeepSeek V4 Flash quantized to Q2KXL for planning, and switches to Qwen 3.6 27B at Q8 as the executor once a plan exists. They're using an agent harness called "pi" and doing the swap manually, typing "proceed with the plan" once the plan model hands off. It's not automated in their setup, but the idea generalizes: a cheap, heavily quantized model for the reasoning/planning step where precision matters less, and a higher-quality quant of a smaller model for the execution step where you're actually generating final output. Anyone building a two-GPU or GPU-plus-CPU-offload rig should think about this as two separate model budgets rather than one.

## What to watch for

There's also a passing mention of "strix" (AMD's Strix Halo APU line with unified memory) as hardware some commenters expect MoE releases to specifically target, on the theory that unified memory architectures sidestep the VRAM-vs-RAM split entirely. If Qwen or anyone else ships a 35B-A3B or A5B model in the next few months, that's the class of hardware, and the offload pattern described above, worth having ready.
