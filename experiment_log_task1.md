# Task 1 Experiment Log

## Purpose

This task studies how to predict one mixed GMsFEM multiscale basis function from a permeability field using a neural operator model. The baseline full model was first validated in quick mode, then trained in full mode on CUDA. Three ablation studies were also completed in full mode to evaluate the contributions of the FNO-style preprocessing block, attention-gated skip connections, and gradient matching loss.

## Basis Selection

* Selected basis function: `basis 04`
* Zero-based index: `3`
* One-based index: `4`

## Local Quick Validation

Quick-mode validation was completed locally on Mac to verify:

* environment setup
* data loading
* model construction
* training loop
* metric export
* visualization
* output saving

Quick-mode runs were completed for:

* `notebook_full_model.ipynb`
* `notebook_ablation_no_fno.ipynb`
* `notebook_ablation_no_attention.ipynb`
* `notebook_ablation_no_gradient_loss.ipynb`

Quick comparison file:

* `outputs/task1_quick_metrics_comparison.csv`

These quick runs were used only for workflow verification and not for final comparison.

---

## Full Model

* Notebook: `notebook_full_model.ipynb`
* RUN_MODE: `full`
* Device: `CUDA`
* Basis function: `basis 04`
* Train size: `3500`
* Validation size: `300`
* Test size: `100`
* Batch size: `32`
* Epochs: `60`
* Learning rate: `1e-3`
* Gradient loss weight: `0.1`
* Output directory: `outputs/teaching_single_basis/basis_04`
* Checkpoint: `best_model/basis_04_best.pt`
* Status: completed successfully

### Full Test Metrics

* MSE: `0.012038`
* RMSE: `0.109716`
* MAE: `0.037017`
* R2: `0.931886`
* Relative L2: `0.260202`
* Gradient MSE: `0.041623`
* Max absolute error: `1.784681`
* Number of test samples: `100`

---

## Ablation: No FNO

* Notebook: `notebook_ablation_no_fno.ipynb`
* RUN_MODE: `full`
* Device: `CUDA`
* Change: removed the FNO-style preprocessing block by setting `use_fno=False`
* Train size: `3500`
* Validation size: `300`
* Test size: `100`
* Batch size: `32`
* Epochs: `60`
* Learning rate: `1e-3`
* Gradient loss weight: `0.1`
* Output directory: `outputs/ablation_no_fno/basis_04`
* Checkpoint: `best_model/ablation_no_fno/basis_04_best.pt`
* Status: completed successfully

### Full Test Metrics

* MSE: `0.009339`
* RMSE: `0.096638`
* MAE: `0.039342`
* R2: `0.947134`
* Relative L2: `0.228946`
* Gradient MSE: `0.031159`
* Max absolute error: `1.793450`
* Number of test samples: `100`

---

## Ablation: No Attention

* Notebook: `notebook_ablation_no_attention.ipynb`
* RUN_MODE: `full`
* Device: `CUDA`
* Change: replaced attention-gated skip connections with ordinary U-Net skip connections
* Train size: `3500`
* Validation size: `300`
* Test size: `100`
* Batch size: `32`
* Epochs: `60`
* Learning rate: `1e-3`
* Gradient loss weight: `0.1`
* Output directory: `outputs/ablation_no_attention/basis_04`
* Checkpoint: `best_model/ablation_no_attention/basis_04_best.pt`
* Status: completed successfully

### Full Test Metrics

* MSE: `0.013008`
* RMSE: `0.114052`
* MAE: `0.039226`
* R2: `0.926424`
* Relative L2: `0.272993`
* Gradient MSE: `0.046128`
* Max absolute error: `1.791959`
* Number of test samples: `100`

---

## Ablation: No Gradient Loss

* Notebook: `notebook_ablation_no_gradient_loss.ipynb`
* RUN_MODE: `full`
* Device: `CUDA`
* Change: trained with MSE loss only, without gradient matching loss
* Train size: `3500`
* Validation size: `300`
* Test size: `100`
* Batch size: `32`
* Epochs: `60`
* Learning rate: `1e-3`
* Gradient loss weight: `0.0`
* Output directory: `outputs/ablation_no_gradient_loss/basis_04`
* Checkpoint: `best_model/ablation_no_gradient_loss/basis_04_best.pt`
* Status: completed successfully

### Full Test Metrics

* MSE: `0.012894`
* RMSE: `0.113552`
* MAE: `0.038463`
* R2: `0.927099`
* Relative L2: `0.269931`
* Gradient MSE: `0.045090`
* Max absolute error: `1.785915`
* Number of test samples: `100`

---

## Full-mode Comparison

Comparison file:

* `outputs/task1_full_metrics_comparison.csv`

### Summary

For basis function 04, the no-FNO ablation achieved the best overall performance among the four full-mode runs. It obtained the lowest MSE, lowest RMSE, highest R2, lowest Relative L2, and lowest Gradient MSE. The baseline full model performed strongly and clearly outperformed both the no-attention and no-gradient-loss variants on most metrics. This suggests that attention gates and gradient matching loss both contribute positively in this task setting.

At the same time, the no-FNO result shows that the FNO-style preprocessing block is not guaranteed to improve empirical performance in every case. Its usefulness may depend on the selected basis function, optimization behavior, or interaction with other architectural components.

## Final Status

Task 1 full-mode baseline training and all three ablation studies have been completed successfully, and the final comparison table has been generated.
