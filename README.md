# Biomedical Image Analysis – Hybrid U-Net and LLM Pipeline

## Overview

This project implements an end-to-end biomedical image-analysis pipeline for cell-nuclei images. It combines classical image processing, deep-learning segmentation and Large Language Model (LLM) interpretation.

The pipeline covers four main tasks:
- Data preparation and exploratory data analysis (EDA)
- Multimodal LLM image description
- Classical segmentation and quantitative feature extraction
- U-Net segmentation and a hybrid LLM interpretation pipeline

The aim is to combine quantitative image analysis with structured LLM interpretation while maintaining transparency and auditability.

## Task 1 – Data Preparation and Multimodal LLM Description

Images are converted to grayscale and resized to 256 × 256 pixels. Exploratory analysis includes representative images and an intensity histogram.

A representative image is provided to a local multimodal model using Qwen2.5-VL-3B-Instruct.

The optimised prompt instructs the model to describe observable features rather than diagnose, avoid unsupported medical claims, use "uncertain" when information cannot be determined, and return structured JSON.

Required fields:
- `modality`
- `tissue_type`
- `notable_features`
- `image_quality`

A naive prompt is compared with the optimised prompt, and repeated runs are used to examine output variability.

## Task 2 – Classical Features and LLM Interpretation

Classical image processing is performed using scikit-image:

1. Otsu thresholding
2. Morphological cleaning
3. Connected-component labelling
4. Region-level feature extraction using `regionprops_table`

Features include area, eccentricity, solidity, mean intensity and object count.

The numerical features are converted into a short summary and provided to the LLM without the original image. The LLM produces `n_objects`, `density_class`, `shape_regularity`, and `quality_flag`.

## Task 3 – U-Net Segmentation

A compact PyTorch U-Net is trained using labelled training images and evaluated on the held-out validation set.

| Metric | Result |
|---|---:|
| Dice | 0.997 |
| IoU | 0.993 |

The notebook also generates training/loss curves, Dice curves, input images, ground-truth masks and predicted masks.

## Task 4 – Hybrid Pipeline

The complete pipeline is applied to the unseen test images:

```text
Test Image → U-Net Mask → Region Features → LLM Interpretation → Structured JSON → Narrative
```

Each test image produces structured information including `image_id`, `n_objects`, `mean_area`, `density_class`, and `quality_flag`.

The records are aggregated into a pandas DataFrame and saved as `aggregated_records.csv`.

The quantitative segmentation and feature measurements are treated as the source of truth, while the LLM provides interpretation and narrative generation.

## Main Results

The U-Net achieved Dice = **0.997** and IoU = **0.993**. It also achieved a higher mean Dice than the classical Otsu approach in the notebook comparison.

The multimodal LLM experiments demonstrated that prompt engineering improved the usefulness and structure of generated descriptions. Repeated runs also showed that LLM outputs can vary.

## Project Structure

```text
Biomedical-Image-Analysis/
├── AI_imaging.ipynb
├── README.md
├── requirements.txt
├── figures/
├── records/
└── report/
    └── Assignment_3_Report.pdf
```

## Requirements

Main Python libraries:

```text
numpy
pandas
matplotlib
scikit-image
scikit-learn
torch
torchvision
transformers
accelerate
Pillow
```

Install with:

```bash
pip install numpy pandas matplotlib scikit-image scikit-learn
pip install torch torchvision transformers accelerate pillow
```

A CUDA-enabled GPU is recommended for efficient multimodal model execution and U-Net training.

## How to Run

1. Clone the repository.
2. Open `AI_imaging.ipynb` in Google Colab or Jupyter.
3. Install the required dependencies.
4. Enable GPU acceleration when using Google Colab.
5. Run the notebook cells sequentially from beginning to end.
6. The notebook generates the figures, model results, JSON records and aggregated CSV.

## Outputs

### Visualisations
- Representative preprocessed images
- Intensity histogram
- Otsu segmentation
- U-Net input/ground-truth/prediction panels
- Training and validation curves

### Structured outputs
- `task1_vlm_comparison.json`
- `aggregated_records.csv`

## Limitations

The main limitations are the relatively small dataset, potential variation in LLM outputs, dependence of Otsu segmentation on image intensity distributions, and the absence of external clinical validation. Therefore, this project is an educational biomedical image-analysis pipeline rather than a clinical diagnostic system.

The most important future improvement would be external validation using a larger and more diverse dataset with independent expert annotations.

## References

1. Ronneberger, O., Fischer, P. and Brox, T. (2015). *U-Net: Convolutional Networks for Biomedical Image Segmentation*. MICCAI.
2. Otsu, N. (1979). *A Threshold Selection Method from Gray-Level Histograms*. IEEE Transactions on Systems, Man, and Cybernetics, 9(1), 62–66.
3. van der Walt, S. et al. (2014). *scikit-image: Image processing in Python*. PeerJ, 2, e453.
4. Paszke, A. et al. (2019). *PyTorch: An Imperative Style, High-Performance Deep Learning Library*. NeurIPS.
5. Qwen Team. *Qwen2.5-VL Technical Report*.
