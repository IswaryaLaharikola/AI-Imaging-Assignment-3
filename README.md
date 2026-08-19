# Hybrid Biomedical Image-Analysis Pipeline

Coursework: *Assignment 3 — AI Imaging Coding Case Study and Report.*

A local, auditable pipeline that moves a biomedical image through several
stages and produces a structured, checkable record rather than a free-text
guess:

```
raw image → segmentation → quantitative region features
          → structured JSON record → short narrative
```

It combines three method families from the module: a local multimodal LLM
(VLM) via Ollama, classical image processing (scikit-image), and a small U-Net
(PyTorch).

> **Educational use only.** None of these models are cleared for clinical use.
> LLM outputs can hallucinate; the structured JSON is kept as the source of
> truth precisely so the narrative can always be checked against the numbers.

## Repository layout

```
ai_imaging/
├── data/                    # put the assigned dataset here
├── src/
│   ├── config.py            # paths + constants (edit DATA_DIR / model names)
│   ├── data_prep.py         # Task 1: load, grayscale, resize, EDA
│   ├── llm_interface.py     # Tasks 1,2,4: Ollama VLM + text prompts (JSON)
│   ├── classical_features.py# Task 2: Otsu, morphology, regionprops table
│   ├── unet.py              # Task 3: small U-Net, training, Dice/IoU, plots
│   ├── masks.py             # image/mask pairing + train/val split
│   └── pipeline.py          # Task 4: hybrid pipeline → JSON records → CSV
├── outputs/
│   ├── figures/             # all saved figures for the report
│   └── records/             # per-image JSON + aggregated_records.csv
├── models/                  # saved U-Net weights
├── run_pipeline.py          # end-to-end driver for all four tasks
├── requirements.txt
└── README.md
```

## Setup

### 1. Python environment
```bash
python -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Ollama (for the LLM steps — Tasks 1, 2, 4)
Install Ollama (https://ollama.com), then:
```bash
ollama serve                      # start the local server
ollama pull llama3.2-vision       # multimodal model (sees images)
ollama pull llama3.2              # text model (numbers-first)
```
If Ollama is not running, the pipeline still executes: the LLM steps are
skipped and deterministic rule-based stand-ins fill the JSON records, so you
can develop the rest offline. The LLM outputs must be present in your final
submission, so run with Ollama up for the real results.

### 3. Dataset
Download from `https://github.com/Nickolay-K/Assingnment-3-dataset` (note the
spelling) and unpack so that `data/nuclei_dataset/` exists. The loaders also
accept the splits placed directly under `data/`.

The assigned modality is **fluorescence-microscopy cell nuclei** (synthetic,
DAPI-like blue nuclei on a dark field). Layout:

```
nuclei_dataset/
    train/  images/*.png  masks/*.png  labels/*.png   (80 pairs)
    val/    images/*.png  masks/*.png  labels/*.png   (20 pairs)
    test/   images/*.png  masks/*.png  labels/*.png   (12 pairs — treat as unseen)
    test_corrupted/images/*.png                        (blur / low-contrast — extension)
    metadata.csv                                       (ground-truth answer key)
```

- **images** — 256×256 RGB (already the target size; grayscaled on load).
- **masks** — binary PNG (0 / 255): foreground vs background — the U-Net's labels.
- **labels** — 16-bit instance maps (each nucleus a distinct integer) for optional
  instance-level work.
- **metadata.csv** — per-image `n_objects`, `density`, `mean_intensity`,
  `area_fraction`. Use this to score your U-Net counts and the LLM density
  labels against truth — strong material for the report's evaluation section.

## Running

Everything at once:
```bash
python run_pipeline.py
```

Or a single stage while developing:
```bash
python -m src.data_prep           # Task 1 EDA
python -m src.classical_features  # Task 2 (synthetic demo)
python -m src.pipeline            # Task 4 (synthetic demo, offline)
```

## Task → code map

| Task | What it does | Where |
|---|---|---|
| 1 | Load/grayscale/resize, EDA, VLM description + repeatability | `data_prep.py`, `llm_interface.py` |
| 2 | Otsu + morphology + regionprops → numbers-first LLM JSON | `classical_features.py`, `llm_interface.py` |
| 3 | Train small U-Net, evaluate Dice/IoU, prediction panels | `unet.py`, `masks.py` |
| 4 | Hybrid pipeline → per-image JSON → aggregated CSV | `pipeline.py` |

## Notes for the report

- **Prompts** (Task 1 & 2) live in `llm_interface.py` as `OPTIMISED_VLM_PROMPT`,
  `NAIVE_VLM_PROMPT`, and `OPTIMISED_TEXT_PROMPT`. The brief requires the
  optimised prompts to appear in the report.
- **Ground-truth masks are provided**, so the U-Net trains on real labels (no
  Otsu bootstrapping needed). This means the U-Net-vs-Otsu comparison in
  question 2 is a fair fight between two independent methods.
- **The corrupted test set** (`test_corrupted/`) is ready-made for the
  robustness extension: run the trained model on the blur / low-contrast
  variants and trace where the corruption first becomes detectable.
- **Hallucination / auditability** (question 4): the LLM only ever sees numbers
  in Task 2/4, the JSON schema is fixed, and "uncertain" is allowed — so the
  free-text narrative can always be reconciled with the structured record.
