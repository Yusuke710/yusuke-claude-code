# LLM Trainer Skill

A disciplined, lightweight recipe for training, scaling, and aligning LLMs without silent bugs or wasted compute.

## What It Does

Claude automatically uses this skill when the user asks about LLM training, debugging, scaling, or alignment.
It provides a minimal, reliable, nanoGPT-inspired workflow with:

Tiny-to-large scalable training pipelines

Tokenizer & data validation guidance

Debug-first methodology (overfit tests, shift checks, masking verification)

Safe scaling rules for model size, data, and hardware

Built-in patterns for SFT, RL/DPO, and evaluation harnesses

Clear tooling advice for reproducible training

## Usage

Examples of when Claude will apply this skill:

“Help me design a 1B-parameter GPT training plan.”

“Give me a scaling roadmap from 100M → 3B params.”

“How do I structure SFT and RLHF for my domain-specific assistant?”

Claude will produce simple, correct, inspectable guidance that avoids expensive mistakes.

Authors
Borrowed [A Recipe for Training Neural Networks](https://karpathy.github.io/2019/04/25/recipe/) converted to LLM version using [ChatGPT(check conversation)](https://chatgpt.com/share/693a17fb-b410-800d-b840-cf38aef3f791).
