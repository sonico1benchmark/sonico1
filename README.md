<h1 align="center">SONIC-O1 <br> Double-Anonymized Github</h1>
<h3 align="center">A Real-World Benchmark for Evaluating Multimodal Large Language Models on Audio-Video Understanding</h3>

<p align="center">
  <a href="https://arxiv.org/abs/XXXX.XXXXX"><img src="https://img.shields.io/badge/arXiv-Paper-b31b1b?logo=arxiv&logoColor=white" alt="Paper"></a>
  <a href="https://huggingface.co/datasets/sonico1org/sonico1"><img src="https://img.shields.io/badge/🤗-Dataset-FFD21E" alt="Dataset"></a>
  <a href="https://creativecommons.org/licenses/by/4.0/"><img src="https://img.shields.io/badge/License-CC%20BY%204.0-green.svg" alt="License"></a>
  <a href="https://www.python.org/downloads/"><img src="https://img.shields.io/badge/Python-3.8+-blue.svg" alt="Python"></a>
</p>

---

## Overview

This repository contains the **pipeline code** for SONIC-O1 — a benchmark for evaluating omnimodal video understanding with systematic fairness analysis. The pipeline enables end-to-end dataset creation and model evaluation across real-world conversational scenarios.

> **Looking for the dataset?** Download videos and annotations from [HuggingFace](https://huggingface.co/datasets/sonico1org/sonico1).

---

## Pipeline Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  01 Data        │───▶│  02 Caption     │───▶│  03 Demographics│───▶│  04 VQA         │───▶│  05 Evaluation  │
│  Curation       │    │  Generation     │    │  Annotation     │    │  Generation     │    │  & Inference    │
└─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘
   YouTube API            WhisperX             Gemini 2.5 Flash       Gemini 2.5 Flash            6+ MLLMs
```

| Stage | Purpose | Technology | Output |
|:-----:|:--------|:-----------|:-------|
| **01** | Video collection & filtering | YouTube Data API | Curated video corpus |
| **02** | Speech transcription | WhisperX | Word-level captions (SRT/JSON) |
| **03** | Character extraction | Gemini 2.5 Flash | Demographics metadata |
| **04** | Question generation | Gemini 2.5 Flash | 3 VQA task annotations |
| **05** | Model benchmarking | Multiple MLLMs | Evaluation scores |

---

## Repository Structure

```
sonico1/                            # Repository root
├── README.md                       # This file
└── sonic-o1/
    ├── requirements_venv.txt       # Dependencies
    └── sonic-o1/                   # Source code
        ├── 01_data_curation/       # YouTube video scraping & filtering
        ├── 02_caption_generation/  # WhisperX transcription pipeline
        ├── 03_demographics_annotation/  # Character demographics extraction
        ├── 04_vqa_generation/      # Multi-task VQA annotation
        ├── 05_evaluation_inference/    # Model evaluation framework
        ├── dataset/                # ⬇️ Download from HuggingFace
        └── vqa/                    # ⬇️ Download from HuggingFace
```

> Each stage has its own detailed README with installation and usage instructions.

---

## Quick Start

### Installation

```bash
# Clone repository
git clone https://github.com/sonico1benchmark/sonico1.git

# Navigate to source code directory
cd sonico1/sonic-o1/sonic-o1

# Create environment
python -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -r ../requirements_venv.txt
```

### Dataset Download

```bash
# From the sonic-o1/sonic-o1/ directory (source code root)
pip install huggingface_hub
huggingface-cli download sonico1org/sonico1 --repo-type dataset --local-dir ./
```

This downloads:
- `dataset/` — 231 videos (~60 hours), audio tracks, captions
- `vqa/` — 4,958 annotations across 3 tasks

---

## Evaluation Tasks

| Task | Description | Instances | Metrics |
|:----:|:------------|:---------:|:--------|
| **T1** | Video Summarization | 231 | ROUGE-L, LLM-as-Judge |
| **T2** | Multiple Choice QA | 1,335 | Accuracy |
| **T3** | Temporal Localization | 3,392 | mIoU, Recall@K, MAE |

### Run Evaluation

```bash
cd 05_evaluation_inference

python run_evaluation.py \
    --model videollama2 \
    --tasks t1,t2,t3 \
    --topics all
```

### Supported Models

| Open-Source | Commercial APIs |
|:------------|:----------------|
| VideoLLaMA2 | Gemini 3.0 Pro |
| VITA 1.5 | GPT-4o |
| Uni-MoE-2 | |
| MiniCPM-o-2.6 | |
| Qwen3-Omni | |

---

## Creating New Datasets

```bash
# From sonic-o1/sonic-o1/ (source code directory)

# Stage 1: Collect videos
cd 01_data_curation
python parse_topic.py --topics "Your_New_Topic"

# Stage 2: Generate captions
cd ../02_caption_generation
python whisper_captionGen.py --topics "Your_New_Topic"

# Stage 3: Extract demographics
cd ../03_demographics_annotation
python run_annotation.py --topics "Your_New_Topic"

# Stage 4: Generate VQA tasks
cd ../04_vqa_generation
python main.py --topics 1 --tasks summarization,mcq,temporal_localization
```

---

## Topics Covered

| Domain | Topics |
|:-------|:-------|
| **Healthcare** | Patient-Doctor Consultations, Mental Health Counseling |
| **Professional** | Job Interviews, Workplace Meetings |
| **Legal/Civic** | Courtroom Proceedings, Community Town Halls |
| **Education** | Parent-Teacher Conferences |
| **Service** | Customer Service, Restaurant Encounters, Housing Tours |
| **Emergency** | Emergency Response Scenarios, Public Transportation |
| **Sports** | Olympics Coverage |

---

## Configuration

| Stage | Config File | Key Settings |
|:------|:------------|:-------------|
| 01 | `config.yaml` | Search queries, filtering thresholds |
| 02 | `config_whisper.yaml` | Model size, language, device |
| 03 | `config.yaml` | LLM backend, rate limits |
| 04 | `config/vqa_config.yaml` | Task parameters, output format |
| 05 | `configs/models_config.yaml` | Model paths, inference settings |

---

## Environment Variables

Create `.env` files in relevant stage directories:

```bash
YOUTUBE_API_KEY=...      # Stage 01
GEMINI_API_KEY=...       # Stages 03, 04, 05
OPENAI_API_KEY=...       # Stages 04, 05
HF_TOKEN=...             # Stage 05
```

---

## Documentation

| Stage | README |
|:------|:-------|
| Data Curation | [01_data_curation/README.md](sonic-o1/sonic-o1/01_data_curation/README.md) |
| Caption Generation | [02_caption_generation/README.md](sonic-o1/sonic-o1/02_caption_generation/README.md) |
| Demographics | [03_demographics_annotation/README.md](sonic-o1/sonic-o1/03_demographics_annotation/README.md) |
| VQA Generation | [04_vqa_generation/README.md](sonic-o1/sonic-o1/04_vqa_generation/README.md) |
| Evaluation | [05_evaluation_inference/README.md](sonic-o1/sonic-o1/05_evaluation_inference/README.md) |

---

## Citation

```bibtex
@article{sonic-o1-2026,
  title={SONIC-O1:A Real-World Benchmark for Evaluating Multimodal Large Language Models on Audio-Video Understanding},
  year={2026}
}
```

---

## License

This project is licensed under **CC BY 4.0**. See [LICENSE](LICENSE) for details.

---

<p align="center">
  <sub>Built for advancing fair and comprehensive video understanding research</sub>
</p>
