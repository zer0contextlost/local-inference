---
title: "Qwen3.8 27B at 1-Bit: What r/LocalLLaMA's 'Brain Damage' Quant Actually Reveals"
date: 2026-08-23
description: "A 1-bit GGUF of Qwen3.8 27B went viral on r/LocalLLaMA for its jokes. Unsloth's real accuracy numbers on it are the actual story."
tags: ["quantization", "qwen3.8", "llama.cpp"]
---

<!-- SOURCE_THREAD: https://old.reddit.com/r/LocalLLaMA/comments/1vtr3h0/ladies_and_gentlemen_i_present_to_you_qwen38_27b/ -->

Someone quantized Qwen3.8 27B down to 1-bit and posted screenshots of it under the label "brain damage quant." The screenshots are what you'd expect: the model refuses to answer a simple question about the latest Python version and instead demands the user tell *it*. Commenters piled on with jokes about giving the thing "lifelines" like a game show contestant, or handing it sudo access as a dare. It's funny. It's also a decent excuse to talk about what actually happens when you push GGUF quantization past the point where a model still works.

## The numbers behind the joke

Daniel Han from Unsloth showed up with real data instead of vibes. Using what he described as a "Divergence-300 @ 32" test suite, which runs quants against actual long-running tasks and measures divergence from the BF16 baseline over 32 tokens, the results were:

- 1-bit: 8% accuracy
- UD-Q2_K_XL: 21% accuracy
- UD-Q4_K_XL: 68% accuracy

Divergence from BF16 at 1-bit hits 92% over that 32-token window. His point is that this gets worse the longer the conversation runs, not better, since errors compound. At extreme quantization, a model retains general knowledge but loses the ability to reliably call tools, and the failure mode isn't graceful. It either stops calling tools entirely, calls them at the wrong time, or gets stuck looping the same call over and over. His recommendation for anyone who hits that looping behavior is to set `presence_penalty` to 1.5 or higher. His bottom line: 1-bit is a proof of concept for seeing how far you can compress a model before it stops functioning, not something to point at real coding or agentic work. UD-Q2_K_XL is the floor he'd actually suggest using.

Several commenters found that grimly amusing given how casually agentic pipelines hand models shell access. One joked that a 1-bit model has about a 2% chance of deciding to `rm -rf` your home directory when nobody double-checks the tool call, and another called the whole setup "a fancy version of russian roulette." That's a joke, but it's downstream of a real number: 8% accuracy on a 32-token task means the model is wrong far more often than it's right, and it doesn't know which one it's doing.

## What people are actually running instead

The more useful thread underneath the memes is what quant level people have found actually holds up.

Artistic_Okra7288 reported good results with Q3_K_XL combined with q8_0 KV cache quantization, running at full native context with vision support. The catch was speculative decoding (mtp) tanking throughput badly, dropping from roughly 1500 prompt tokens/sec and 44 tokens/sec generation down to around 123pp and 1.28tg. The fix ended up being setting `fit=off`, after which switching to a `draft-mtp,ngram-mod` combination got speeds back to a reasonable ~833 tokens/sec prompt processing and ~40.5 tokens/sec generation, with a draft acceptance rate around 0.59. Worth knowing if you're pairing MTP-style speculative decoding with KV quantization on this model and see performance fall off a cliff for no obvious reason. `fit` is one of the first settings to check.

Express_Quail_1493 has been coding against UD-Q2_K_XL and says it holds up "surprisingly well" but caps usage under roughly 126k context, expecting degradation past 128k. DeathByPain runs the same quant with q8 KV cache at a 120k context limit through VS Code Copilot, which compacts context around 80k, and reports a long single-conversation session on a Python/Tkinter app going well.

## Not everyone's convinced it's worth the disk space

WhoRoger pushed back on the whole exercise in the comments, questioning why anyone bothers with 1-bit or even Q2 quants outside the largest models, calling it a waste of electricity and Hugging Face storage unless you're dealing with an actual binary-weight model architecture.

*Source: [r/LocalLLaMA thread](https://old.reddit.com/r/LocalLLaMA/comments/1vtr3h0/ladies_and_gentlemen_i_present_to_you_qwen38_27b/) — "Ladies and gentlemen I present to you Qwen3.8 27b 1bit brain damage quant"*
