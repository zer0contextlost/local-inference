---
title: "Qwen 3.8 27B Hits r/LocalLLaMA: Real Quant Numbers Across 16GB to 32GB Rigs"
date: 2026-08-20
description: "r/LocalLLaMA's release-day thread for Qwen 3.8 27B, with quantization results from 16GB to 32GB VRAM rigs and the community's benchmark skepticism."
tags: ["Qwen", "quantization"]
---

<!-- SOURCE_THREAD: https://old.reddit.com/r/LocalLLaMA/comments/1vo9mj4/its_out/ -->

## A new Qwen, and immediate stress-testing

Qwen skipped a 3.7 release and went straight to 3.8, and the 27B model landed on r/LocalLLaMA under the thread title "IT'S OUT." Within hours the top comments were quantized numbers from home rigs rather than official benchmark charts, which tends to be the more useful signal anyway.

One user declared Qwen 3.8 27B the best local model they'd seen on the informal "flappy bird bench" that tends to resurface whenever a new model ships. Asked how it handled the (tongue-in-cheek) pelican bench, they followed up with a result from the official fp8 weights running across two 3090s: an animated SVG that took roughly half an hour of "thinking" before it rendered.

## What fits where

The more useful thread ran in the replies, where people reported numbers from real cards:

- A 16GB RX 9060 XT (32GB system RAM) ran Q3 in LM Studio. Switching to a Q3_K_M quant more than doubled throughput over the first attempt, and speed climbed further as context shrank. The user reported smooth performance up to 64K context while coding through Cline in VS Code.
- A 24GB M4 MacBook Pro managed 5 tokens/sec at Q4_K_M, prompting the obvious follow-up question of how much output quality a drop to Q3 costs.
- A 32GB RAM / 5070 Ti system got 4 tokens/sec on a Q3 XL quant. Another user with the same setup was curious whether their own numbers would land close.
- On a 5090, Q6 landed at 25 tokens/sec. That reads slow next to the same user's numbers for Qwen 3.6's 35B-A3B mixture-of-experts checkpoint, which they clocked closer to 200 tokens/sec. Fast enough that they're sticking with 3.6 MoE for now, running four Claude Code subagents through it with room for larger context windows each.
- A 4060 Ti owner confirmed the model runs there too.

The throughput gap between the dense 27B model and the 3.6 MoE variant is the most concrete tradeoff to come out of the thread. A dense 27B at a higher quant may buy more capability per parameter, but a mixture-of-experts model at a comparable active-parameter budget is going to move far more tokens per second on the same card. Which one is worth running depends on whether a given workload is bottlenecked on latency or on output quality.

## Benchmark skepticism, right on schedule

The bigger claims got picked apart almost immediately. Some commenters framed 3.8 27B as competitive with Claude Opus 4.6 Max on certain benchmarks, and others compared it to DeepSeek4 Flash 0731, with more than one commenter shorthanding it as "V4 Flash 0731." That would be notable for a model that fits on a single consumer GPU, but the pushback arrived just as fast. One commenter listed the assumptions being made outright: that the benchmarks are reliable, and that Qwen didn't optimize for them. Others pointed to the elephant in the room, native context length: one commenter guessed that would be the main gap against a model like Opus 4.6 Max, while conceding that even a 256K native window is plenty usable in practice.

There's institutional memory at work here too. One commenter brought up how confidently Qwen 3.6 27B was once called Opus 4.5-level, a claim that didn't hold up once people used it. The rough consensus by the end of the thread: expect a step up from 3.6, but treat the headline numbers as a starting point rather than a verdict until more people have run it against their own workloads. Gemma 4 got a mention in the same breath, brought up by a commenter as a reminder that coding leaderboards aren't the whole picture. Some people in the thread are running these models for writing and research, where Gemma still holds its own.

## The test that moved people

The most convincing result in the thread wasn't a benchmark at all. One user has a standing test for new releases: a single detailed prompt asking for a full Next.js ecommerce frontend for a bakery, with WhatsApp checkout and no backend, then watching what comes back with zero follow-up. Running Qwen 3.8 27B at UD-Q4_K_XL on a single RTX 3090, with 110K context, f16 KV cache, and no MTP, they got a result they said beat every other model or finetune they'd tried on the same prompt, without the broken Next.js warnings that usually show up. The detail that stood out most: when the model realized it didn't have vision, it built SVG placeholder graphics for the product cards on its own instead of leaving them blank, which the commenter joked was because it didn't have enough VRAM to spare. Vision benchmark numbers for the model drew similar praise, though the thread's overall mood was to wait for more people to run their own tests before trusting them.

*Source: [r/LocalLLaMA thread](https://old.reddit.com/r/LocalLLaMA/comments/1vo9mj4/its_out/) — "IT'S OUT"*
