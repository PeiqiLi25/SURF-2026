# Task 2: Quantitative Spectral Analysis of Multiscale Basis Function Models

## Project Context

In this task, you will analyze trained AI models for predicting 12 multiscale basis functions in a Darcy flow problem. We assume that the following pretrained model files are already available:

- Full model: 12 trained models, one for each multiscale basis function.
- Ablation 1: 12 trained models.
- Ablation 2: 12 trained models.
- Ablation 3: 12 trained models.

Your goal is not only to compare prediction errors in physical space, but also to explain model performance from the frequency-domain perspective. In particular, you should investigate whether the full model is better than the ablation models, and identify which spectral evidence supports that conclusion.

This is a two-week group task for 5 undergraduate summer research students.

## Main Research Question

How can spectral analysis quantitatively explain why the full model, or one specific ablation model, performs better when predicting the 12 multiscale basis functions?

You should answer this question using numerical indicators, plots, and clear interpretation.

## Expected Inputs

You will be given:

- Test data for permeability fields and true multiscale basis functions.
- Full-model checkpoints for basis functions 1 to 12.
- Ablation-model checkpoints for basis functions 1 to 12 for each ablation experiment.
- Existing model definitions and data-loading code.

Before starting the analysis, confirm that every model produces predictions with shape:

```python
(batch_size, 1, 128, 128)
```

and that the corresponding target basis field has the same shape.

## Required Spectral Analysis

For each model type and each basis function, compute the following quantities.

## Mathematical Method

This section defines the exact mathematical quantities you should implement. Use the same formulas for the full model and all ablation models.

Let $u(x_i,y_j)$ be the true basis function on a $128 \times 128$ grid, and let $\hat{u}(x_i,y_j)$ be the model prediction. Here $i,j=0,\ldots,127$.

### Discrete 2D Fourier Transform

For each field $u$, compute its 2D discrete Fourier transform:

$$
\mathcal{F}[u](k_x,k_y)
=
\sum_{i=0}^{N_x-1}
\sum_{j=0}^{N_y-1}
u(x_i,y_j)
\exp\left[
-2\pi \mathrm{i}
\left(
\frac{k_x i}{N_x}
+
\frac{k_y j}{N_y}
\right)
\right].
$$

Here $N_x=N_y=128$. In Python, compute this using:

```python
u_fft = np.fft.fftshift(np.fft.fft2(u, norm="ortho"))
```

Use `fftshift` so that the zero-frequency mode is moved to the center of the spectrum.

The Fourier amplitude is:

$$
A_u(k_x,k_y)=\left|\mathcal{F}[u](k_x,k_y)\right|.
$$

The Fourier power is:

$$
P_u(k_x,k_y)=\left|\mathcal{F}[u](k_x,k_y)\right|^2.
$$

For visualization, use the log-amplitude:

$$
\log_{10}\left(A_u(k_x,k_y)+\varepsilon\right),
$$

where $\varepsilon$ is a small number such as $10^{-12}$ to avoid taking the logarithm of zero.

### Frequency Radius

For each Fourier mode $(k_x,k_y)$, define the radial wavenumber:

$$
k=\sqrt{k_x^2+k_y^2}.
$$

In Python, the frequency grid can be constructed by:

```python
ky = np.fft.fftshift(np.fft.fftfreq(128) * 128)
kx = np.fft.fftshift(np.fft.fftfreq(128) * 128)
grid_y, grid_x = np.meshgrid(ky, kx, indexing="ij")
k_radius = np.sqrt(grid_x**2 + grid_y**2)
```

The value $k$ tells us whether a Fourier component is low frequency, middle frequency, or high frequency.

### 1. Physical-Space Metrics

These metrics are used as a baseline comparison:

- Mean squared error, MSE.
- R2 score.
- Gradient MSE, if gradient loss is used in the model evaluation.

Use the same metric definitions for all models.

For one test sample, define:

$$
\mathrm{MSE}
=
\frac{1}{N_xN_y}
\sum_{i=0}^{N_x-1}
\sum_{j=0}^{N_y-1}
\left(\hat{u}_{ij}-u_{ij}\right)^2.
$$

The $R^2$ score is:

$$
R^2
=
1
-
\frac{\sum_{i,j}\left(u_{ij}-\hat{u}_{ij}\right)^2}
{\sum_{i,j}\left(u_{ij}-\bar{u}\right)^2+\varepsilon},
$$

where

$$
\bar{u}
=
\frac{1}{N_xN_y}
\sum_{i,j}u_{ij}.
$$

The gradient MSE can be defined by finite differences:

$$
\mathrm{GradMSE}
=
\mathrm{MSE}\left(\partial_x\hat{u},\partial_x u\right)
+
\mathrm{MSE}\left(\partial_y\hat{u},\partial_y u\right),
$$

where

$$
\partial_x u_{ij}=u_{i+1,j}-u_{i,j},
\qquad
\partial_y u_{ij}=u_{i,j+1}-u_{i,j}.
$$

### 2. Two-Dimensional Fourier Spectrum

For each predicted and true basis field:

1. Apply 2D FFT to the physical field.
2. Shift the zero frequency to the center.
3. Compute amplitude and power:

```python
amplitude = abs(fft2(field))
power = amplitude ** 2
```

You should visualize:

- True basis field.
- Predicted basis field.
- Absolute error field.
- Log-amplitude spectrum of the true field.
- Log-amplitude spectrum of the prediction.
- Log-amplitude spectrum of the error.

### 3. Radially Averaged Power Spectrum

Compute the radial power spectrum $E(k)$, where $k$ is the radial wavenumber.

Let $B_m$ be the set of Fourier modes whose radial wavenumber lies in the $m$-th bin:

$$
B_m
=
\left\{
(k_x,k_y):
k_m \le \sqrt{k_x^2+k_y^2} < k_{m+1}
\right\}.
$$

The radially averaged power spectrum is:

$$
E_u(k_m)
=
\frac{1}{|B_m|}
\sum_{(k_x,k_y)\in B_m}
P_u(k_x,k_y),
$$

where

$$
P_u(k_x,k_y)
=
\left|\mathcal{F}[u](k_x,k_y)\right|^2.
$$

For each model and basis function, compare:

- $E_{\mathrm{true}}(k)$
- $E_{\mathrm{pred}}(k)$

The full model should be considered better in spectral space if its predicted spectrum is closer to the true spectrum across low, middle, and high frequencies.

Implementation hint:

```python
power = np.abs(np.fft.fftshift(np.fft.fft2(u, norm="ortho"))) ** 2
for each radial bin B_m:
    E[m] = power[B_m].mean()
```

### 4. Relative Spectral Error by Wavenumber

For each radial frequency bin, compute:

$$
\mathrm{RSE}(k_m)
=
\frac{
\sum_{(k_x,k_y)\in B_m}
\left|
\mathcal{F}[\hat{u}](k_x,k_y)
-
\mathcal{F}[u](k_x,k_y)
\right|^2
}{
\sum_{(k_x,k_y)\in B_m}
\left|
\mathcal{F}[u](k_x,k_y)
\right|^2
+
\varepsilon
}.
$$

This indicator tells you where the model makes frequency-domain errors.

Interpretation:

- Smaller $\mathrm{RSE}(k_m)$ means better prediction in that frequency range.
- Large low-frequency error means the model fails to capture the global structure.
- Large high-frequency error means the model loses fine-scale details or creates noisy artifacts.

Analyze:

- Does the model mainly fail at low frequencies?
- Does the model mainly lose high-frequency details?
- Does one ablation model over-smooth the prediction?
- Does one model produce artificial high-frequency noise?

### 5. Bandwise Spectral Error

Group frequencies into several bands, for example:

- Low frequency: $0 \le k < 4$
- Lower-middle frequency: $4 \le k < 8$
- Middle frequency: $8 \le k < 16$
- High frequency: $16 \le k < 32$
- Very high frequency: $32 \le k \le 64$

For each band, compute:

- True spectral energy.
- Predicted spectral energy.
- Error spectral energy.
- Relative spectral error.
- Predicted/true energy ratio.

For a frequency band $S$, define the true energy:

$$
E_{\mathrm{true}}(S)
=
\sum_{(k_x,k_y)\in S}
\left|
\mathcal{F}[u](k_x,k_y)
\right|^2.
$$

The predicted energy is:

$$
E_{\mathrm{pred}}(S)
=
\sum_{(k_x,k_y)\in S}
\left|
\mathcal{F}[\hat{u}](k_x,k_y)
\right|^2.
$$

The error energy is:

$$
E_{\mathrm{err}}(S)
=
\sum_{(k_x,k_y)\in S}
\left|
\mathcal{F}[\hat{u}](k_x,k_y)
-
\mathcal{F}[u](k_x,k_y)
\right|^2.
$$

The bandwise relative spectral error is:

$$
\mathrm{BandRSE}(S)
=
\frac{E_{\mathrm{err}}(S)}
{E_{\mathrm{true}}(S)+\varepsilon}.
$$

The predicted/true energy ratio is:

$$
\rho(S)
=
\frac{E_{\mathrm{pred}}(S)}
{E_{\mathrm{true}}(S)+\varepsilon}.
$$

The predicted/true energy ratio is especially important:

- Ratio near 1: the model preserves the energy in that frequency band.
- Ratio below 1: the model underestimates that frequency band, possibly over-smoothing.
- Ratio above 1: the model overestimates that frequency band, possibly adding noise or artifacts.

### 6. Cross-Basis Spectral Consistency

For each model, compare whether it preserves the spectral behavior across all 12 basis functions.

Suggested indicators:

- Correlation between $\log E_{\mathrm{true}}(k)$ and $\log E_{\mathrm{pred}}(k)$ for each basis.
- Mean absolute error of log power:

For basis function $b$, define:

$$
C_b
=
\mathrm{Corr}
\left(
\log_{10}\left(E_{\mathrm{true},b}(k)+\varepsilon\right),
\log_{10}\left(E_{\mathrm{pred},b}(k)+\varepsilon\right)
\right).
$$

The log-power mean absolute error is:

$$
\mathrm{LMAE}_b
=
\frac{1}{M}
\sum_{m=1}^{M}
\left|
\log_{10}\left(E_{\mathrm{pred},b}(k_m)+\varepsilon\right)
-
\log_{10}\left(E_{\mathrm{true},b}(k_m)+\varepsilon\right)
\right|.
$$

where $M$ is the number of radial frequency bins.

A better model should have:

- Higher spectral correlation.
- Lower log-power error.
- More stable behavior across all 12 basis functions.

You should report the mean and standard deviation across all 12 basis functions:

$$
\overline{C}
=
\frac{1}{12}
\sum_{b=1}^{12} C_b.
$$

$$
\overline{\mathrm{LMAE}}
=
\frac{1}{12}
\sum_{b=1}^{12}
\mathrm{LMAE}_b.
$$

## Practical Implementation Steps

For every model type, basis function, and test sample:

1. Load the model checkpoint.
2. Generate prediction $\hat{u}$.
3. Store $\hat{u}$ and the true field $u$.
4. Compute physical-space metrics.
5. Compute $\mathcal{F}[u]$ and $\mathcal{F}[\hat{u}]$.
6. Compute 2D log-amplitude spectra.
7. Compute radial power spectra.
8. Compute relative spectral error by wavenumber.
9. Compute bandwise spectral energy and energy ratios.
10. Average the indicators over test samples.
11. Average or summarize the indicators over the 12 basis functions.

Use the same test set for all models. Otherwise, the comparison is not fair.

## Visualization Requirements

The group should include several types of visualizations. The purpose of the figures is not only to make the report look complete, but to help explain where and why one model is better than another.

### 1. Physical-Space Prediction Comparison

For selected test samples and selected basis functions, plot:

- Input permeability field.
- True basis function.
- Full-model prediction.
- Ablation-model predictions.
- Absolute error maps for each model.

This figure answers:

- Does the model capture the main spatial structure?
- Does the prediction miss local details?
- Does any ablation model produce visibly larger local errors?

Suggested format:

- One row for the true field and model predictions.
- One row for absolute error maps.
- Use the same color scale for all basis-function fields in the same figure.
- Use the same color scale for all error maps in the same figure.

### 2. 2D Fourier Log-Amplitude Spectrum

For the same selected samples, plot:

- Log-amplitude spectrum of the true basis function.
- Log-amplitude spectrum of the full-model prediction.
- Log-amplitude spectrum of each ablation-model prediction.
- Log-amplitude spectrum of the prediction error.

Use:

$$
\log_{10}\left(\left|\mathcal{F}[u](k_x,k_y)\right|+\varepsilon\right)
$$

and similarly for the prediction and error.

This figure answers:

- Does the model preserve the correct frequency structure?
- Does the ablation model lose high-frequency components?
- Does the model introduce artificial high-frequency artifacts?

### 3. Radial Power Spectrum Curves

For each basis function, plot the radially averaged power spectrum:

- True spectrum, $E_{\mathrm{true}}(k)$.
- Full-model predicted spectrum, $E_{\mathrm{full}}(k)$.
- Ablation-model predicted spectra, $E_{\mathrm{ablation}}(k)$.

Use a log-log plot when possible.

This figure answers:

- Which model follows the true spectrum most closely?
- Does a model overestimate or underestimate spectral energy?
- Are the differences mainly in low, middle, or high frequencies?

### 4. Relative Spectral Error Curves

For each basis function, plot:

- $\mathrm{RSE}(k)$ for the full model.
- $\mathrm{RSE}(k)$ for each ablation model.

Use a logarithmic y-axis if the error range is large.

This figure answers:

- Which model has the lowest spectral error?
- At which wavenumbers does each ablation model fail?
- Is the full model consistently better across the spectrum, or only in certain frequency ranges?

### 5. Bandwise Spectral Error Bar Plot

For the selected frequency bands, plot grouped bars for:

- Low frequency.
- Lower-middle frequency.
- Middle frequency.
- High frequency.
- Very high frequency.

For each band, compare the full model and all ablation models using:

- Bandwise relative spectral error, $\mathrm{BandRSE}(S)$.
- Predicted/true energy ratio, $\rho(S)$.

This figure answers:

- Which frequency band contributes most to the model difference?
- Does a model over-smooth the prediction by losing high-frequency energy?
- Does a model add noise by overestimating high-frequency energy?

### 6. Cross-Basis Summary Plots

Across all 12 basis functions, plot:

- Mean physical-space metrics by model.
- Mean relative spectral error by model.
- Mean spectral correlation $\overline{C}$ by model.
- Mean log-power error $\overline{\mathrm{LMAE}}$ by model.

Recommended plot types:

- Bar charts for model-level averages.
- Line plots across basis index 1 to 12.
- Heatmaps with model type on one axis and basis index on the other axis.

This figure answers:

- Is the full model consistently better across all basis functions?
- Are some basis functions especially difficult?
- Which ablation causes the largest degradation?

### Minimum Figure Set for the Final Report

The final report should include at least:

- One physical-space prediction comparison figure.
- One 2D Fourier spectrum comparison figure.
- One radial power spectrum comparison figure.
- One relative spectral error curve figure.
- One bandwise spectral error or energy-ratio figure.
- One cross-basis summary figure.

All figures should have clear titles, legends, axis labels, colorbars where needed, and consistent model names.

## Suggested Group Work Packages

This is a 5-person group task. The group only needs a broad division of work; members may collaborate across packages as needed. The following packages should all be completed within the two-week period.

### Package A: Data Loading and Prediction Generation

Main tasks:

- Load the test data and all pretrained model checkpoints.
- Confirm that all models are evaluated on the same test samples.
- Generate predictions for the full model and the three ablation models for all 12 basis functions.
- Save predictions and ground truth fields in a consistent format, such as `.npz`.
- Record checkpoint paths, basis indices, prediction shapes, and target shapes.

Expected outputs:

- Saved prediction files.
- A checkpoint-and-shape verification table.
- A short note describing the data-loading and prediction procedure.

### Package B: Physical-Space Evaluation

Main tasks:

- Compute MSE, R2 score, and gradient MSE for each model and each basis function.
- Produce comparison tables and summary statistics.
- Check whether the physical-space results agree with the spectral-analysis conclusions.

Expected outputs:

- `physical_metrics_by_basis.csv`
- `physical_metrics_summary.csv`
- One figure comparing physical-space performance across the 12 basis functions.

### Package C: Fourier Spectrum Visualization

Main tasks:

- Generate physical-space prediction plots and error maps.
- Generate 2D Fourier log-amplitude spectrum plots.
- Select representative samples that clearly show differences between the full model and ablation models.
- Explain what the visual spectral differences mean.

Expected outputs:

- Representative prediction/error figures.
- Representative 2D spectrum-comparison figures.
- Short written interpretation of the figures.

### Package D: Quantitative Spectral Metrics

Main tasks:

- Compute radially averaged power spectra.
- Compute relative spectral error by wavenumber.
- Compute bandwise true energy, predicted energy, error energy, bandwise spectral error, and predicted/true energy ratio.
- Compute cross-basis spectral consistency indicators.
- Identify which frequency bands explain the performance differences most clearly.

Expected outputs:

- `radial_power_spectra.csv`
- `relative_spectral_error_by_k.csv`
- `bandwise_spectral_error.csv`
- Cross-basis spectral consistency tables.
- Figures comparing spectral error curves and bandwise energy ratios.

### Package E: Interpretation, Report, and Presentation

Main tasks:

- Combine physical-space and spectral-space results.
- Explain why the full model, or a specific ablation model, performs better.
- Connect the spectral findings with the model architecture.
- Prepare the final report and presentation.

Expected outputs:

- Final report.
- Final slides.
- A concise summary table of key conclusions.

## Time Frame

The expected duration of this task is two weeks.

The group should first complete data loading and prediction generation, then compute physical-space and spectral-space metrics, and finally prepare the interpretation, report, and presentation. The exact internal schedule can be decided by the group.

## Reference Materials

Students may search for reliable reference materials on Fourier analysis, spectral error metrics, neural operators, and multiscale finite element methods. Any external materials used in the final report should be properly cited.

## Final Deliverables

The group should submit:

- Clean analysis code or notebook.
- All generated CSV tables.
- All important figures.
- A final written report.
- A 10-minute presentation.

The final report should include:

1. Research objective.
2. Data and model description.
3. Physical-space metric comparison.
4. Spectral-analysis method.
5. Radial spectrum results.
6. Bandwise spectral error results.
7. Cross-basis consistency results.
8. Explanation of why one model is better.
9. Limitations and possible future work.

## Evaluation Criteria

Your work will be evaluated based on:

- Correct implementation of spectral analysis.
- Fair comparison across all models.
- Quality of quantitative indicators.
- Clarity of plots.
- Depth of interpretation.
- Reproducibility of code.
- Team coordination and presentation quality.

## Key Questions to Answer

At the end of the task, your group should be able to answer:

- Which model has the best overall physical-space accuracy?
- Which model has the lowest spectral error?
- Does the full model better preserve low-frequency structure?
- Does the full model better preserve high-frequency details?
- Which ablation causes the largest spectral degradation?
- Is the main advantage of the full model visible in low, middle, or high frequencies?
- Are some basis functions harder to predict spectrally than others?
- Can spectral metrics explain performance differences that are not obvious from MSE alone?

## Suggested Conclusion Format

Use evidence-based statements such as:

> The full model achieves the lowest relative spectral error in the middle- and high-frequency bands, while Ablation 1 underestimates high-frequency energy. This suggests that the removed module is important for preserving fine-scale basis-function structure.

or:

> Although two models have similar MSE, their spectra are different. The ablation model has a lower predicted/true energy ratio at high frequencies, indicating over-smoothing. Therefore, spectral analysis reveals a difference that is not fully captured by physical-space error metrics.




