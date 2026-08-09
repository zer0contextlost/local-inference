---
title: "Self-Hosted LLMs for Fiction and Roleplay Writing: Hardware, Models, and Stack"
date: 2026-08-09
description: "Why homelabbers run local LLMs for fiction and roleplay writing instead of a hosted API, what 'uncensored' and 'abliterated' actually mean, and which models r/LocalLLaMA, r/LocalLLM, and r/SillyTavernAI actually recommend for it."
tags: ["llm", "homelab", "self-hosting", "gpu"]
---

Hosted chat APIs apply content filters tuned for a general audience, which
means anything even adjacent to mature fiction — violence, adult themes,
morally gray characters — can get flagged, refused, or silently softened
mid-story. That's the main reason r/LocalLLaMA and r/LocalLLM both have a
whole recurring thread genre around "best uncensored model for writing."
Run the model on
your own hardware and there's no third-party filter sitting between your
prompt and the output, no chat log leaving the house, and no surprise
policy change breaking a story you're partway through.

This is a rundown of what that setup actually looks like: what
"uncensored" means technically, what hardware tier you need for it, and
which tools people are actually running.

## "Uncensored" vs "abliterated"

Two different techniques get lumped under the same label:

- **Fine-tuned uncensored models** are retrained on datasets that don't
  include the refusal examples base models are trained on, so they never
  learned to say no in the first place. Dolphin and Hermes are the
  best-known families built this way.
- **Abliterated models** start from a normal instruct model and have the
  specific internal direction responsible for refusal mathematically
  identified and removed post-training — the base model's other behavior
  (instruction-following, reasoning) is mostly left intact. You'll see
  this as "-abliterated" in a model name on Hugging Face.

Neither one is a jailbreak prompt or a system-prompt trick. Both change
the weights themselves, which is why they behave consistently instead of
occasionally snapping back to a refusal.

## Picking a size for your GPU

Community consensus tracks pretty closely with general GGUF sizing —
nothing about creative-writing models needs unusual VRAM headroom, you're
choosing quant level against the same budget as any other local model:

| VRAM | Comfortable tier | Notes |
|---|---|---|
| 8 GB | 7-9B, Q4_K_M–Q5_K_M | fits with headroom for KV cache |
| 12 GB | 8-14B, Q4–Q5 | sweet spot for a single consumer GPU |
| 16-24 GB | 13-24B, Q5–Q6 | noticeably better prose coherence over long context |
| 48 GB+ | 70B, Q4–Q5 | multi-GPU or a workstation card; best character/plot consistency |

Rule of thumb across all tiers: budget the quoted quant size for weights,
then add 15-25% on top for KV cache and context overhead before you call
a card "enough." Newer quant formats like IQ4_XS and the GGUF-V3 imatrix
builds do a better job preserving activation precision than the older
plain Q4 quants, which matters most on the smaller end of this table
where every bit of precision counts.

## Models people actually cite

Recurring names across current LocalLLaMA, LocalLLM, and
SillyTavern-community roundups, by tier. A recent r/LocalLLM thread asking
specifically for uncensored-writing recommendations is a good snapshot of
where consensus sits right now:

- **Small (7-9B)** — Dolphin 3.0 Mistral and OpenHermes 2.5 Mistral for
  modest gaming GPUs, plus Anubis-8B, a smaller cut of the Anubis
  roleplay-tuned line below. Best for quick scenes rather than long
  multi-chapter continuity.
- **Mid (12-36B)** — MythoMax-L2-13B is the long-running default a lot of
  comparisons still benchmark against; Qwen 3.5 gets recommended broadly
  across use cases including this one; Nous Hermes 2 Yi 34B and Dan's
  PersonalityEngine (24B) both add stronger instruction-following and
  longer-context handling on top of the same roleplay/story focus. TheDrummer's
  Cydonia-24B and Skyfall-31B/36B come up repeatedly as the strongest
  budget picks under ~24GB — the tradeoff noted in practice is a tendency
  to over-pad prose and echo earlier lines back rather than move a scene
  forward.
- **Large (70B+)** — Midnight Rose, Midnight Miqu, and Anubis-70B show up
  repeatedly for prose quality and keeping a character consistent across a
  long story, and reportedly hold up reasonably even at lower quants if
  you're VRAM-constrained. Behemoth-ReduX-123B goes past what a single
  consumer GPU handles at reasonable throughput but gets cited for ad hoc
  scenes when the hardware's there.
- **MoE (mixture-of-experts)** — GLM-4.5-iceblink (106B total / ~12B
  active parameters) is a newer entrant in these threads: it needs enough
  RAM/VRAM to hold the full model but runs at roughly the *active* param
  count's speed, which is a different tradeoff than a same-size dense
  model.

**One caution actually worth repeating**: a fine-tune from HauhauCS
(`Qwen3.6-35B-A3B-Uncensored-HauhauCS-Aggressive`) that was getting
recommended in that thread was called out by other commenters as having a
plagiarism problem with its training data — worth checking a model's
community reception past the download count before trusting it, not just
taking the top comment at face value. "Heretic" (an abliteration tool
distinct from any one model) came up as the preferred alternative approach
for de-censoring a base model yourself rather than trusting someone else's
fine-tune.

Also worth knowing: r/LocalLLM and r/LocalLLaMA get general questions on
this topic, but commenters consistently redirect to **r/SillyTavernAI**
specifically for roleplay-model comparisons — that's the more concentrated
community if you want ongoing recommendations rather than a one-off
thread.

Model rankings in this space move fast and are subjective by nature —
treat any specific name as a starting point to test against your own
prompts, not a settled answer.

## The stack

Three pieces, same pattern regardless of which model you land on:

1. **Inference backend** — `llama.cpp`/KoboldCpp or Ollama for GGUF
   models. KoboldCpp is the more common pick specifically in the
   roleplay/story community because its API and context handling were
   built with that use case in mind.
2. **Frontend** — SillyTavern is the default here: persistent character
   cards, world-info/lorebooks for long-running stories, and a chat UI
   built around back-and-forth roleplay rather than one-shot Q&A.
3. **Quant format** — GGUF for anything running on llama.cpp-family
   backends; check a model's Hugging Face page for which imatrix/quant
   variant fits your VRAM tier from the table above.

All three run fine in an LXC container or VM with GPU passthrough on a
Proxmox box, same pattern as any other GPU workload here — the model
itself doesn't care that it's virtualized, it just needs the VRAM handed
through cleanly.

## Why local over hosted here specifically

Beyond content policy, the other reasons that come up constantly in these
threads: no per-token cost for long writing sessions, no chat history
sitting on a third party's servers, and no risk of a provider changing
its acceptable-use policy mid-project and orphaning a long-running story.
The tradeoff is upfront hardware cost and slower iteration on model
quality compared to a frontier hosted model — for short-form or
general-purpose writing that tradeoff often isn't worth it. For a
long-running, private, mature-themed story, it usually is.

*Source: [r/LocalLLM thread](https://old.reddit.com/r/LocalLLM/comments/1vibj9o/llm_for_nsfw_writing/) — "LLM for NSFW writing"*
