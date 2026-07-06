# Task 1 Report

## Neural Prediction of Mixed GMsFEM Basis Functions Using FNO and Attention U-Net

### Student Information

- Name: Shengzhe Zhao
- Project: SURF 2026
- Task: Full Model Reproduction and Ablation Study
- Date: June 2026

---

# 1. Introduction

The objective of this project is to reproduce and analyze a neural-network-based framework for predicting Mixed Generalized Multiscale Finite Element Method (Mixed GMsFEM) basis functions from permeability fields.

Traditional Mixed GMsFEM requires the construction of multiscale basis functions through local spectral problems, which can be computationally expensive during the offline stage. The goal of this project is to investigate whether deep learning can directly learn the mapping between permeability fields and multiscale basis functions, thereby reducing computational cost while preserving the accuracy of the multiscale method.

The overall mapping learned by the network is:

Permeability Field → Multiscale Basis Function

This work combines multiscale numerical methods and modern neural operator architectures to accelerate multiscale simulation workflows.

---

# 2. Background

Mixed Generalized Multiscale Finite Element Method (Mixed GMsFEM) is a numerical framework designed for solving flow problems in highly heterogeneous porous media.

The standard Mixed GMsFEM workflow consists of:

1. Constructing local snapshot spaces.
2. Solving local spectral decomposition problems.
3. Selecting dominant multiscale basis functions.
4. Assembling the global coarse-scale system.

Although the method is mathematically rigorous and highly accurate, the offline basis construction stage can be computationally expensive, especially when many permeability realizations are involved.

The motivation of this project is to replace repeated basis construction with a neural network approximation while maintaining the multiscale characteristics of the original method.

---

# 3. Neural Network Architecture

The complete model is implemented in:

```text
notebook_full_model.ipynb
```

The architecture contains four important components.

---

## 3.1 FNO-Style Preprocessing Block

The model first applies a Fourier Neural Operator (FNO) style preprocessing module.

Purpose:

- Capture long-range spatial interactions.
- Learn global permeability patterns.
- Extract frequency-domain information.

This module is designed to help the network recognize global structures that may influence basis functions over large spatial regions.

---

## 3.2 Attention U-Net

The backbone of the model is an Attention U-Net architecture.

Purpose:

- Learn hierarchical multiscale representations.
- Preserve local geometric structures.
- Reconstruct basis functions with high spatial accuracy.

The encoder extracts increasingly abstract features while the decoder reconstructs the target basis function.

---

## 3.3 Attention Gates

Attention gates are inserted into U-Net skip connections.

Purpose:

- Suppress irrelevant encoder features.
- Highlight important spatial information.
- Improve feature fusion between encoder and decoder.

Attention mechanisms allow the network to focus on physically meaningful regions of the permeability field.

---

## 3.4 Gradient Matching Loss

The training objective combines MSE loss and gradient loss:

L = L_MSE + λL_grad

where:

- L_MSE measures prediction error.
- L_grad measures gradient consistency.

The gradient term helps preserve local structures and prevents excessive smoothing of basis functions.

---

# 4. Dataset and Experimental Setup

Input:

```text
Permeability Field
```

Output:

```text
Mixed GMsFEM Basis Function
```

Grid Resolution:

```text
128 × 128
```

Selected Basis Function:

```text
Basis Index = 3
Basis Number = 4
```

---

# 5. Training Configuration

## Hardware

```text
GPU: Tesla V100-PCIE-32GB
```

## Hyperparameters

```text
Epochs: 60
Batch Size: 32
Learning Rate: 0.001
```

---

# 6. Full Model Results

The complete model was successfully trained and evaluated.

## Evaluation Metrics

| Metric | Value |
|----------|----------:|
| Test MSE | 0.01188 |
| Test R² | 0.93282 |
| Gradient Error | 0.04139 |

## Observations

The full model successfully learns the mapping between permeability fields and multiscale basis functions.

Visual inspection of the predicted basis functions indicates that the model captures both global structures and local spatial details. The prediction quality demonstrates that neural networks can effectively approximate the expensive basis-construction stage of Mixed GMsFEM.

---

# 7. Ablation Study Design

To understand the contribution of each component, three controlled ablation studies were performed.

All experiments used:

- The same dataset
- The same basis function
- The same optimizer
- The same learning rate
- The same batch size
- The same number of epochs

Only one component was removed in each experiment.

---

## Ablation A: No FNO

Notebook:

```text
notebook_ablation_no_fno.ipynb
```

Modification:

- Removed the FNO-style preprocessing block.

Purpose:

- Evaluate the importance of frequency-domain feature extraction.

---

## Ablation B: No Attention

Notebook:

```text
notebook_ablation_no_attention.ipynb
```

Modification:

- Removed attention gates from all U-Net skip connections.

Purpose:

- Evaluate the contribution of attention-based feature filtering.

---

## Ablation C: No Gradient Loss

Notebook:

```text
notebook_ablation_no_gradient_loss.ipynb
```

Modification:

- Removed gradient matching loss.

New objective:

L = L_MSE

Purpose:

- Evaluate the effect of gradient consistency constraints.

---

# 8. Ablation Study Results

The quantitative results are summarized below.

| Model | FNO | Attention | Gradient Loss | Test MSE | Test R² |
|---------|---------|---------|---------|---------:|---------:|
| Full Model | Yes | Yes | Yes | 0.01188 | 0.93282 |
| No FNO | No | Yes | Yes | **0.00952** | **0.94609** |
| No Attention | Yes | No | Yes | 0.01303 | 0.92627 |
| No Gradient Loss | Yes | Yes | No | 0.01274 | 0.92794 |

---

# 9. Discussion

The ablation study provides valuable insight into the contribution of each architectural component.

## Effect of Attention Gates

Removing attention gates increased the prediction error and reduced the R² score.

| Metric | Full Model | No Attention |
|----------|----------:|----------:|
| Test MSE | 0.01188 | 0.01303 |
| Test R² | 0.93282 | 0.92627 |

This indicates that attention gates improve feature selection and help the decoder focus on physically relevant information.

---

## Effect of Gradient Matching Loss

Removing gradient loss also reduced prediction quality.

| Metric | Full Model | No Gradient Loss |
|----------|----------:|----------:|
| Test MSE | 0.01188 | 0.01274 |
| Test R² | 0.93282 | 0.92794 |

This result suggests that gradient consistency helps preserve local structures and improves reconstruction accuracy.

---

## Effect of FNO Preprocessing

Interestingly, removing the FNO-style preprocessing block improved both MSE and R².

| Metric | Full Model | No FNO |
|----------|----------:|----------:|
| Test MSE | 0.01188 | 0.00952 |
| Test R² | 0.93282 | 0.94609 |

Possible explanations include:

- The selected basis function is dominated by local spatial structures.
- The dataset size may not be large enough to fully exploit FNO capabilities.
- The U-Net backbone already captures most useful information.
- Additional FNO layers may increase optimization difficulty.

Further experiments on different basis functions and larger datasets would be necessary before drawing a general conclusion regarding the usefulness of FNO modules.

---

# 10. Key Findings

The ablation study reveals three important observations:

1. Attention gates improve prediction accuracy.
2. Gradient matching loss improves reconstruction quality.
3. The current FNO preprocessing block does not improve performance for the selected basis function and may even reduce prediction accuracy.

Among all tested models, the No-FNO variant achieved the best performance.

| Metric | Value |
|----------|----------:|
| Best Model | No FNO |
| Test MSE | 0.00952 |
| Test R² | 0.94609 |

---

# 11. Conclusion

This project successfully reproduced the complete neural framework for predicting Mixed GMsFEM basis functions from permeability fields.

The full model and all three ablation models were implemented, trained, and evaluated under identical experimental conditions.

The results demonstrate that:

- Neural networks can effectively approximate multiscale basis functions.
- Attention mechanisms contribute positively to prediction performance.
- Gradient matching loss improves geometric reconstruction quality.
- The current FNO preprocessing block does not provide additional benefit for the selected basis function.

These findings provide useful insight into the design of neural architectures for accelerating multiscale finite element methods.

---

# 12. Future Work

Future work may include:

1. Testing additional basis functions.
2. Increasing dataset size.
3. Exploring alternative neural operator architectures.
4. Comparing inference speed with traditional Mixed GMsFEM basis construction.
5. Investigating model generalization across permeability distributions.
6. Performing hyperparameter optimization for FNO modules.

The long-term goal is to develop efficient neural surrogates that accelerate multiscale simulation while preserving physical accuracy.