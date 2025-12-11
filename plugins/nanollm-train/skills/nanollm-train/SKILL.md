---
name: nanollm-train
description: End-to-end recipe for training LLMs from scratch or adapting them. Use this skill whenever the user asks about designing, debugging, scaling, or aligning transformer LLMs. Prioritizes correctness, minimal abstractions, and avoiding expensive mistakes.
license: refer to README.md
---

## Purpose

Guide the agent to build **correct, scalable, debuggable LLM pipelines** inspired by nanoGPT + nanochat: start tiny, validate every component, scale only one axis at a time, then add SFT/RL and evaluations.

---

# Core Principles (Critical)

* **Start tiny → scale later.** All pipelines must first work with micro-models (1–4 layers) and tiny datasets.
* **Avoid silent bugs.** Attention mask, tokenizer shift, BOS/EOS handling, data shuffling, logging, and checkpointing MUST be explicitly validated.
* **One new complexity at a time.** Never scale model + data + hardware simultaneously.
* **Metrics matter.** Pretrain loss + eval harness (perplexity + tasks) must be established early.
* **Cost awareness.** Scaling laws: ~20–30 tokens per parameter for pretraining.

---

# Workflow

## 1. Define Scope & Budget

* Model classes: **base LM**, **instruct/SFT**, **RL-aligned**, **domain-specific**.
* Pick feasible parameter counts:

  * toy (1–50M),
  * small (100–500M),
  * medium (1–3B),
  * large (7B+).
* Compute required tokens: params × (20–30 tokens).

## 2. Data + Tokenizer Verification

* Inspect raw text. Remove duplicates, garbage, broken unicode, etc.
* Choose tokenizer:

  * micro-scale: **char-level**;
  * real model: **BPE**.
* Validate:

  * BOS/EOS rules
  * newline preservation
  * special tokens
  * vocab frequency distribution
  * train/val contamination prevention.

## 3. Build Minimal End-to-End GPT (nanoGPT style)

Before any scaling:

* Use tiny dataset (Shakespeare or 1–5MB slice of real data).
* Tiny transformer (2–4 layers, 128–256 dim).
* Validate:

  * loss at init ≈ log(vocab)
  * correct causal mask
  * correct token shift
  * overfit 1 batch → loss → 0
  * reproducibility with fixed seed
  * checkpoint/resume correctness
* Sample outputs to visually check tokenization + autoregressive behavior.

## 4. Medium-Scale Real Data Test

* Move to 100MB–1GB corpus.
* Keep hardware + optimizer fixed.
* Validate scaling:

  * model ↑ → loss ↓
  * data ↑ → val loss ↓
* Test bf16/FP16 stability, DDP, dataloader throughput.

## 5. Scale Systematically

**Only one axis at a time:**

**Axis A: Model size**
Increase depth/width gradually (e.g., 124M → 350M → 1B).

**Axis B: Data tokens**
Target compute-optimal tokens for chosen N params.

**Axis C: Hardware**
Scale single GPU → multi-GPU → 8×H100 node.

Use nanochat’s design philosophy: reproducible shell scripts (`speedrun.sh`), simple configs, full pipeline in one machine where possible.

## 6. Build Evaluation Harness Early

* Loss curves (train + multiple val shards).
* Tasks: GSM8K, MMLU, ARC, HumanEval (choose subset).
* Add interactive “chat UI” for qualitative detection of hallucinations or data issues.
* Regression tests: fixed prompts → deterministic expected patterns.

## 7. Add Instruction Tuning (SFT)

* Create high-quality prompt→response data.
* Start tiny (1k samples) — MUST overfit cleanly.
* Then scale.
* Evaluate effect vs BASE.

## 8. Preference Optimization (RL, DPO, RRHF)

* Tiny preference set first.
* Monitor reward hacking.
* Track BASE/SFT/RL versions side-by-side (as nanochat does).
* Validate no degradation in reasoning benchmarks.

## 9. Inference & Deployment

* Choose decoding modes (greedy, nucleus, etc.).
* Validate KV-cache and max-seq behavior.
* Produce quantized variants (4/8-bit).
* Monitor drift + user feedback → iterate SFT/RL loops.

---

# When to Apply This Skill

Use **llm-trainer** skill when the user asks for:

* training a GPT from scratch (small or large)
* dataset prep/tokenizer correctness
* architecture sizing / scaling law decisions
* debugging unstable loss, NaNs, or bad generations
* building an evaluation harness
* designing SFT / RL / preference optimization
* safe deployments and inference tuning

---

# Guarantees / Constraints

* Prefer minimal code/architecture, nanoGPT-style clarity.
* Never assume correctness without explicit verification.
* Always propose a *tiny-scale test* before a costly run.
* Avoid hidden complexity: no magical abstractions, every step inspectable.
