---
title: "Running Kimi K3 Locally: A 16x GB10 Cluster and the Community's Alternatives"
date: 2026-08-09
description: A hobbyist ran the full Kimi K3 model across 16 NVIDIA GB10 nodes, and the ensuing r/LocalLLaMA thread digs into the real throughput, hardware costs, and cheaper single-node alternatives commenters proposed.
tags: ["LLMs", "Kimi K3", "GPU Clusters", "Self-hosting"]
---

<!-- SOURCE_THREAD: https://old.reddit.com/r/LocalLLaMA/comments/1vfl525/kimi_k3_full_model_running_on_16x_gb10_cluster_at/ -->

## Overview

A recent r/LocalLLaMA post showed the full Kimi K3 model running across a cluster of 16 NVIDIA GB10 nodes, the same Grace Blackwell Superchip found in devices like the DGX Spark. The build runs its monitoring dashboard off a Raspberry Pi, which amused a string of commenters given the price gap between the Pi and the rest of the rig; the joke spun off into a side debate about whether Raspberry Pis are even still worth buying given how well used corporate desktops compare on price and speed.

Each GB10 node carries 128GB of unified memory, putting the cluster's total around 2TB, though commenter Caffdy pointed out that only about 110-112GB per node is actually usable once the OS and other processes take their share.

The thread's title advertises 20+ tokens per second, but commenter fastheadcrab noted that OP said elsewhere the real number without speculative decoding is 6-7 tps, considerably slower than the headline figure.

## Technical Considerations

### Networking Overhead

Commenters fastheadcrab and FullstackSensei both pointed to networking overhead as the likely culprit behind the modest throughput. Coordinating inference across 16 separate nodes adds communication cost that a single-box setup wouldn't have.

### Hardware Cost

FullstackSensei estimated the cluster at roughly $64-80k in hardware, and CYTR_ noted that outside the US, pricing (and taxes) can push a comparable setup to $75-120k depending on the country. FullstackSensei went further, calling the OP's approach "wasteful" given what similar money could buy in a single-node build.

## Alternative Approaches

### A Single-Node High-End Build

FullstackSensei, who said they were reworking their own homelab to run K3 locally, sketched an alternative: a dual Xeon 8480 board with CPUs (~$4k), 1.5TB of DDR5-4800 RAM (~$15k), plus a power supply, coolers, case, and fans (rounding to ~$20k), and around $6k more for four AMD R9700 GPUs to add 128GB of VRAM, about $26k all-in versus the cluster's $64-80k+. They expect this to land somewhere between 7-15 tps for token generation depending on software optimization, and noted that Sapphire Rapids' AMX support speeds up both token generation and prompt processing, and that each CPU's roughly 200GB/s real-world memory bandwidth is within about 10% of a single GB10's (~215GB/s).

### Software Optimization

The same commenter mentioned that the R9700 now has vllm support, and that a vllm fork exists which adds NUMA awareness and CPU offloading of experts, both aimed at squeezing out more throughput without additional hardware spend.

## Community Perspectives

Reactions to the cluster were mixed. FullstackSensei argued that if you have $64-80k to throw at hardware like this, break-even math probably isn't the point, a sentiment that also answered Own_Calligrapher8508's question about whether the setup could ever pay for itself.

Others focused on why anyone would go local at these prices at all. FullstackSensei and commenter caowcaow discussed regulated industries where data can't leave the premises, with caowcaow noting that some of the same organizations keeping PII strictly on-prem are nonetheless comfortable sending data to frontier models through custom cloud deployments, a contradiction a few commenters found hard to square. For orgs in that position, a fully local frontier-capable rig sidesteps the question entirely.

A side conversation also broke out about whether a future high-memory Mac Studio could compete, with commenters split on whether Apple could hit both the 1.5-2TB memory range and 20+ tps on a model this size, and general skepticism about what such a machine would even cost.

*Source: [r/LocalLLaMA thread](https://old.reddit.com/r/LocalLLaMA/comments/1vfl525/kimi_k3_full_model_running_on_16x_gb10_cluster_at/) — "Kimi K3 full model running on 16x GB10 cluster at 20+tps"*
