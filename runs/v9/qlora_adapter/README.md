---
base_model: Qwen/Qwen2.5-7B-Instruct
library_name: peft
pipeline_tag: text-generation
tags:
- base_model:adapter:Qwen/Qwen2.5-7B-Instruct
- lora
- transformers
---

# QLoRA adapter — UK bank annual reports (Condition B / C)

A LoRA adapter over `Qwen/Qwen2.5-7B-Instruct`, trained on instruction pairs written from the
2023 core financial statements of HSBC Holdings plc, Lloyds Banking Group plc and NatWest Group
plc. It is the fine-tuning half of the comparison reported in *Fine-Tuning vs RAG vs Hybrid: A
Comparative Framework for Domain-Specific LLM Adaptation in Financial Services* (MSc
dissertation, Manchester Metropolitan University, 2026).

The adapter exists to be measured, not deployed. It is published so the reported experiment can
be reproduced.

## Details

- **Developed by:** Ihsan Abourshaid, supervised by Prof. Keeley Crockett
- **Model type:** LoRA adapter (PEFT 0.20.0) for a causal language model
- **Language:** English
- **Base model:** `Qwen/Qwen2.5-7B-Instruct`, loaded in 4-bit NF4 with double quantisation and
  float16 compute
- **Licence:** the adapter weights are covered by the MIT licence of the repository below. The
  base model carries its own licence from its publisher — check Qwen's model card before
  redistributing anything merged with it.
- **Repository:** https://github.com/ian-shade/llm-adaptation-fintech

## Training

| | |
|---|---|
| Data | `data/3banks_trainalpaca769.jsonl` — 769 Alpaca-format rows |
| LoRA | rank 16, alpha 32, dropout 0.05 |
| Target modules | `q_proj`, `k_proj`, `v_proj`, `o_proj`, `gate_proj`, `up_proj`, `down_proj` |
| Schedule | 2 epochs, 194 optimiser steps, batch size 1 with gradient accumulation 8 |
| Optimiser | learning rate 2e-4, maximum sequence length 1,024 |
| Cost | 1,805 s ≈ 0.50 GPU-hours on a Colab T4 |

Section 7 of `dissertation_full_pipeline_v9_1.ipynb` reproduces the training. The seed is fixed,
so the run repeats, though bit-identical weights across different GPUs or library versions are
not guaranteed.

## Use

```python
from peft import PeftModel
model = PeftModel.from_pretrained(base_model, "runs/v9/qlora_adapter")
```

Apply `chat_template.jinja` from this directory. The adapter was trained under that template, and
substituting the tokenizer's default changes the prompt it sees.

## Results

On 240 held-out objective questions, scored identically across all four conditions:

| Condition | Objective accuracy |
|---|---|
| Baseline (no adapter, no retrieval) | 22.1% |
| **Fine-tuned (this adapter, no retrieval)** | **27.5%** |
| RAG (no adapter, top-6 retrieval) | 64.2% |
| Hybrid (this adapter + retrieval) | 66.3% |

The adapter adds about five points without retrieval and about two with it — and the hybrid's
lead over RAG alone is not statistically significant (exact McNemar p = 0.42).

## Limitations

**It removes refusal.** On 30 questions with no answer in the corpus, RAG abstains on 23; both
adapter conditions abstain on 0 and answer confidently instead (Fisher p = 1.7 × 10⁻¹⁰). This is
the most important thing to know about these weights: fine-tuning on a set of confidently
answered pairs taught the model to answer, including when it should not.

**It learned register, not facts.** On open-ended questions the adapter raises BERTScore while
scoring below the unadapted baseline on judged correctness — it writes like a financial analyst
without knowing more than one.

**The domain is narrow.** Three UK banks, one financial year, the core statements only. Nothing
here transfers to other issuers, years or jurisdictions.

**Not financial advice.** This is research output. It hallucinates — measuring that is the point
of the study — and no answer it produces should inform a financial decision.
