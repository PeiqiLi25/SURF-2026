# Week 01 Progress Report

## SURF 2026 Project

### Neural Prediction of Mixed GMsFEM Basis Functions

**Student:** Shengzhe Zhao
**Week:** Week 02
**Date:** June 2026

---

# 1. Overview

This week, I focused on understanding and reproducing the full neural-network-based model for predicting Mixed GMsFEM multiscale basis functions from permeability fields.

The main objective was to run the complete model pipeline, prepare three ablation study notebooks, and analyze the contribution of different model components, including the FNO-style preprocessing block, attention gates, and gradient matching loss.

The project studies the mapping:

```text
Permeability Field → Mixed GMsFEM Basis Function
```

This approach aims to accelerate the offline basis construction stage in Mixed GMsFEM using deep learning.

---

# 2. Work Completed

During this week, I completed the following tasks:

1. Read the project instructions and clarified the requirements for Task 1.
2. Reviewed the full model notebook:

   ```text
   notebook_full_model.ipynb
   ```
3. Understood the main structure of the model, including:

   * FNO-style preprocessing block
   * Attention U-Net
   * Attention gates
   * Gradient matching loss
4. Ran the full model experiment.
5. Prepared and ran three ablation study notebooks:

   ```text
   notebook_ablation_no_fno.ipynb
   notebook_ablation_no_attention.ipynb
   notebook_ablation_no_gradient_loss.ipynb
   ```
6. Fixed implementation issues in the no-attention ablation model.
7. Collected and summarized the experimental results.
8. Prepared the final Task 1 report:

   ```text
   task1.md
   ```

---

# 3. Technical Understanding

This week, I developed a clearer understanding of the connection between Mixed GMsFEM and neural-network-based basis prediction.

In traditional Mixed GMsFEM, multiscale basis functions are constructed through local snapshot spaces and local spectral decomposition. This process is accurate but computationally expensive.

The neural-network model attempts to replace this repeated offline basis construction stage by directly learning the relationship between the permeability field and the corresponding multiscale basis function.

The full model contains several important components:

## 3.1 FNO-Style Preprocessing Block

The FNO-style block is designed to extract global and frequency-domain information from the permeability field.

Its purpose is to help the model capture nonlocal structures that may affect the multiscale basis function.

## 3.2 Attention U-Net

The Attention U-Net is used as the main spatial reconstruction network.

Its encoder extracts hierarchical features, while its decoder reconstructs the predicted basis function.

## 3.3 Attention Gates

Attention gates are used in skip connections to filter encoder features before passing them into the decoder.

The goal is to emphasize important spatial structures and suppress irrelevant information.

## 3.4 Gradient Matching Loss

The gradient matching loss is added to the standard MSE loss.

It helps the model preserve local spatial variations and avoid overly smooth predictions.

---

# 4. Experimental Setup

The selected basis function was:

```text
Basis Index = 3
Basis Number = 4
```

The training configuration was:

```text
Epochs: 60
Batch Size: 32
Learning Rate: 0.001
Optimizer: Adam
GPU: Tesla V100-PCIE-32GB
```

The full model and all ablation models were trained under the same experimental settings to ensure fair comparison.

---

# 5. Full Model Result

The full model was successfully trained and evaluated.

| Metric         |   Value |
| -------------- | ------: |
| Test MSE       | 0.01188 |
| Test R²        | 0.93282 |
| Gradient Error | 0.04139 |

The full model successfully learned the mapping from permeability fields to multiscale basis functions. The result shows that the model is able to capture both global spatial patterns and local basis-function structures.

---

# 6. Ablation Study Results

Three ablation studies were conducted to evaluate the contribution of individual components.

| Model            | FNO | Attention | Gradient Loss |    Test MSE |     Test R² |
| ---------------- | --- | --------- | ------------- | ----------: | ----------: |
| Full Model       | Yes | Yes       | Yes           |     0.01188 |     0.93282 |
| No FNO           | No  | Yes       | Yes           | **0.00952** | **0.94609** |
| No Attention     | Yes | No        | Yes           |     0.01303 |     0.92627 |
| No Gradient Loss | Yes | Yes       | No            |     0.01274 |     0.92794 |

---

# 7. Discussion

The ablation study gave several useful observations.

## 7.1 Effect of Attention Gates

Removing attention gates caused the model performance to decrease.

Compared with the full model:

```text
Full Model MSE: 0.01188
No Attention MSE: 0.01303
```

The R² score also decreased from 0.93282 to 0.92627.

This suggests that attention gates help the model select useful features from skip connections and improve the reconstruction of multiscale basis functions.

## 7.2 Effect of Gradient Matching Loss

Removing gradient loss also reduced the performance.

Compared with the full model:

```text
Full Model MSE: 0.01188
No Gradient Loss MSE: 0.01274
```

This indicates that gradient matching loss provides useful structural information and helps preserve local spatial variations in the predicted basis functions.

## 7.3 Effect of FNO-Style Preprocessing

Interestingly, the model without the FNO-style preprocessing block achieved the best performance.

Compared with the full model:

```text
Full Model MSE: 0.01188
No FNO MSE: 0.00952
```

The R² score also improved from 0.93282 to 0.94609.

This suggests that, for the selected basis function, the FNO-style preprocessing block may not provide additional benefit. Possible reasons include:

1. The selected basis function may be dominated by local spatial structures.
2. The U-Net backbone may already capture sufficient information.
3. The FNO block may introduce additional optimization difficulty.
4. More experiments on other basis functions are needed before making a general conclusion.

---

# 8. Problems Encountered

During the no-attention ablation study, I encountered an implementation issue:

```text
AttributeError: 'UpNoAtt' object has no attribute 'att'
```

The reason was that the new `UpNoAtt` class still contained the line:

```python
skip = self.att(x, skip)
```

This line belonged to the original attention-based upsampling module. Since the no-attention version should not contain attention gates, the line was removed and the encoder skip feature was directly concatenated with the decoder feature.

The corrected logic was:

```python
x = torch.cat([skip, x], dim=1)
return self.conv(x)
```

After this correction, the no-attention ablation model ran successfully.

---

# 9. GitHub Documentation Progress

The following files were prepared for GitHub submission:

```text
task1.md
week-01-progress.md
notebook_full_model.ipynb
notebook_ablation_no_fno.ipynb
notebook_ablation_no_attention.ipynb
notebook_ablation_no_gradient_loss.ipynb
```

The project files are organized to document both the experimental process and the final results.

Large raw datasets, large model checkpoints, cache files, and private information should not be committed to GitHub.

---

# 10. Summary of This Week

This week, I completed the main Task 1 workflow:

1. Ran the full model.
2. Prepared and ran three ablation study notebooks.
3. Debugged the no-attention model.
4. Collected quantitative results.
5. Compared the full model with ablation variants.
6. Wrote the Task 1 report.
7. Prepared this weekly progress report.

The most important finding is that attention gates and gradient matching loss improve model performance, while the FNO-style preprocessing block did not improve the result for the selected basis function.

The best-performing model in this week’s experiments was:

```text
No FNO model
Test MSE = 0.00952
Test R² = 0.94609
```

---

# 11. Plan for Next Week

Next week, I plan to:

1. Test additional basis functions.
2. Compare whether the No-FNO result remains best across other basis functions.
3. Add more visualization comparisons between prediction, ground truth, and error fields.
4. Study frequency-domain error patterns.
5. Improve the final experimental report with more figures.
6. Continue organizing the GitHub repository with clear commit messages.

---

# 12. Conclusion

This week established a complete experimental workflow for neural prediction of Mixed GMsFEM basis functions.

The full model and three ablation models were successfully executed and compared. The results provide useful insight into the roles of attention gates, gradient loss, and FNO-style preprocessing in basis-function prediction.

This progress provides a solid foundation for further experiments on model generalization, additional basis functions, and spectral error analysis.
