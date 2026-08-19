Biomedical Image Analysis – Hybrid U-Net and LLM Pipeline
Overview

This project implements an end-to-end biomedical image-analysis pipeline for cell-nuclei images. It combines classical image processing, deep-learning segmentation and Large Language Model (LLM) interpretation.

The pipeline covers four main tasks:

Data preparation and exploratory data analysis
Multimodal LLM image description
Classical segmentation and quantitative feature extraction
U-Net segmentation and a hybrid LLM interpretation pipeline

The aim is to combine quantitative image analysis with structured LLM interpretation while maintaining transparency and auditability.

Tasks
Task 1 – Data Preparation and Multimodal LLM Description

The images are converted to grayscale and resized to 256 × 256 pixels. Exploratory analysis includes representative images and an intensity histogram.

A representative image is provided to a local multimodal model using Qwen2.5-VL-3B-Instruct.

The optimised prompt instructs the model to:

Describe observable features rather than diagnose
Avoid unsupported medical claims
Use "uncertain" when information cannot be determined
Return structured JSON

Required fields:

modality
tissue_type
notable_features
image_quality

A naive prompt is compared with the optimised prompt, and repeated runs are used to examine output variability.

Task 2 – Classical Features and LLM Interpretation

Classical image processing is performed using scikit-image.

The workflow includes:

Otsu thresholding
Morphological cleaning
Connected-component labelling
Region-level feature extraction using regionprops_table

Features include:

Area
Eccentricity
Solidity
Mean intensity
Object count

The numerical features are converted into a short summary and provided to the LLM without the original image.

The LLM produces a structured record containing:

n_objects
density_class
shape_regularity
quality_flag

This provides a numbers-first interpretation that is easier to audit.

Task 3 – U-Net Segmentation

A compact PyTorch U-Net is trained using the labelled training images and evaluated on the held-out validation set.

The notebook reports:

Metric	Result
Dice	0.997
IoU	0.993

The notebook also generates:

Training/loss curves
Dice curves
Input images
Ground-truth masks
Predicted masks
Validation comparisons

The U-Net results are also compared with classical Otsu segmentation.

Task 4 – Hybrid Pipeline

The complete pipeline is applied to the unseen test images:

Test Image
    ↓
U-Net Mask
    ↓
Region Features
    ↓
LLM Interpretation
    ↓
Structured JSON
    ↓
Narrative

Each test image produces a structured record containing:

image_id
n_objects
mean_area
density_class
quality_flag

The individual records are aggregated into a pandas DataFrame and saved as:

aggregated_records.csv

The quantitative segmentation and feature measurements are treated as the source of truth, while the LLM provides interpretation and narrative generation.

Main Results

The U-Net achieved strong validation performance:

Dice: 0.997
IoU: 0.993

The U-Net also achieved a higher mean Dice than the classical Otsu approach in the notebook comparison.

The multimodal LLM experiments demonstrated that prompt engineering improved the usefulness and structure of generated descriptions. Repeated runs also showed that LLM outputs can vary.

The hybrid pipeline reduces this risk by grounding LLM interpretation in quantitative image features.

Optimised Prompt Strategy

The prompts were designed to make the LLM outputs more structured and auditable.

For direct image description, the model was instructed to remain descriptive rather than diagnostic, use "uncertain" where appropriate, and return predefined JSON fields.

For the numbers-first and hybrid stages, the model received quantitative measurements rather than the original image. This reduces unsupported visual interpretation and keeps the measured image features as the primary evidence.

Project Structure
Biomedical-Image-Analysis/
│
├── AI_imaging.ipynb
├── README.md
├── requirements.txt
│
├── figures/
│   ├── representative_image.png
│   ├── intensity_histogram.png
│   ├── otsu_segmentation.png
│   ├── unet_predictions.png
│   └── training_curves.png
│
├── records/
│   ├── task1_vlm_comparison.json
│   └── aggregated_records.csv
│
└── report/
    └── Assignment_3_Report.pdf
Requirements

The project uses Python and the following main libraries:

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

Install the required packages using:

pip install numpy pandas matplotlib scikit-image scikit-learn
pip install torch torchvision transformers accelerate pillow

A CUDA-enabled GPU is recommended for running the multimodal model and training the U-Net efficiently.

How to Run
Clone the repository.
git clone <YOUR-GITHUB-REPOSITORY-URL>
Open the notebook:
AI_imaging.ipynb
Install the required dependencies.
Enable GPU acceleration if using Google Colab.
Run the notebook cells in order from beginning to end.
The notebook will generate the figures, model results, JSON records and aggregated CSV.
Outputs

The main outputs include:

Visualisations
Representative preprocessed images
Intensity histogram
Otsu segmentation
U-Net segmentation predictions
Ground-truth versus predicted masks
Training and validation curves
Structured outputs
task1_vlm_comparison.json
aggregated_records.csv

These outputs are used to support the results and discussion presented in the report.

Limitations

The main limitations are:

The dataset is relatively small.
Strong validation results may not generalise to other datasets.
LLM outputs can vary between repeated runs.
Classical Otsu segmentation depends on image intensity distributions.
The system has not undergone external clinical validation.

Therefore, this project should be considered an educational biomedical image-analysis pipeline rather than a clinical diagnostic system.

The most important future improvement would be external validation using a larger and more diverse dataset with independent expert annotations.

References
Ronneberger, O., Fischer, P. and Brox, T. (2015). U-Net: Convolutional Networks for Biomedical Image Segmentation. MICCAI.
Otsu, N. (1979). A Threshold Selection Method from Gray-Level Histograms. IEEE Transactions on Systems, Man, and Cybernetics, 9(1), 62–66.
van der Walt, S. et al. (2014). scikit-image: Image processing in Python. PeerJ, 2, e453.
Paszke, A. et al. (2019). PyTorch: An Imperative Style, High-Performance Deep Learning Library. NeurIPS.
Qwen Team. Qwen2.5-VL Technical Report.
