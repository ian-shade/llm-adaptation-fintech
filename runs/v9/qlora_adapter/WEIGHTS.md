# Adapter weights

This directory holds the QLoRA adapter trained for Condition B (Fine-tuned) and reused in
Condition C (Hybrid), as reported in Chapter 5 of the dissertation.

| File | Size | SHA-256 |
|---|---|---|
| `adapter_config.json` | 1,157 B | `79b853f46ec9121b6914b0b45e8c694d82c77862961f8cb2f542f1fd8e20bd71` |
| `tokenizer.json` | 11,421,892 B | `3fd169731d2cbde95e10bf356d66d5997fd885dd8dbb6fb4684da3f23b2585d8` |
| `adapter_model.safetensors` | 161,533,192 B | `e5647a6e97141b12739907aa39f42f4f13e8680e721a6b6795e582e50f8e593e` |

`adapter_model.safetensors` exceeds GitHub's 100 MB per-file limit, so it is stored with
Git LFS. If `adapter_model.safetensors` appears here as a short text file beginning
`version https://git-lfs.github.com/spec/v1`, install Git LFS and run `git lfs pull` to
fetch the real weights.

If the weights are not available at all, they can be reproduced: Section 7 of
`dissertation_full_pipeline_v9_1.ipynb` trains this adapter from
`data/3banks_trainalpaca769.jsonl` in roughly 0.50 GPU-hours on a Colab T4. Training used a
fixed seed, so the run is repeatable, though bit-identical weights are not guaranteed across
different GPU or library versions.

## Configuration

LoRA rank 16, alpha 32, dropout 0.05, applied to `q_proj`, `k_proj`, `v_proj`, `o_proj`,
`gate_proj`, `up_proj` and `down_proj` of `Qwen/Qwen2.5-7B-Instruct` loaded in 4-bit NF4.
Two epochs over 769 Alpaca-format rows, batch size 1 with gradient accumulation 8, learning
rate 2e-4, maximum sequence length 1,024.

## Loading

```python
from peft import PeftModel
model = PeftModel.from_pretrained(base_model, "runs/v9/qlora_adapter")
```

The base model must be `Qwen/Qwen2.5-7B-Instruct` in 4-bit NF4 with double quantisation and
float16 compute, and the chat template in `chat_template.jinja` must be the one applied — the
adapter was trained under it, and substituting the tokenizer's default template changes the
prompt the adapter sees.
