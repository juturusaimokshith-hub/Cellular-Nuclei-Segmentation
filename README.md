
# Cellular Nuclei Segmentation and Analysis

A deep learning-based computer vision project for segmenting cellular nuclei from microscopy images and separating individual nuclei using a combination of U-Net-based segmentation and Watershed post-processing.

---

## 1. Problem Statement

In microscopy images, cellular nuclei can appear close together or touch one another, making it difficult to accurately identify the boundaries of individual nuclei.

The goal of this project is to build a segmentation pipeline that:

- Identifies nuclei in microscopy images
- Produces accurate pixel-level segmentation masks
- Separates touching or overlapping nuclei
- Evaluates the quality of the segmentation
- Provides visual outputs for analyzing the results

---

## 2. Approach

The project uses a two-stage segmentation approach.

### Stage 1 — Semantic Segmentation

An Enhanced U-Net model is used to identify pixels belonging to cellular nuclei.

The model uses:

- **EfficientNet-B7** as the encoder
- **ImageNet pretrained weights** for feature extraction
- **Squeeze-and-Excitation (SE) blocks** to improve feature representation
- A U-Net decoder to reconstruct the segmentation mask

The output of this stage is a **binary nuclei mask**:

```text
Background → 0
Nuclei     → 1
````

### Stage 2 — Instance Segmentation

A binary mask may contain multiple nuclei connected together when nuclei are touching.

To separate these individual nuclei, the project applies **marker-controlled Watershed segmentation**.

```text
Input Image
     ↓
Preprocessing
     ↓
Enhanced U-Net
     ↓
Binary Nuclei Mask
     ↓
Marker Generation
     ↓
Watershed Segmentation
     ↓
Individual Nuclei Instances
```

This combination allows the project to perform both:

* **Semantic segmentation** — Where are the nuclei?
* **Instance segmentation** — Which pixels belong to each individual nucleus?

---

## 3. Dataset

The project uses the **PanNuke dataset**, which contains microscopy/histopathology images with cellular nuclei annotations.

The dataset is not included in this repository.

Dataset information and access details are provided in:

```text
dataset_info/README.md
```

---

## 4. Model Architecture

### Enhanced U-Net

The main segmentation model is an Enhanced U-Net.

```text
                    Input Image
                         │
                         ▼
               EfficientNet-B7
                  Encoder
                         │
              ┌──────────┴──────────┐
              │                     │
        Feature Extraction     SE Blocks
              │                     │
              └──────────┬──────────┘
                         ▼
                   U-Net Decoder
                         │
                         ▼
                 Nuclei Probability
                         │
                         ▼
                  Binary Nuclei Mask
```

### Why EfficientNet-B7?

EfficientNet-B7 is used as the encoder to extract meaningful visual features from the microscopy images while benefiting from pretrained ImageNet representations.

### Why SE Blocks?

Squeeze-and-Excitation blocks help the network emphasize informative feature channels and suppress less useful ones.

### Why U-Net?

U-Net combines high-level semantic information from the encoder with spatial information through skip connections, making it suitable for pixel-level segmentation.

---

## 5. Loss Function

The model uses a combined loss consisting of:

### Binary Cross Entropy Loss

Helps classify individual pixels as background or nuclei.

### Dice Loss

Encourages better overlap between predicted nuclei regions and the ground-truth masks.

### Edge Loss

Encourages better preservation of nuclei boundaries.

The overall training objective combines:

```text
Total Loss =
    BCE Loss
    + Dice Loss
    + Edge Loss
```

This combination is designed to improve both region overlap and boundary quality.

---

## 6. Data Processing

The pipeline includes:

* Image loading
* Mask processing
* Distance-map generation
* Image normalization
* Data augmentation
* Patch-based processing

Albumentations is used for data augmentation during training.

---

## 7. Training Configuration

| Parameter          | Value                          |
| ------------------ | ------------------------------ |
| Model              | Enhanced U-Net                 |
| Encoder            | EfficientNet-B7                |
| Pretrained Weights | ImageNet                       |
| Optimizer          | AdamW                          |
| Learning Rate      | 1e-4                           |
| Batch Size         | 16                             |
| Loss               | BCE + Dice + Edge              |
| Augmentation       | Albumentations                 |
| Training           | 100 epochs + 50 resumed epochs |
| Early Stopping     | Enabled                        |
| LR Scheduler       | ReduceLROnPlateau              |
| Checkpointing      | Enabled                        |
| TensorBoard        | Enabled                        |

---

## 8. Results

The model was evaluated on validation and test data.

| Metric                 |    Validation |          Test |
| ---------------------- | ------------: | ------------: |
| Dice Score             |        0.8066 |        0.8048 |
| Average Inference Time | 0.324 s/patch | 0.324 s/patch |

### What does the Dice Score mean?

Dice Score measures the overlap between the predicted segmentation and the ground-truth segmentation.

A higher Dice Score means that the predicted nuclei regions have better overlap with the actual nuclei regions.

The model achieved:

**Test Dice Score: 0.8048**

---

## 9. Watershed Instance Segmentation

After the U-Net produces the binary nuclei mask, connected nuclei may still appear as one region.

Marker-controlled Watershed is therefore applied to divide connected nuclei into separate instances.

```text
U-Net Prediction
       ↓
Binary Mask
       ↓
Distance / Marker Generation
       ↓
Watershed
       ↓
Separate Nuclei
```

This step is particularly useful when multiple nuclei are touching each other.

---

## 10. Inference Performance

The average inference time was approximately:

**0.324 seconds per patch**

The pipeline supports batch processing and generates segmentation outputs that can be visually inspected and quantitatively evaluated.

---

## 11. Project Structure

```text
Cellular-Nuclei-Segmentation/
│
├── notebook/
│   └── Cellular_Nuclei_Segmentation.ipynb
│
├── src/
│   ├── data_pipeline.py
│   ├── enhanced_unet.py
│   ├── evaluation_metrics.py
│   ├── losses.py
│   ├── postprocessing_utils.py
│   ├── segmentation_pipeline.py
│   ├── se_blocks.py
│   ├── trainer.py
│   ├── visualization_tools.py
│   └── watershed_segmentation.py
│
├── scripts/
│   ├── config.py
│   ├── train_unet.py
│   ├── final_benchmark.py
│   └── test_complete_pipeline.py
│
├── models/
├── results/
├── dataset_info/
│   └── README.md
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## 12. Technologies Used

* Python
* PyTorch
* U-Net
* EfficientNet-B7
* Segmentation Models PyTorch
* OpenCV
* Scikit-image
* Albumentations
* NumPy
* Matplotlib
* Scikit-learn
* TensorBoard

---

## 13. How to Run

### Install Dependencies

```bash
pip install -r requirements.txt
```

### Train the Model

```bash
python scripts/train_unet.py
```

### Run Benchmarking

```bash
python scripts/final_benchmark.py
```

### Test the Complete Pipeline

```bash
python scripts/test_complete_pipeline.py
```

The complete workflow can also be explored through:

```text
notebook/Cellular_Nuclei_Segmentation.ipynb
```

---

## 14. Key Takeaways

This project demonstrates an end-to-end image segmentation workflow:

* Designing an enhanced U-Net architecture
* Using transfer learning with EfficientNet-B7
* Improving feature extraction using SE blocks
* Combining multiple loss functions for segmentation
* Performing pixel-level nuclei segmentation
* Separating touching nuclei using Watershed
* Evaluating segmentation using Dice Score
* Measuring inference performance
* Organizing a reproducible deep learning pipeline

---

## 15. Future Improvements

Possible improvements include:

* Improving segmentation performance on densely packed nuclei
* Experimenting with different segmentation architectures
* Optimizing Watershed marker generation
* Evaluating additional instance-segmentation approaches
* Improving inference speed
* Testing the pipeline on additional microscopy datasets

---

