# Task 1 Learning Notes

## 1. What is this task?

This task trains a neural operator model to predict one mixed GMsFEM multiscale basis function from a permeability field.

Input:

* permeability field `kappa`

Target:

* one selected basis function from the multiscale basis dataset

In this task, the selected target is:

* `BASIS_INDEX = 3`
* basis function `4` in one-based numbering
* saved as `basis 04`

So the learned mapping is:

`permeability field -> one multiscale basis function`

---

## 2. Why are there quick and full modes?

### Quick mode

Quick mode uses a much smaller dataset and fewer epochs.

Purpose:

* verify the environment
* check data loading
* make sure the training loop works
* confirm outputs, metrics, and plots are saved correctly

Quick mode is **not** used for final scientific comparison.

### Full mode

Full mode uses the intended larger train/validation/test split and longer training.

Purpose:

* obtain meaningful final metrics
* compare the full model against ablations fairly
* evaluate the contribution of each design choice

In this task, full mode uses:

* train size: `3500`
* validation size: `300`
* test size: `100`
* epochs: `60`
* batch size: `32`

---

## 3. What is the baseline full model?

The baseline model contains four main ideas:

1. FNO-style preprocessing block
2. Attention U-Net backbone
3. Standard MSE loss
4. Gradient matching loss

The baseline tries to combine:

* nonlocal / spectral processing
* encoder-decoder spatial feature learning
* pointwise error minimization
* local shape / gradient consistency

---

## 4. What is the purpose of each ablation?

### No FNO

File:

* `notebook_ablation_no_fno.ipynb`

Change:

* `use_fno=False`

Purpose:

* remove the FNO-style spectral preprocessing block
* test whether this nonlocal preprocessing actually helps performance

### No Attention

File:

* `notebook_ablation_no_attention.ipynb`

Change:

* replace attention-gated skip connections with ordinary U-Net skip connections

Purpose:

* test whether attention gates improve feature selection in skip connections

### No Gradient Loss

File:

* `notebook_ablation_no_gradient_loss.ipynb`

Change:

* keep the architecture unchanged
* remove gradient matching from the training loss
* train with MSE only

Purpose:

* test whether gradient-aware supervision improves local structure and derivative consistency

---

## 5. What do the metrics mean?

### MSE

Mean squared error. Lower is better.

### RMSE

Square root of MSE. Lower is better and easier to interpret in the original scale.

### MAE

Mean absolute error. Lower is better.

### R2

Coefficient of determination. Closer to 1 is better.

### Relative L2

Relative L2 error. Lower is better.

### Gradient MSE

Measures gradient-related error. Lower is better. This is important because basis functions are spatial fields, so local variation matters in addition to pointwise values.

### Max absolute error

Largest pointwise error in the test set. Lower is better.

---

## 6. Full-mode final results for basis 04

### Full model

* MSE: `0.012038`
* RMSE: `0.109716`
* MAE: `0.037017`
* R2: `0.931886`
* Relative L2: `0.260202`
* Gradient MSE: `0.041623`

### No FNO

* MSE: `0.009339`
* RMSE: `0.096638`
* MAE: `0.039342`
* R2: `0.947134`
* Relative L2: `0.228946`
* Gradient MSE: `0.031159`

### No Attention

* MSE: `0.013008`
* RMSE: `0.114052`
* MAE: `0.039226`
* R2: `0.926424`
* Relative L2: `0.272993`
* Gradient MSE: `0.046128`

### No Gradient Loss

* MSE: `0.012894`
* RMSE: `0.113552`
* MAE: `0.038463`
* R2: `0.927099`
* Relative L2: `0.269931`
* Gradient MSE: `0.045090`

---

## 7. What do these results mean?

### Observation 1: the no-FNO variant performed best on this basis-04 task

This was the most surprising result.

The no-FNO ablation achieved the best:

* MSE
* RMSE
* R2
* Relative L2
* Gradient MSE

This means that, for basis function 04 under this training setup, the FNO-style preprocessing block did not improve the final result. In fact, removing it improved performance.

### Observation 2: removing attention made performance worse

Compared with the full model, the no-attention variant had:

* higher MSE
* higher RMSE
* lower R2
* higher Relative L2
* higher Gradient MSE

This suggests that the attention-gated skip connections were helpful in selecting useful encoder features for the decoder.

### Observation 3: removing gradient loss also made performance worse

Compared with the full model, the no-gradient-loss variant also had worse overall performance.

This suggests that gradient matching was useful, not only for gradient-related behavior, but also for overall prediction quality in this task.

---

## 8. What did I learn from this task?

### Controlled comparison matters

Ablation study means changing only one factor at a time while keeping the rest fixed. This makes it possible to attribute performance changes to a specific design choice.

### Intuition is not always the same as empirical result

I might expect every added module to help, but the no-FNO result shows that a theoretically attractive component does not always improve actual performance.

### Different modules help in different ways

From this task:

* attention gates appear to help feature selection
* gradient loss helps preserve local structure
* FNO-style preprocessing may or may not help depending on the target basis function and training setup

### Quick mode and full mode serve different purposes

Quick mode is for debugging and verification.
Full mode is for actual evaluation and interpretation.

---

## 9. Final takeaway

For basis function 04, the baseline full model performed well, but the no-FNO ablation performed even better. Removing attention gates or removing gradient matching loss reduced performance. Therefore, in this experiment:

* attention gates were useful
* gradient matching loss was useful
* the FNO-style preprocessing block was not beneficial for this specific basis function under the current setup

This task helped me understand both the workflow of a controlled ablation study and the importance of interpreting results based on real metrics rather than architectural intuition alone.
