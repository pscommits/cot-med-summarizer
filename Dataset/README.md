---
language:
- en
license: apache-2.0
task_categories:
- text-generation
- summarization
task_ids:
- language-modeling
- abstractive-summarization
pretty_name: CoT-Med-Summarizer Histopathology Dataset
size_categories:
- 10K<n<100K
tags:
- medical
- histopathology
- chain-of-thought
- clinical-nlp
- summarization
- reasoning
- small-language-models
- synthetic
- pathology
---

# CoT-Med-Summarizer: Chain-of-Thought Histopathology Summarization Dataset

## Dataset Summary

This dataset supports research on Chain-of-Thought (CoT) supervised fine-tuning of Small Language Models (SLMs) for explainable histopathology report summarization. It provides a synthetic, privacy-preserving corpus of histopathology reports paired with structured three-step reasoning traces and diagnostic summaries, designed for controlled investigation of how reasoning supervision — and its diversity — affects summarization fidelity, generalization, and clinical interpretability.

The dataset spans **110 histopathology subtopics** covering neoplastic, infectious, autoimmune, transplant-related, and pediatric pathology domains. It is constructed in three distinct formats to support ablation across supervision type. To the best of our knowledge, this is the first publicly available dataset combining synthetic histopathology reports with structured, expert-validated Chain-of-Thought reasoning annotations for supervised training of explainable summarization models.

**GitHub Repository:** [https://github.com/pscommits/cot-med-summarizer](https://github.com/pscommits/cot-med-summarizer)

---

## Dataset Structure

The dataset is partitioned into three configuration variants — **Single-CoT**, **Diverse-CoT**, and **Summary-Only** — each split into train and validation sets, yielding 16,280 records across six JSONL files.

### File Overview

| File | Configuration | Split | Records |
|---|---|---|---|
| `train_cot.jsonl` | Single-CoT | Train | 4,950 |
| `val_cot.jsonl` | Single-CoT | Validation | 550 |
| `train_diverse_cot.jsonl` | Diverse-CoT | Train | 4,488 |
| `val_diverse_cot.jsonl` | Diverse-CoT | Validation | 792 |
| `train_summary.jsonl` | Summary-Only | Train | 4,950 |
| `val_summary.jsonl` | Summary-Only | Validation | 550 |
| **Total** | | | **16,280** |

---

## Configuration Descriptions

### Single-CoT (`train_cot.jsonl`, `val_cot.jsonl`)

Each record pairs one histopathology report with a single structured three-step reasoning chain and a final diagnostic summary. The dataset contains 50 reports per subtopic across 110 subtopics (5,500 total), split 90/10 into train and validation (4,950 / 550).

This configuration is designed for baseline supervised fine-tuning with structured reasoning guidance. It exposes the model to one reasoning path per report — a consistent clinical reasoning template, but with limited perspective diversity.

**Schema:**

```json
{
  "topic": "Hashimoto Thyroiditis",
  "id": "hashimoto_thyroiditis_report26",
  "instruction": "Analyze the following histopathology report and provide a detailed chain-of-thought reasoning leading to the diagnosis, followed by a concise summary.",
  "input": "<histopathology report text>",
  "reasoning_output": "1. Histopathological Correlation – ... 2. Ancillary Interpretation – ... 3. Diagnostic Integration – ...",
  "summary_output": "<concise diagnostic summary>"
}
```

| Field | Type | Description |
|---|---|---|
| `topic` | string | Histopathology subtopic label (110 categories) |
| `id` | string | Unique record identifier in format `{topic_slug}_report{N}` |
| `instruction` | string | Fixed task instruction for the model |
| `input` | string | Synthetic histopathology report (100–150 words) |
| `reasoning_output` | string | Three-step structured CoT reasoning (150–200 words) |
| `summary_output` | string | Concise diagnostic summary (1–2 sentences) |

---

### Diverse-CoT (`train_diverse_cot.jsonl`, `val_diverse_cot.jsonl`)

Each of 330 base reports (3 per subtopic across 110 subtopics) is paired with **16 distinct reasoning paths**, each written from a different clinical perspective, yielding 5,280 total records (330 × 16). The train/validation split is approximately 85/15 (4,488 / 792).

This is the primary configuration of the dataset. Motivated by findings in knowledge distillation and reasoning diversity literature (Ho et al., 2023; Magister et al., 2023), the 16 reasoning types expose the model to varied but clinically valid diagnostic thought processes, improving generalization and reducing overfitting to a single reasoning style.

**Schema:**

```json
{
  "topic": "Gastrointestinal Biopsies",
  "id": "gastrointestinal_biopsies_report1_reasoning16",
  "instruction": "Analyze the following histopathology report and provide a detailed chain-of-thought reasoning leading to the diagnosis, followed by a concise summary.",
  "input": "<histopathology report text>",
  "reasoning_output": "<reasoning from one of the 16 clinical perspectives>",
  "summary_output": "<concise diagnostic summary>"
}
```

The `id` field encodes both the base report and the reasoning index: `{topic_slug}_report{N}_reasoning{M}`, where N is the base report index (1–3) and M is the reasoning type index (1–16).

**The 16 Reasoning Perspectives:**

| Index | Reasoning Type | Description |
|---|---|---|
| 1 | `clinical_correlation` | Clinical Correlation Analysis |
| 2 | `differential_diagnosis` | Differential Diagnosis Reasoning |
| 3 | `morphological_pattern` | Morphological Pattern Recognition |
| 4 | `immunohistochemical` | Immunohistochemical Analysis |
| 5 | `molecular_pathology` | Molecular Pathology Integration |
| 6 | `staging_grading` | Staging and Grading Assessment |
| 7 | `prognostic_analysis` | Prognostic Factor Analysis |
| 8 | `treatment_planning` | Treatment Planning Orientation |
| 9 | `comparative_pathology` | Comparative Pathology Reasoning |
| 10 | `evidence_based` | Evidence-Based Medicine Approach |
| 11 | `systematic_workup` | Systematic Diagnostic Workup |
| 12 | `risk_stratification` | Risk Stratification Framework |
| 13 | `biomarker_driven` | Biomarker-Driven Interpretation |
| 14 | `multidisciplinary` | Multidisciplinary Team Perspective |
| 15 | `quality_assurance` | Quality Assurance and Audit Review |
| 16 | `educational_case` | Educational Case Presentation |

---

### Summary-Only (`train_summary.jsonl`, `val_summary.jsonl`)

Each record pairs a histopathology report directly with a diagnostic summary, without any intermediate reasoning chain. The structure mirrors Single-CoT in report coverage (5,500 total; 4,950 / 550 split), but the `output` field contains only the summary.

This configuration serves as the baseline ablation condition, enabling measurement of the isolated contribution of task-specific fine-tuning independent of reasoning supervision.

**Schema:**

```json
{
  "topic": "Hashimoto Thyroiditis",
  "id": "hashimoto_thyroiditis_report26",
  "instruction": "Analyze the following histopathology report and provide a detailed chain-of-thought reasoning leading to the diagnosis, followed by a concise summary.",
  "input": "<histopathology report text>",
  "output": "<concise diagnostic summary>"
}
```

Note that in this configuration the target field is named `output` (not `summary_output`), and no `reasoning_output` field is present.

---

## Data Statistics

### Input Report Statistics (word count, across all configurations)

| Split | Mean | Median | P90 | P95 | P99 | Min | Max |
|---|---|---|---|---|---|---|---|
| train_cot | 130.5 | 131 | 143 | 146 | 151 | 95 | 161 |
| val_cot | 130.6 | 131 | 143 | 146 | 150 | 103 | 153 |
| train_diverse_cot | 135.1 | 136 | 145 | 147 | 150 | 110 | 159 |
| val_diverse_cot | 135.0 | 136 | 144 | 147 | 150 | 110 | 159 |
| train_summary | 130.5 | 131 | 143 | 146 | 151 | 95 | 161 |
| val_summary | 130.6 | 131 | 143 | 146 | 150 | 103 | 153 |

Reports are consistently 100–150 words in length, enforced by the generation prompt. Average tokenized input length is approximately 169–175 tokens.

### Reasoning Output Statistics (Single-CoT and Diverse-CoT)

| Split | Mean | Median | P90 | P95 | P99 | Min | Max |
|---|---|---|---|---|---|---|---|
| train_cot | 146.9 | 147 | 162 | 166 | 175 | 106 | 195 |
| val_cot | 148.0 | 148 | 163 | 167 | 177 | 111 | 184 |
| train_diverse_cot | 179.2 | 178 | 200 | 206 | 221 | 109 | 241 |
| val_diverse_cot | 178.8 | 178 | 200 | 206 | 216 | 138 | 230 |

Diverse-CoT reasoning traces are systematically longer (~179 words on average) than Single-CoT traces (~147 words), reflecting the broader clinical perspectives integrated across reasoning types. Average tokenized reasoning length is approximately 190 tokens for Single-CoT and 232 tokens for Diverse-CoT.

### Summary Output Statistics

| Split | Mean | Median | P90 | P95 | P99 | Min | Max |
|---|---|---|---|---|---|---|---|
| train_cot | 25.4 | 25 | 31 | 33 | 35 | 11 | 43 |
| val_cot | 25.4 | 25 | 30 | 32 | 35 | 12 | 41 |
| train_diverse_cot | 28.6 | 28 | 34 | 35 | 38 | 17 | 43 |
| val_diverse_cot | 28.5 | 28 | 33 | 35 | 37 | 18 | 42 |
| train_summary | 25.4 | 25 | 31 | 33 | 35 | 11 | 43 |
| val_summary | 25.4 | 25 | 30 | 32 | 35 | 12 | 41 |

Summaries are concise 1–2 sentence diagnostic conclusions. Average tokenized summary length is approximately 32–37 tokens.

---

## Covered Histopathology Subtopics

The dataset covers 110 distinct histopathology subtopics spanning five broad clinical domains:

**Neoplastic — Carcinomas and Adenocarcinomas**
Adrenal Cortical Carcinomas, Anaplastic Thyroid Carcinomas, Basal Cell Carcinomas of Skin, Bladder Transitional Cell Carcinomas, Brain Tumor Craniotomies, Breast Carcinoma Resections, Cervical Squamous Cell Carcinomas, Cholangiocarcinomas, Colorectal Adenocarcinomas, Esophageal Adenocarcinomas, Follicular Thyroid Carcinomas, Gallbladder Carcinomas, Gastric Signet Ring Cell Carcinomas, Gastrointestinal Biopsies, Head and Neck Squamous Cell Carcinomas, Hepatocellular Carcinomas, Lung Adenocarcinoma Specimens, Medullary Thyroid Carcinomas, Merkel Cell Carcinomas, Nasopharyngeal Carcinomas, Ovarian Serous Carcinomas, Pancreatic Neuroendocrine Tumors, Penile Carcinomas, Prostate Core Needle Biopsies, Renal Cell Carcinomas, Small Intestinal Adenocarcinomas, Thymic Carcinomas, Urothelial Carcinomas in Situ, Vulvar Squamous Cell Carcinomas

**Neoplastic — Sarcomas, Lymphomas, and Other Tumors**
Acute Myeloid Leukemia Infiltrates, Angiosarcomas, Appendiceal Mucinous Neoplasms, Bone Marrow Biopsies for Plasma Cell Myeloma, Bone Marrow Core Biopsies, Bone Sarcoma Resections, Carcinoid Tumors of Appendix, Chondrosarcomas, Choriocarcinomas, Chronic Lymphocytic Leukemia Nodes, Clear Cell Sarcomas, Diffuse Large B-cell Lymphoma Biopsies, Endocrine Pancreatic Tumors, Endoscopic Ultrasound-Guided FNA, Ewing Sarcomas, Follicular Lymphomas, Gastrointestinal Stromal Tumors (GIST), Hemangiopericytomas, Hodgkin Lymphoma Specimens, Langerhans Cell Histiocytosis, Lymph Node Resections, Mediastinal Mass Biopsies, Medulloblastomas, Meningioma Resections, Mesothelioma Specimens, Metastatic Melanoma Lymph Nodes, Myelodysplastic Syndromes, Neuroblastoma Resections, Osteosarcomas, Peripheral Nerve Sheath Tumors, Pheochromocytomas, Pituitary Adenomas, Recurrent Glioblastoma Specimens, Retinoblastomas, Rhabdomyosarcomas, Salivary Gland Tumors, Sentinel Lymph Node Biopsies, Skin Excisions for Melanoma, Soft Tissue Liposarcomas, Stereotactic Brain Biopsies, Synovial Sarcomas, Testicular Seminomas, Thyroid Fine Needle Aspirates

**Infectious and Granulomatous**
Cytomegalovirus Colitis, Fungal Infections in Lung Biopsies, Herpes Simplex Esophagitis, HIV-associated Lymphadenopathy, Parasitic Infections in Tissue, Sarcoidosis Lymph Node Biopsies, Syphilitic Placentitis, Tuberculosis Granulomas

**Autoimmune and Inflammatory**
Amyloidosis Specimens, Autoimmune Gastritis, Autoimmune Hepatitis Specimens, Celiac Disease Biopsies, Graft-versus-Host Disease in GI Biopsies, Hashimoto Thyroiditis, Inflammatory Bowel Disease Biopsies, Lupus Nephritis Biopsies, Sjögren Syndrome Salivary Glands

**Transplant, Pediatric, and Procedural**
Cardiac Transplant Biopsies, Chronic Villitis of Unknown Etiology (VUE), Congenital Pulmonary Airway Malformation (CPAM), Ectopic Pregnancies, Endometrial Biopsies, Fine Needle Aspirations of Salivary Glands, Frozen Section Intraoperative Consultations, Hydatidiform Moles, Liver Transplant Evaluations, Lung Transplant Rejection Specimens, Parathyroid Adenomas, Pediatric Wilms Tumors, Placental Abruption with Infarcts, Placental Pathology Specimens, Punch Biopsies of Skin Rashes, Renal Allograft Biopsies, Teratomas (Pediatric), Transbronchial Lung Biopsies, Tru-Cut Biopsies of Retroperitoneal Masses, Uterine Leiomyoma Hysterectomies

---

## Data Generation Methodology

### Synthetic Generation Pipeline

All data was generated synthetically using GPT-4.1-mini via a two-stage pipeline: (1) histopathology report generation, and (2) structured reasoning-summary generation. No real patient data was used at any stage. The pipeline is implemented in `CoTSingleData.py` and `CoTDiverseData.py` in the repository root.

**Stage 1 — Report Generation**

Each report is generated using a parameterized prompt template that injects seven controlled diversity parameters sampled from predefined distributions:

```
age:           sampled from {[20–35], [35–50], [50–65], [65–80], [80–95]}
gender:        {Male, Female}
severity:      {early-stage, intermediate, advanced, metastatic}
presentation:  {symptomatic, incidental, screening-detected, follow-up}
specimen_size: {small, medium, large}
grade:         {well-differentiated, moderately-differentiated, poorly-differentiated}
margin_status: {clear, close, positive}
```

The prompt enforces a 100–150 word single-paragraph clinical report in formal pathology language containing nine mandatory diagnostic components: patient demographics, procedure indication, specimen type and anatomical site, gross findings, microscopic features, margin status, lymph node evaluation, IHC panel interpretation, and final diagnosis.

**Stage 2 — Reasoning and Summary Generation**

For Single-CoT records, each report is passed through a second prompt that elicits a structured three-step reasoning chain followed by a diagnostic summary. The three steps mirror the actual diagnostic workflow in surgical pathology:

1. **Histopathological Correlation** — linking clinical context and gross findings with microscopic morphology
2. **Ancillary Interpretation** — interpreting IHC and molecular markers with diagnostic implications
3. **Diagnostic Integration** — synthesizing all evidence into a final diagnosis

For Diverse-CoT records, each base report is passed through all 16 specialized reasoning prompts, each emphasizing a distinct clinical perspective (see the 16 Reasoning Perspectives table above). Temperature is set to 0.8 to introduce natural linguistic variation across reasoning paths.

**Multi-Stage Validation**

Each generated sample was validated against five quality criteria before inclusion:

- Q1: Length constraints satisfied (100–150 words for reports, 150–200 for reasoning)
- Q2: All nine diagnostic components present in the report
- Q3: No duplicate or nonsensical content
- Q4: CoT steps are logically connected and sequentially coherent
- Q5: Summary accurately reflects both the report and the reasoning chain

### Clinical Expert Validation

Finalized prompt templates and stratified samples from all three dataset variants were evaluated by a panel of three board-certified pathologists. Each sample was rated on a 5-point Likert scale (1 = Poor, 5 = Excellent) across three dimensions: clinical accuracy (Q_clinical), structural completeness (Q_structure), and logical coherence of reasoning (Q_reasoning). Inter-rater reliability was assessed using Fleiss' Kappa.

| Dataset Variant | Q_clinical | Q_structure | Q_reasoning | Fleiss' κ (mean) |
|---|---|---|---|---|
| Single-CoT (D_single) | 4.6 ± 0.3 | 4.5 ± 0.4 | 4.4 ± 0.3 | 0.78 |
| Diverse-CoT (D_diverse) | 4.7 ± 0.2 | 4.6 ± 0.3 | 4.7 ± 0.2 | 0.81 |
| Summary-Only (D_summary) | 4.3 ± 0.4 | 4.2 ± 0.4 | 4.1 ± 0.5 | 0.75 |

All variants achieved high clinical fidelity (mean ≥ 4.1/5) with substantial to near-perfect inter-rater agreement (κ ≥ 0.73), confirming that the prompt refinement pipeline produces clinically robust and reproducible synthetic data.

---

## Prompt Templates for Fine-Tuning

The following templates are used in the paper's fine-tuning experiments. Tokenize the full concatenated string and mask padding tokens with `-100` in the labels to exclude them from the loss.

**CoT-trained models (Single-CoT and Diverse-CoT):**

```python
def format_cot_example(example, tokenizer, max_length=768):
    text = (
        f"Instruction: {example['instruction']}\n\n"
        f"Input: {example['input']}\n\n"
        f"Response:\n"
        f"{example['reasoning_output']}\n\n"
        f"Summary:\n{example['summary_output']}"
        f"{tokenizer.eos_token}"
    )
    return tokenizer(text, truncation=True, max_length=max_length, padding="max_length")
```

**Summary-Only models:**

```python
def format_summary_example(example, tokenizer, max_length=768):
    text = (
        f"Instruction: {example['instruction']}\n\n"
        f"Input: {example['input']}\n\n"
        f"Response:\n{example['output']}"
        f"{tokenizer.eos_token}"
    )
    return tokenizer(text, truncation=True, max_length=max_length, padding="max_length")
```

---

## Prompt Templates for Inference

**CoT-trained Qwen-2.5-0.5B models:**

```
Instruction: Analyze the following histopathology report and provide a detailed chain-of-thought
reasoning leading to the diagnosis, followed by a concise summary.

Input: {histopathology_report}

Response:
```

**Summary-only or base Qwen-2.5-0.5B models:**

```
Instruction: Analyze the following histopathology report and provide a concise summary.

Input: {histopathology_report}

Response:
```

**CoT-trained Qwen3 models (activate `/think` mode):**

```
/think
Instruction: Analyze the following histopathology report and provide a detailed chain-of-thought
reasoning leading to the diagnosis, followed by a concise summary.

Input: {histopathology_report}

Response:
```

**Summary-only Qwen3 models:**

```
/no_think
Instruction: Analyze the following histopathology report and provide a concise summary.

Input: {histopathology_report}

Response:
```

---

## Experimental Results Summary

The following results are from the accompanying paper. All fine-tuned models use LoRA (rank 16) on the base architectures listed.

### Primary SLM Ablation (Qwen-2.5-0.5B, NVIDIA T4)

Evaluated on 550 held-out benchmark samples with expert-written reference summaries.

| Model | Training Config | ROUGE-L F1 | BERT F1 | Perplexity | GPT-4.1 Score | GPT-4.1 Blind Picks |
|---|---|---|---|---|---|---|
| Qwen2.5-0.5B (base) | None | 7.11 | 56.77 | 36.97 | 4.66 | 0 |
| MedSummary | Summary-Only | 33.56 | 73.30 | 15.17 | 7.79 | 21 |
| MedReason | Single-CoT | 39.59 | 75.45 | 4.14 | 8.14 | 25 |
| **Med16Reason** | **Diverse-CoT** | **43.00** | **76.06** | **6.40** | **8.99** | **43** |
| AlpaCare-7B | external | 11.85 | 57.35 | 29.51 | 7.58 | 11 |
| BioMistral-7B | external | 11.67 | 58.19 | 50.87 | 5.94 | 0 |
| BioMedLM-2.7B | external | 8.20 | 49.00 | 18.47 | 5.57 | 0 |

### Cross-Scale Comparison (Qwen3 family, NVIDIA H100)

Evaluated on 10 clinically diverse held-out reports with 141 curated clinical terms. Hit Rate = important-term hit rate (fuzzy matching). CO2 = total grams CO2eq for 10-sample inference run.

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

The Diverse-CoT fine-tuned 0.6B SLM achieves a clinical-term hit rate of 72.2% and a BERTScore F1 of 0.892 — within 0.1 percentage points and 0.009 respectively of the 53× larger 32B model — while operating at 2.3× faster inference, requiring 28× less VRAM, and emitting 3.0× less carbon per inference run.

---

## Data Generation Scripts

Two generation scripts are included in the repository root:

**`CoTSingleData.py`** — Generates Single-CoT records. Uses asynchronous API calls to GPT-4.1-mini with up to 10 concurrent requests. Generates 50 reports per subtopic across 110 subtopics.

**`CoTDiverseData.py`** — Generates Diverse-CoT records. Generates 3 base reports per subtopic and expands each into 16 reasoning paths using distinct reasoning-type prompts.

Both scripts require an OpenAI API key and are configurable via the `GenerationConfig` dataclass. Generation is resumable: the scripts detect existing records in the output file and skip already-completed entries on restart.

---

## Limitations

**Synthetic data.** All records are synthetically generated. While expert-validated for clinical fidelity, the data does not capture the full linguistic variability of real clinical notes — including institution-specific shorthand, addenda, non-standard formatting, and rare disease presentations. A pilot evaluation on 20 de-identified real reports revealed a ~15% drop in BERT F1 compared to performance on synthetic test data.

**Domain scope.** The dataset covers histopathology exclusively. Generalization to other medical specialties requires dedicated evaluation and likely additional domain-specific prompt engineering.

**GPT-4.1-mini as generator.** CoT annotations are generated by GPT-4.1-mini under expert-constrained prompts, not authored directly by practicing pathologists. The expert panel validated outputs retrospectively. Quality is high (mean ≥ 4.4/5 across all criteria) but this distinction is relevant in safety-critical applications.

**No deployment readiness.** Models trained on this dataset should not be used for autonomous clinical decision-making. The dataset and trained models are intended for research and educational purposes. Responsible deployment requires prospective validation on multi-institutional real-world data, formal safety assessment, integration with human oversight workflows, and regulatory review.

**Demographic and linguistic coverage.** Diversity parameters cover age, gender, severity, and presentation type but do not model geographic, ethnic, or institutional variation in reporting style. Rare conditions and diagnostically borderline presentations may be underrepresented.

---

## Ethical Considerations

No real patient data was used in the creation of this dataset. All histopathology reports are fully synthetic and contain no identifiable patient information. The synthetic generation approach was chosen specifically to eliminate privacy risks while enabling the creation of a large, structured, and annotatable corpus.

Clinical expert participation in the validation phase was voluntary. Experts provided feedback on model-generated synthetic outputs; no real patient records were involved. Formal ethics committee approval was not required under institutional guidelines for studies using only synthetic data and expert consultation.

Users of this dataset are expected to treat outputs from models trained on it as decision-support tools, not autonomous diagnostic systems. Any deployment in patient-facing settings requires human oversight by qualified medical professionals.

---

## License

This dataset is released under the **Apache 2.0 License**. See [LICENSE](LICENSE) for details.

The synthetic data was generated using GPT-4.1-mini (OpenAI). Users should consult OpenAI's terms of service regarding the use of model-generated content in downstream research and commercial applications.

---

## Contact

For questions regarding the dataset, please open an issue on the [GitHub repository](https://github.com/pscommits/cot-med-summarizer) or contact the corresponding author via the email listed in the associated paper.
