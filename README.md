# Technical and Evaluation Report
Submission by: Dr. S.Kiruthika - Assistant Professor, VIT Chennai. Dr. J. Joshan Athanesious, VIT Chennai. Linga Adithya K, student - VIT Chennai.


# 1. Technical Report

## 1.1 Problem Statement

The objective of this work is to perform automatic glacial lake segmentation from satellite imagery using deep learning techniques. The task is formulated as a binary semantic segmentation problem where each pixel in an input image is classified as either background or glacial lake.

Accurate segmentation of glacial lakes is important for environmental monitoring, glacier dynamics analysis, water resource estimation, and Glacial Lake Outburst Flood (GLOF) risk assessment. Since only a limited amount of labeled data is available, a semi-supervised learning framework was adopted to leverage both labeled and unlabeled satellite imagery.

---

# 2. Data Preprocessing

## 2.1 Image Normalization

Input RGB images were normalized by scaling pixel values from the range [0,255] to [0,1]. Normalization improves numerical stability and accelerates convergence during optimization.

## 2.2 Mask Preparation

Ground-truth masks were converted into binary segmentation masks:

- 0 → Background
- 1 → Glacial Lake

All non-zero pixels were assigned to the foreground class.

## 2.3 Tensor Conversion

Images were converted from:


Height × Width × Channels


to:


Channels × Height × Width


to satisfy PyTorch input requirements.

---

# 3. Model Architecture

The segmentation model was implemented using the Segmentation Models PyTorch (SMP) library.

## 3.1 U-Net Architecture

U-Net is a fully convolutional encoder-decoder architecture specifically designed for semantic segmentation tasks. It consists of:

- Encoder for feature extraction
- Decoder for mask reconstruction
- Skip connections for preserving spatial information

The workflow of the model is:


Input Image
↓
ResNet-34 Encoder
↓
Feature Extraction
↓
Bottleneck Features
↓
U-Net Decoder
↓
Segmentation Mask


The encoder extracts hierarchical features from satellite imagery, while the decoder reconstructs the final segmentation mask at the original resolution.

## 3.2 ResNet-34 Encoder

ResNet-34 was selected as the encoder backbone.

The encoder was initialized with ImageNet pretrained weights, providing the following advantages:

- Faster convergence
- Improved feature representation
- Better generalization
- Reduced training time

The encoder progressively learns:


Edges
↓
Textures
↓
Shapes
↓
Water Bodies
↓
Glacial Lake Structures


## 3.3 Skip Connections

A key component of U-Net is the use of skip connections between encoder and decoder layers.

These connections:

- Preserve fine spatial details
- Improve boundary localization
- Enhance reconstruction quality
- Improve segmentation of small lake regions

Without skip connections, significant spatial information may be lost during downsampling.

---

# 4. Loss Function

The model employs a hybrid loss function consisting of:

1. Dice Loss
2. BCEWithLogitsLoss

## 4.1 Dice Loss

Dice Loss directly optimizes overlap between predicted masks and ground-truth masks.

Advantages:

- Effective for segmentation tasks
- Handles class imbalance
- Improves region overlap

Since lake pixels occupy a relatively small portion of the image, Dice Loss is particularly beneficial.

## 4.2 BCEWithLogitsLoss

Binary Cross Entropy with Logits Loss performs pixel-level binary classification.

Advantages:

- Stable optimization
- Strong gradient propagation
- Reliable convergence

## 4.3 Combined Loss

The total loss is computed as:


Total Loss = Dice Loss + BCEWithLogitsLoss


This combines overlap optimization with stable pixel-wise supervision.

---

# 5. Data Augmentation

To improve model generalization and reduce overfitting, the following augmentations were applied during training:

| Augmentation | Probability |
|-------------|-------------|
| Horizontal Flip | 0.5 |
| Vertical Flip | 0.5 |
| Random 90° Rotation | 0.5 |
| Random Brightness & Contrast | 0.3 |

These augmentations expose the model to a wider variety of visual conditions and orientations.

---

# 6. Semi-Supervised Learning Framework

Due to the limited availability of labeled data, a semi-supervised learning strategy was implemented.

## 6.1 Pseudo-Label Generation

A baseline U-Net model was first trained using the labeled dataset.

The trained model was then used to generate segmentation masks for unlabeled images.

Workflow:


Labeled Images
↓
Train Baseline U-Net
↓
Predict Unlabeled Images
↓
Generate Pseudo Labels
↓
Retrain Model


To improve pseudo-label quality:

- Low-confidence predictions were removed
- Small isolated regions were filtered
- Only reliable pseudo-labels were retained

This significantly increased the effective training dataset size.

---

# 7. Mean Teacher Framework

To improve stability and pseudo-label quality, a Mean Teacher framework was incorporated.

The framework consists of:

### Student Network

The student network is updated using standard backpropagation.

### Teacher Network

The teacher network is updated using Exponential Moving Average (EMA) of student weights.

Benefits include:

- Stable pseudo-label generation
- Reduced prediction noise
- Improved consistency
- Better utilization of unlabeled data

The teacher network serves as a more stable version of the student model during training.

---

# 8. Bidirectional Copy-Paste (BCP) Augmentation

Bidirectional Copy-Paste augmentation was employed to improve data diversity.

The method creates new training samples by exchanging foreground regions between labeled and pseudo-labeled images.

### Forward Copy-Paste


Labeled Region
↓
Paste into Unlabeled Image


### Reverse Copy-Paste


Pseudo-Labeled Region
↓
Paste into Labeled Image


Advantages:

- Increased sample diversity
- Improved regularization
- Enhanced robustness
- Better generalization performance

---

# 9. Training Configuration

| Hyperparameter | Value |
|---------------|--------|
| Learning Rate | 1e-4 |
| Batch Size | 4 |
| Number of Epochs | 40 |
| Optimizer | Adam |
| Loss Function | Dice Loss + BCEWithLogitsLoss |
| Train Split | 80% |
| Validation Split | 20% |

The Adam optimizer was selected due to its adaptive learning rate mechanism and strong performance in segmentation applications.

---

# 10. Test-Time Augmentation (TTA)

To improve prediction robustness, Test-Time Augmentation was employed during inference.

Predictions were generated using:

1. Original Image
2. Horizontally Flipped Image
3. Vertically Flipped Image

The final segmentation mask was obtained by averaging the predicted probabilities from all augmented versions.

Benefits include:

- Reduced prediction variance
- Improved robustness
- Better segmentation consistency

---

# 11. Evaluation Methodology

The trained model was evaluated on the validation dataset using standard semantic segmentation metrics.

The evaluation process consisted of:

1. Applying sigmoid activation
2. Thresholding predictions at 0.5
3. Converting probability maps into binary masks
4. Comparing predictions with ground-truth masks

Metrics were computed for each validation image and averaged across the validation set.

---

# 12. Evaluation Results

| Metric | Score |
|----------|----------|
| Intersection over Union (IoU) | 65.09% |
| F1 Score (Dice Score) | 74.31% |
| Precision | 70.95% |
| Recall | 79.93% |
| Accuracy | 99.61% |
| Cohen's Kappa | 74.10% |

---

# 13. Discussion of Results

The model achieved an IoU score of 65.09%, indicating substantial overlap between predicted and ground-truth lake regions.

The Dice/F1 score of 74.31% demonstrates strong segmentation performance and confirms the effectiveness of the proposed framework.

The recall value of 79.93% exceeds the precision value of 70.95%, suggesting that the model successfully captures the majority of lake pixels while maintaining a reasonable false positive rate.

A Cohen's Kappa score of 74.10% indicates substantial agreement between model predictions and manually annotated masks.

The high accuracy value is expected because the majority of pixels belong to the background class. Consequently, IoU, Dice Score, Precision, Recall, and Cohen's Kappa provide a more informative assessment of segmentation quality.

The performance gains demonstrate the effectiveness of incorporating pseudo-labeling, Mean Teacher learning, Bidirectional Copy-Paste augmentation, and Test-Time Augmentation into the segmentation pipeline.

---

# 14. Conclusion

This work presents a semi-supervised deep learning framework for glacial lake segmentation using a U-Net architecture with a ResNet-34 encoder backbone.

The proposed framework combines:

- Transfer Learning
- Pseudo-Labeling
- Mean Teacher Learning
- Bidirectional Copy-Paste Augmentation
- Test-Time Augmentation

to effectively utilize both labeled and unlabeled satellite imagery.

Experimental results demonstrate that the proposed approach achieves strong segmentation performance despite limited labeled data availability. The combination of supervised and semi-supervised learning techniques enables accurate extraction of glacial lake regions and provides a robust framework for remote sensing segmentation applications.
## Validation Results

The final model was evaluated on an independent validation dataset using Test-Time Augmentation (TTA) with horizontal and vertical flipping. Predictions were converted to binary masks using a threshold of 0.5 and evaluated against ground-truth annotations.

### Dataset-Level Performance

| Metric                        |      Score |
| ----------------------------- | ---------: |
| IoU (Intersection over Union) | **50.79%** |
| Dice Score (F1 Score)         | **67.36%** |
| Precision                     | **71.12%** |
| Recall                        | **63.98%** |
| Accuracy                      | **98.63%** |
| Cohen's Kappa                 | **66.66%** |

### Key Observations

* Strong pixel-level classification accuracy (**98.63%**).
* Good segmentation overlap with a Dice Score of **67.36%**.
* Higher precision than recall indicates fewer false positives while maintaining robust lake detection.
* Cohen's Kappa of **66.66%** demonstrates substantial agreement with ground-truth masks.
* Results confirm the effectiveness of the semi-supervised learning pipeline, including pseudo-labeling, Mean Teacher training, Bidirectional Copy-Paste augmentation, and Test-Time Augmentation.

These results demonstrate the model's ability to generalize to previously unseen satellite imagery and reliably segment glacial lake regions.

Validation Result Images: https://drive.google.com/drive/folders/1bgaH0fXQ7-Fjeb51G-wVR2DTNIbMlFDC?usp=sharing

# Segmentation masks of all the images: https://drive.google.com/drive/folders/1_XoKI2QwJqksLKMCc-w_cELq7DWTlpmn?usp=sharing
# Trained Models: https://drive.google.com/drive/folders/1mz2PI8zhoNIigY70jdinwdDS7cykSyv_?usp=sharing
# Explanation Video: https://drive.google.com/file/d/1ate3AL6HXG_VCOPPecE4ZuE6fWEq1CjJ/view?usp=sharing

