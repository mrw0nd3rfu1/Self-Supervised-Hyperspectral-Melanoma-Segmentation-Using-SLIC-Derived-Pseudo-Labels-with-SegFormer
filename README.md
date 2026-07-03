# Hyperspectral Image Segmentation for Melanoma Detection

A label-efficient pipeline for melanoma lesion segmentation in hyperspectral dermoscopy images using SLIC superpixels for pseudo-labeling and SegFormer for semantic segmentation.

## Abstract

Hyperspectral imaging (HSI) has emerged as a promising modality for melanoma detection due to its ability to capture rich spectral information beyond conventional RGB images. However, annotated hyperspectral datasets remain extremely limited, particularly at the pixel level. In this work, we present a label-efficient pipeline that leverages Simple Linear Iterative Clustering (SLIC) superpixels to generate pseudo ground truth masks for hyperspectral dermoscopy images. These pseudo masks are then used to fine-tune a SegFormer model for lesion segmentation in a self-supervised manner. We evaluated our approach on the publicly available HSIDermoscopy dataset and achieved an overall accuracy of 98.0%, Dice score of 0.79, and IoU of 0.66. These results demonstrate that the proposed method produces spatially coherent lesion masks without requiring manual annotations and achieves competitive segmentation performance compared to existing hyperspectral methods. Our findings highlight the potential of combining superpixel-based pseudo labeling with transformer-based architectures as a scalable and annotation-efficient solution for HSI-driven melanoma analysis.

## Dataset

The dataset used in this project is the **HSIDermoscopy** dataset, publicly available at:

🔗 [https://github.com/heugyy/HSIDermoscopy](https://github.com/heugyy/HSIDermoscopy)

### Dataset Structure

```
Dataset/
├── DNCube/       # Dysplastic Nevi hyperspectral cubes
├── MMCube/       # Malignant Melanoma hyperspectral cubes
└── OtherCube/    # Other lesion types
```

Each `.mat` file contains a 3D hyperspectral data cube with dimensions (height × width × spectral bands).

## Method Overview

### Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    PSEUDO-LABEL GENERATION                       │
├─────────────────────────────────────────────────────────────────┤
│  HSI Cube → Normalization → PCA (3 components) → SLIC Superpixels│
│     ↓                                                ↓          │
│  Spectral Signatures → KMeans Clustering → Pixel-wise Mask      │
│                              ↓                                  │
│            Morphological Refinement → Elliptical ROI Filter     │
│                              ↓                                  │
│                    Final Pseudo Ground Truth                    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    SEGFORMER TRAINING                           │
├─────────────────────────────────────────────────────────────────┤
│  PCA RGB Images + Pseudo Masks → Data Augmentation → SegFormer │
│                              ↓                                  │
│              Fine-tuned Lesion Segmentation Model               │
└─────────────────────────────────────────────────────────────────┘
```

### Key Components

1. **HSI Preprocessing**
   - Band-wise normalization to [0, 1] range
   - PCA dimensionality reduction (spectral bands → 3 components for RGB visualization)

2. **Pseudo-Label Generation**
   - SLIC superpixel segmentation (200 segments, compactness=10)
   - Mean spectral signature extraction per superpixel
   - KMeans clustering (k=2) for lesion/background separation
   - Morphological filtering (opening + closing with disk structuring elements)
   - Elliptical ROI masking to focus on central lesion region
   - Connected component analysis to select the central lesion blob

3. **SegFormer Training**
   - Base model: `nvidia/segformer-b0-finetuned-ade-512-512`
   - Binary segmentation (background=0, lesion=1)
   - Image resolution: 256×256
   - Training epochs: 30
   - Batch size: 4

## Installation

### Requirements

```bash
pip install numpy scipy scikit-learn scikit-image opencv-python matplotlib pillow
pip install torch torchvision
pip install transformers albumentations
pip install seaborn tqdm
```

### Dependencies

- Python 3.8+
- NumPy
- SciPy
- scikit-learn
- scikit-image
- OpenCV
- Matplotlib
- Pillow
- PyTorch
- Transformers (HuggingFace)
- Albumentations

## Usage

### 1. Data Preparation

Place the HSIDermoscopy dataset in a `Dataset/` directory with the following structure:

```
Dataset/
├── DNCube/
│   ├── sample1.mat
│   └── ...
├── MMCube/
│   ├── sample1.mat
│   └── ...
└── OtherCube/
    ├── sample1.mat
    └── ...
```

### 2. Generate Pseudo Labels

Run the notebook cells for pseudo-label generation. This will:
- Load all HSI cubes from the dataset
- Apply PCA for dimensionality reduction
- Generate pseudo masks using SLIC + KMeans
- Save RGB images and masks to `TrainData/` directory

```python
# Output directories
TrainData/
├── images/    # PCA-reduced RGB images (.png)
└── masks/     # Binary pseudo ground truth masks (.png)
```

### 3. Train SegFormer

The notebook includes SegFormer training with:
- 80/20 train-validation split
- Comprehensive evaluation metrics
- Visualization of predictions

### 4. Evaluate and Visualize

The pipeline generates:
- Training metrics plots
- Prediction visualizations
- Overlay comparisons (ground truth vs prediction)

## Results

| Metric | Score |
|--------|-------|
| **Accuracy** | 98.0% |
| **Dice Score** | 0.79 |
| **IoU (Jaccard)** | 0.66 |
| **Precision** | High |
| **Recall/Sensitivity** | High |
| **Specificity** | High |

## Output Structure

```
project/
├── Dataset/                 # Input HSI data
├── TrainData/
│   ├── images/             # PCA RGB images for training
│   └── masks/              # Pseudo ground truth masks
├── OutputMasks/            # Visualization outputs
├── SegformerResults/       # Model checkpoints and logs
├── Visualizations/
│   ├── training_metrics.png
│   ├── predictions.png
│   └── overlay_predictions.png
└── segformer_results/      # HuggingFace Trainer outputs
```

## Key Functions

| Function | Description |
|----------|-------------|
| `load_all_hsi()` | Load all .mat HSI cubes from dataset directories |
| `normalize_hsi_cube()` | Band-wise normalization to [0, 1] |
| `apply_pca()` | PCA dimensionality reduction |
| `generate_mask()` | SLIC + KMeans pseudo-label generation |
| `refine_mask()` | Select central lesion region |
| `apply_elliptical_roi()` | Create elliptical focus region |
| `evaluate_model_comprehensive()` | Full evaluation with all metrics |
| `visualize_predictions()` | Plot predictions vs ground truth |
| `visualize_overlay_predictions()` | Color overlay visualizations |

## Citation

If you use the HSIDermoscopy dataset, please cite appropriately:

```bibtex
@misc{hsidermoscopy,
  title={HSIDermoscopy Dataset},
  author={heugyy},
  url={https://github.com/heugyy/HSIDermoscopy}
}
```

This code accompanies the following peer-reviewed paper. If you use this repository in your research, please cite:

>A. Pandey et al., Big Data and Cognitive Computing, vol. 10, no. 3, p. 75, 2026. Available: https://www.mdpi.com/2504-2289/10/3/75

```bibtex
@article{pandey2026gnn,
  title   = {Graph Neural Networks and Language Models for Academic Performance Evaluation},
  author  = {Pandey, Abhinav and others},
  journal = {Big Data and Cognitive Computing},
  volume  = {10},
  number  = {3},
  pages   = {75},
  year    = {2026},
  url     = {https://www.mdpi.com/2504-2289/10/3/75}
}
```

(The journal “Big Data and Cognitive Computing” is an MDPI publication, ISSN 2504-2289.)

## License

Please refer to the original dataset license at [HSIDermoscopy](https://github.com/heugyy/HSIDermoscopy).

## Acknowledgments

- Dataset: [HSIDermoscopy](https://github.com/heugyy/HSIDermoscopy)
- SegFormer: NVIDIA pretrained model via HuggingFace Transformers
- SLIC implementation: scikit-image
