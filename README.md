# cot-med-summarizer

Chain-of-Thought Supervision for Explainable and Efficient Medical Summarization with Small Language Models.

This repository contains the datasets, fine-tuned LoRA adapters, training scripts, data generation scripts, and evaluation code associated with our research on reasoning-supervised small language models (SLMs) for histopathology report summarization.

---

## Overview

A central challenge in deploying language models for clinical NLP is the tension between model capability, computational cost, data privacy, and interpretability. This work investigates whether structured Chain-of-Thought (CoT) supervision can compensate for the implicit domain knowledge acquired through large-scale pretraining, enabling compact SLMs to achieve competitive performance with much larger models at a fraction of the computational cost.

We fine-tune models from the Qwen family at two scales — Qwen-2.5-0.5B (SLM) and Qwen3-0.6B / Qwen3-32B (controlled cross-scale comparison) — using LoRA on three dataset variants: Summary-Only, Single-CoT, and Diverse-CoT. The Diverse-CoT trained 0.6B model achieves a clinical-term hit rate of 72.2% and a BERTScore F1 of 0.892, within 0.1 percentage points and 0.009 respectively of the fine-tuned 32B model, while operating at 2.3× faster inference, requiring 28× less GPU memory, and emitting 3.0× less carbon per inference run.

All training data is synthetically generated using GPT-4.1-mini under expert-validated prompt templates. No real patient data was used at any stage.

---

## Repository Structure

```
cot-med-summarizer/
│
├── Dataset/                          # Histopathology summarization dataset (JSONL)
│   ├── train_cot.jsonl               # Single-CoT training split (4,950 records)
│   ├── val_cot.jsonl                 # Single-CoT validation split (550 records)
│   ├── train_diverse_cot.jsonl       # Diverse-CoT training split (4,488 records)
│   ├── val_diverse_cot.jsonl         # Diverse-CoT validation split (792 records)
│   ├── train_summary.jsonl           # Summary-Only training split (4,950 records)
│   ├── val_summary.jsonl             # Summary-Only validation split (550 records)
│   └── README.md                     # Dataset documentation
│
├── FineTuned_Adapters/               # LoRA adapter weights for all fine-tuned models
│   ├── qwen2.5_0.5B_MedReason_adapter/
│   ├── qwen2.5_0.5B_Med16Reason_adapter/
│   ├── qwen2.5_0.5B_MedSummary_adapter/
│   ├── qwen3_0.6B_cot_adapter/
│   ├── qwen3_0.6B_diversecot_adapter/
│   ├── qwen3_0.6B_summary_adapter/
│   ├── qwen3_32B_cot_adapter/
│   ├── qwen3_32B_diversecot_adapter/
│   ├── qwen3_32B_summary_adapter/
│   └── README.md                     # Adapter documentation and usage
│
├── CoTSingleData.py                  # Generates Single-CoT dataset records
├── CoTDiverseData.py                 # Generates Diverse-CoT dataset records
│
├── LoRA_Summary_Code.ipynb           # Fine-tuning: Qwen-2.5-0.5B, Summary-Only
├── LoRA_Reasoning_Code.ipynb         # Fine-tuning: Qwen-2.5-0.5B, Single-CoT / Diverse-CoT
├── FT_Code_qwen32b_cot.ipynb         # Fine-tuning: Qwen3-32B, Single-CoT
├── FT_Code_qwen32b_diversecot.ipynb  # Fine-tuning: Qwen3-32B, Diverse-CoT
├── FT_Code_qwen32b_summary.ipynb     # Fine-tuning: Qwen3-32B, Summary-Only
├── FT_Code_qwen600m_diversecot.ipynb # Fine-tuning: Qwen3-0.6B, Diverse-CoT
├── FT_Code_qwen600m_summary.ipynb    # Fine-tuning: Qwen3-0.6B, Summary-Only
│
├── qwen3_eval_pipeline_h100.ipynb    # Full evaluation pipeline (H100, all 8 Qwen3 variants)
│
└── README.md                         # This file
```

---

## Models

### Primary SLM Comparison (Qwen-2.5-0.5B, NVIDIA T4)

Three model variants were trained on the original Qwen-2.5-0.5B base:

| Model | Training Config | ROUGE-L F1 | BERT F1 | Perplexity | GPT-4.1 Score | GPT-4.1 Blind Picks |
|---|---|---|---|---|---|---|
| Qwen2.5-0.5B (base) | None | 7.11 | 56.77 | 36.97 | 4.66 | 0 |
| MedSummary | Summary-Only | 33.56 | 73.30 | 15.17 | 7.79 | 21 |
| MedReason | Single-CoT | 39.59 | 75.45 | 4.14 | 8.14 | 25 |
| **Med16Reason** | **Diverse-CoT** | **43.00** | **76.06** | **6.40** | **8.99** | **43** |
| AlpaCare-7B | external baseline | 11.85 | 57.35 | 29.51 | 7.58 | 11 |
| BioMistral-7B | external baseline | 11.67 | 58.19 | 50.87 | 5.94 | 0 |
| BioMedLM-2.7B | external baseline | 8.20 | 49.00 | 18.47 | 5.57 | 0 |

Evaluated on 550 held-out benchmark samples with expert-written reference summaries. GPT-4.1 blind picks are out of 550 total comparisons.

### Cross-Scale Comparison (Qwen3-0.6B vs. Qwen3-32B, NVIDIA H100)

A controlled cross-scale evaluation using the Qwen3 model family, holding architecture, training data, fine-tuning procedure, and evaluation protocol constant across scale:

| Scale | Model | Training | ROUGE-L F1 | BERT F1 | Hit Rate | Tok/s | VRAM (GB) | CO2 (g) |
|---|---|---|---|---|---|---|---|---|
| SLM (0.6B) | base_0.6B | None | 0.164 | 0.789 | 46.6% | 32.1 | 1.3 | 4.87 |
| SLM (0.6B) | ft_0.6B_summary | Summary-Only | 0.159 | 0.843 | 20.5% | 21.4 | 0.8 | 0.63 |
| SLM (0.6B) | ft_0.6B_cot | Single-CoT | 0.203 | 0.794 | 35.6% | 21.6 | 16.8 | 6.42 |
| SLM (0.6B) | **ft_0.6B_diversecot** | **Diverse-CoT** | **0.410** | **0.892** | **72.2%** | 21.8 | 1.2 | 4.50 |
| LLM (32B) | base_32B | None | 0.311 | 0.859 | 69.3% | 14.2 | 18.5 | 7.46 |
| LLM (32B) | ft_32B_summary | Summary-Only | 0.150 | 0.845 | 17.2% | 9.3 | 33.4 | 2.08 |
| LLM (32B) | ft_32B_cot | Single-CoT | 0.386 | 0.884 | 73.3% | 9.4 | 18.8 | 12.68 |
| LLM (32B) | **ft_32B_diversecot** | **Diverse-CoT** | **0.454** | **0.901** | **72.3%** | 9.4 | 33.6 | 13.62 |

Hit Rate = important-term hit rate (fuzzy matching, partial-ratio threshold 80, averaged over 10 clinically diverse test reports with 141 curated terms). CO2 reported as total grams CO2eq for a full 10-sample inference run on NVIDIA H100.

---

## Dataset

The dataset spans 110 histopathology subtopics across neoplastic, infectious, autoimmune, transplant-related, and pediatric domains. Three configurations are provided:

- **Single-CoT** — 5,500 records, each pairing a report with one structured three-step reasoning chain and a diagnostic summary.
- **Diverse-CoT** — 5,280 records derived from 330 base reports, each expanded into 16 distinct reasoning paths covering different clinical perspectives (clinical correlation, differential diagnosis, IHC analysis, molecular pathology, staging, prognosis, etc.).
- **Summary-Only** — 5,500 records, report-to-summary pairs without intermediate reasoning, used as the ablation baseline.

All reports are synthetically generated using GPT-4.1-mini under expert-validated prompt templates. A panel of three board-certified pathologists validated stratified samples from each configuration using a 5-point Likert scale across clinical accuracy, structural completeness, and reasoning coherence. All variants achieved mean scores ≥ 4.1/5 with Fleiss' κ ≥ 0.73.

See `Dataset/README.md` for full documentation, field schemas, coverage statistics, and generation methodology.

---

## Training

### Hardware Requirements

| Configuration | GPU | VRAM Required |
|---|---|---|
| Qwen-2.5-0.5B (QLoRA) | NVIDIA T4 | 15 GB |
| Qwen3-0.6B (QLoRA) | NVIDIA H100 | ~4 GB |
| Qwen3-32B (QLoRA) | NVIDIA H100 | ~40 GB |

### LoRA Hyperparameters

All models were trained with the following configuration:

| Parameter | Value |
|---|---|
| LoRA Rank (r) | 16 |
| LoRA Alpha (α) | 32 |
| LoRA Dropout | 0.05 |
| Learning Rate | 2 × 10⁻⁴ |
| Optimizer | AdamW |
| LR Schedule | Cosine with warmup |
| Warmup Ratio | 0.05 |
| Batch Size (per device) | 1 |
| Gradient Accumulation | 8 steps (effective batch = 8) |
| Quantization | 4-bit NF4 (QLoRA), BFloat16 compute |
| Max Sequence Length | 768 tokens |
| Target Modules | Attention layers (q_proj, k_proj, v_proj, o_proj) |

For Qwen3-32B CoT variants, a `WeightedCoTTrainer` applies 2× loss weight to summary tokens relative to reasoning tokens to prioritize summary fidelity while still learning from the chain-of-thought trace.

### Running Fine-Tuning

Each notebook is self-contained. Update the dataset file paths and adapter output directory at the top of the configuration cell before running.

```
LoRA_Summary_Code.ipynb           →  Qwen-2.5-0.5B + Summary-Only data
LoRA_Reasoning_Code.ipynb         →  Qwen-2.5-0.5B + Single-CoT or Diverse-CoT data
FT_Code_qwen600m_diversecot.ipynb →  Qwen3-0.6B + Diverse-CoT data
FT_Code_qwen600m_summary.ipynb    →  Qwen3-0.6B + Summary-Only data
FT_Code_qwen32b_cot.ipynb         →  Qwen3-32B + Single-CoT data
FT_Code_qwen32b_diversecot.ipynb  →  Qwen3-32B + Diverse-CoT data
FT_Code_qwen32b_summary.ipynb     →  Qwen3-32B + Summary-Only data
```

---

## Evaluation

`qwen3_eval_pipeline_h100.ipynb` runs the full evaluation for all eight Qwen3 model variants on a held-out test set of 10 clinically diverse histopathology reports. Metrics computed:

- ROUGE-L F1
- BERTScore Precision / Recall / F1 (using BiomedBERT)
- Perplexity
- Important-term hit rate (fuzzy matching, per-report curated vocabulary)
- Entity F1 (harmonic mean of hit rate recall and ROUGE-L precision)
- Inference throughput (tokens/sec)
- Peak inference VRAM (GB)
- Carbon footprint (g CO2eq, via CodeCarbon)
- Distinct-1 / Distinct-2 (lexical diversity)
- Output token length

To run the evaluation, update the adapter paths in Block 2 of the notebook to point to the corresponding directories under `FineTuned_Adapters/`.

---

## Data Generation

Two scripts are provided for reproducing the dataset from scratch. Both require an OpenAI API key (set via the `OPENAI_API_KEY` environment variable or entered at the prompt).

`CoTSingleData.py` generates Single-CoT records. It uses asynchronous API calls to GPT-4.1-mini with configurable concurrency (default: 10 concurrent requests), processing 50 reports per subtopic across all 110 subtopics.

`CoTDiverseData.py` generates Diverse-CoT records. It generates 3 base reports per subtopic and expands each into 16 reasoning paths using distinct reasoning-type prompts.

```bash
# Set your API key
export OPENAI_API_KEY="your-key-here"

# Generate Single-CoT data
python CoTSingleData.py

# Generate Diverse-CoT data
python CoTDiverseData.py
```

Key configuration parameters (editable in the `GenerationConfig` dataclass at the bottom of each script):

```python
config = GenerationConfig(
    report_model="gpt-4.1-mini-2025-04-14",
    cot_model="gpt-4.1-mini-2025-04-14",
    temperature=0.8,
    max_tokens=1000,
    reports_per_subtopic=50,       # 50 for Single-CoT; 3 for Diverse-CoT
    reasoning_types=16,            # Diverse-CoT only
    max_concurrent_requests=10,
    batch_size=50,
)
```

Generation is resumable: the scripts detect existing records in the output file and skip already-completed entries on restart.

---

## Inference

### Qwen-2.5-0.5B Models (MedReason / Med16Reason / MedSummary)

```python
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM
from peft import PeftModel

base_model_id = "Qwen/Qwen2.5-0.5B"
adapter_path   = "FineTuned_Adapters/qwen2.5_0.5B_Med16Reason_adapter"

tokenizer = AutoTokenizer.from_pretrained(adapter_path)
base_model = AutoModelForCausalLM.from_pretrained(
    base_model_id, torch_dtype=torch.bfloat16, device_map="auto"
)
model = PeftModel.from_pretrained(base_model, adapter_path)

report = "<your histopathology report text here>"

prompt = (
    "Instruction: Analyze the following histopathology report and provide a detailed "
    "chain-of-thought reasoning leading to the diagnosis, followed by a concise summary.\n\n"
    f"Input: {report}\n\n"
    "Response:\n"
)

inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
with torch.no_grad():
    output = model.generate(**inputs, max_new_tokens=512, do_sample=False)
print(tokenizer.decode(output[0], skip_special_tokens=True))
```

### Qwen3 Models (ft_0.6B_diversecot / ft_32B_diversecot / etc.)

Qwen3 models use the `/think` and `/no_think` mode toggles. CoT-trained variants should be prompted with `/think`; summary-only variants with `/no_think`.

```python
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM
from peft import PeftModel

base_model_id = "Qwen/Qwen3-0.6B"
adapter_path   = "FineTuned_Adapters/qwen3_0.6B_diversecot_adapter"

tokenizer = AutoTokenizer.from_pretrained(adapter_path)
base_model = AutoModelForCausalLM.from_pretrained(
    base_model_id, torch_dtype=torch.bfloat16, device_map="auto"
)
model = PeftModel.from_pretrained(base_model, adapter_path)

report = "<your histopathology report text here>"

prompt = (
    "/think\n"
    "Instruction: Analyze the following histopathology report and provide a detailed "
    "chain-of-thought reasoning leading to the diagnosis, followed by a concise summary.\n\n"
    f"Input: {report}\n\n"
    "Response:\n"
)

inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
with torch.no_grad():
    output = model.generate(**inputs, max_new_tokens=512, do_sample=False)
print(tokenizer.decode(output[0], skip_special_tokens=True))
```

---

## Dependencies

```
torch>=2.0
transformers>=4.40
peft>=0.10
bitsandbytes>=0.41
accelerate>=0.27
datasets>=2.18
rouge-score
bert-score
rapidfuzz
codecarbon
openpyxl
aiohttp        # data generation scripts only
```

Install with:

```bash
pip install torch transformers peft bitsandbytes accelerate datasets \
            rouge-score bert-score rapidfuzz codecarbon openpyxl aiohttp
```

---

## Limitations

**Synthetic data.** All training and evaluation data is synthetically generated. A pilot evaluation on 20 de-identified real reports showed a ~15% drop in BERT F1 compared to synthetic test performance, indicating that robustness to real-world linguistic variability requires further validation.

**Domain scope.** The framework is developed and evaluated exclusively on histopathology. Generalization to radiology, cardiology, or other clinical specialties requires dedicated evaluation.

**No deployment readiness.** Models in this repository are research artifacts. They should not be used for autonomous clinical decision-making. Any patient-facing application requires prospective multi-institutional validation, formal safety assessment, and regulatory review.

**Human oversight required.** CoT supervision improves interpretability but does not eliminate hallucination. Systematic failure modes identified in the paper include negation-centric term omissions, incomplete marker verbalization in final summaries, and rare vocabulary gaps at 0.6B scale.

---

## Ethical Considerations

No real patient data was used at any stage of this project. All histopathology reports are fully synthetic. Clinical expert involvement was limited to validation of prompt templates and generated outputs; no patient records were reviewed.

Models trained with this framework must be positioned as decision-support tools. Final diagnostic responsibility remains with qualified medical professionals. Users are expected to comply with applicable institutional and regulatory requirements before any clinical deployment.

---

## License

This repository is released under the **Apache 2.0 License**. See [LICENSE](LICENSE) for details.

The synthetic dataset was generated using GPT-4.1-mini (OpenAI). Users should consult OpenAI's terms of service regarding the use of model-generated content in downstream research and commercial applications.

---

## Contact

For questions or issues, please open a GitHub issue or contact the corresponding author at the email listed in the associated paper.
