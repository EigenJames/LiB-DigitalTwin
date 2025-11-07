# Lithium-Ion Battery Digital Twin: Cycling Aging Study

This repository disseminates a reproducible workflow for analysing lithium-ion battery (LiB) degradation using both physics-guided and machine-learning (ML) approaches. The artefacts combine NASA Prognostics Center of Excellence (PCoE) datasets with neural forecasting pipelines to prototype a digital twin for cycle-level capacity fade.

## Why Machine Learning Matters for LiB Research

LiB aging dynamics are governed by coupled electrochemical, thermal, and mechanical processes that are only partially observable through laboratory measurements. First-principles models often require extensive parameter identification and can struggle with cell-to-cell variability and mission-profile heterogeneity. ML-based surrogates complement physics by:

- capturing non-linear degradation signatures directly from telemetry;
- enabling transfer learning across chemistries and operating regimes;
- quantifying uncertainty for predictive maintenance policies;
- accelerating what-if simulations within a digital-twin control loop.

Hybrid workflows—where ML augments mechanistic priors—provide actionable forecasts for scheduling, warranty design, and safety diagnostics.

## Dataset: Experimental Protocol and Significance

### Data Provenance

The study leverages the **NASA PCoE Li-ion Battery Aging Dataset**, comprising controlled cycle-life experiments on four commercial 18650 cylindrical cells (model Sony US18650VTC, nominal 3.7 V, 2.6 Ah):

- **Cell IDs**: B0005, B0006, B0007, B0018
- **Total Cycles**: 616 galvanostatic discharge cycles
- **Sampling Frequency**: 6 Hz (≈167 ms resolution)
- **Access**: <https://data.nasa.gov/dataset/Li-ion-Battery-Aging-Datasets/uj5r-zjdb>.

### Experimental Protocol

Each cycle followed a standardised **charge-then-discharge** protocol:

1. **Charging Phase**:
   - Constant-current charging at 1.5 A from rest to 4.2 V (end-of-charge voltage).
   - Constant-voltage hold at 4.2 V until current dropped below 20 mA.
   - Typical charge time: 70–80 minutes.

2. **Resting Phase**:
   - 30-minute rest at open-circuit voltage to reach thermal equilibrium.

3. **Discharging Phase**:
   - Galvanostatic discharge at 1.5 A from 4.2 V to 2.7 V end-of-discharge voltage.
   - Typical discharge time: 90–110 minutes (depending on state of health).

4. **Measurements Recorded**:
   - Instantaneous voltage (V)
   - Instantaneous current (A)
   - Cell surface temperature (°C)
   - **Discharged capacity** (Ah) — the primary degradation indicator
   - Cycle number and timestamp

### Why This Experiment Is Important

**Relevance to Real-World Applications:**

- The 1.5 A discharge rate (≈0.58C for a 2.6 Ah cell) approximates consumer device duty cycles (e.g., power tools, electric vehicles at moderate discharge).
- The 4.2 V charging ceiling is industry-standard; systematic voltage stress is a primary aging accelerant.
- 600+ cycles represent ≈2–3 years of typical consumer usage, bridging laboratory timescales with practical warranties.

**Degradation Signature:**

- Capacity fade is the **most observable metric** of cell aging, directly impacting run-time and user experience.
- Capacity typically decays exponentially, dropping ≈2–5% per 100 cycles before potential thermal runaway or lithium plating at extreme degradation.
- The NASA dataset captures this fade trajectory at high resolution, enabling precise forecasting of end-of-life (typically defined as 80% state of health).

**Reproducibility and Accessibility:**

- Publicly available raw data eliminates proprietary barriers and allows reproducible benchmarking.
- Multiple cells (B0005–B0018) reveal cell-to-cell variability, which is critical for designing robust predictive algorithms.
- High-frequency telemetry (6 Hz) enables feature extraction of charging/discharging dynamics, temperature transients, and voltage hysteresis—all implicated in degradation mechanisms.

**Machine Learning Opportunity:**

- While physics-based models (e.g., Single Particle Model, empirical exponential decay) provide mechanistic insight, they struggle to capture:
  - Stochastic variability between nominally identical cells
  - Complex interactions between temperature, depth-of-discharge, and current rate
  - Subtle electrochemical side reactions (solid-electrolyte interface growth, lithium plating).
- ML surrogates trained on this dataset can infer latent degradation patterns and **transfer to unseen cells or chemistries**, accelerating battery technology adoption.

## Repository Map

- `discharge.csv` – Canonicalised discharge summary table derived from NASA raw MAT files.
- `01_Initial_Neural_Network_Test.ipynb` – Proof-of-concept neural network aligning simulated outputs with synthetic experimental targets.
- `02_Capacity_Prediction_Development.ipynb` – Development notebook integrating data wrangling, physical baselines, and empirical regressors.
- `03_Capacity_Prediction_Comprehensive_Analysis.ipynb` – Research-grade narrative with methodology, ablation studies, and diagnostic plots.
- `04_Physical_vs_Empirical_NN_Comparison.ipynb` – Comparative study contrasting mechanistic models with neural predictors.
- `tuner_results/` – Hyperparameter optimisation artefacts (Keras Tuner checkpoints and metadata) for reproducibility.

## Methodological Scope

This project implements and contrasts three distinct modeling paradigms for Li-ion battery degradation, with a focus on two primary prognostic tasks: **unseen cell generalization** and **time-series forecasting**.

### 1. Physics-Based Baseline Model

A first-principles model based on an exponential decay formulation provides an analytical baseline. Capacity C at cycle i is estimated as:

$$C(i) = C_0 \cdot e^{-k \cdot T_c \cdot i / t}$$

where C₀ is initial capacity, T_c is cell temperature, t is charge time, and k is an empirical degradation constant. This model encodes fundamental domain knowledge but is limited in its ability to capture cell-specific, non-linear aging dynamics.

### 2. Purely Data-Driven Neural Network (NN)

A standard feedforward neural network (Dense NN) is trained to directly map engineered features (cycle number, temperature, mean voltage/current) to the battery's capacity. This approach serves as a pure empirical benchmark to assess the predictive power of the data alone, without physical priors.

### 3. Unique ML Methodology: Residual-Enhanced Physical Model (Hybrid Approach)

This is the core data-driven technique of the project, representing a form of **physics-informed machine learning**. The workflow is as follows:

1. **Initial Prediction**: The physics-based model generates an initial capacity forecast. This prediction captures the primary exponential decay trend but contains systematic errors.
2. **Residual Learning**: A neural network is trained specifically to predict the **residual error** between the physical model's output and the ground-truth experimental data.
3. **Corrected Forecast**: The final, refined prediction is the sum of the physical model's output and the neural network's learned residual correction.

$$C_{\text{final}} = C_{\text{physical}} + \text{NN}(X)_{\text{residual}}$$

This hybrid model constrains the neural network, preventing it from learning non-physical behavior while leveraging its power to model complex, unmodeled degradation phenomena.

### Prognostic Tasks Investigated

The above models are evaluated against two critical battery management tasks:

- **Unseen Cell Generalization**: To simulate deployment on a new battery, we employ a **Leave-One-Cell-Out (LOCO)** cross-validation strategy. The models are trained on data from N-1 cells and tested on the held-out cell. This rigorously assesses the model's ability to generalize beyond its training distribution and handle manufacturing variability.
- **Time-Series Forecasting**: To simulate remaining useful life (RUL) prediction for an in-service battery, models are trained on the first *X%* of cycles for a given cell and evaluated on their ability to forecast the subsequent *(100-X)%* of cycles. This tests the model's capacity for long-term extrapolation.

### Hyperparameter Optimization

The neural network architectures are systematically optimized using **Keras Tuner** with a `RandomSearch` strategy. This ensures that model performance is not an artifact of arbitrary architectural choices and allows for reproducible, robust comparisons between the different modeling approaches.

## Getting Started

```bash
git clone https://github.com/EigenJames/lib-digital-twin-cycling-aging.git
cd lib-digital-twin-cycling-aging
python -m venv .venv
source .venv/bin/activate  # On macOS/Linux; use .venv\Scripts\activate on Windows
pip install -r requirements.txt  # forthcoming dependency export
```

Open any notebook in JupyterLab or VS Code with the Python kernel set to the created virtual environment. The repository includes large `.h5` checkpoint files; ensure sufficient storage (≈30 MB) when cloning.

## Project Status and Roadmap

- Baseline data ingestion and exploratory plots
- Initial physics vs ML comparative study
- Formal requirements export (`requirements.txt`, `environment.yml`)
- Probabilistic uncertainty quantification for cycle predictions
- Integration of on-device monitoring signals (voltage/current waveforms)

## Citation

If you build upon this work, please cite the NASA dataset and relevant degradation modelling literature listed below.

## License

This project is licensed under the **MIT License** – see the [LICENSE](LICENSE) file for details. The MIT License permits free use, modification, and distribution, provided attribution is retained.

## References

1. Bole, B., Kulkarni, C., Daigle, M. (2014). *Li-ion Battery Aging Datasets.* NASA Ames Prognostics Center of Excellence. <https://data.nasa.gov/dataset/Li-ion-Battery-Aging-Datasets/uj5r-zjdb>
2. Xu, B., Oudalov, A., Ulbig, A., Andersson, G., Kirschen, D. S. (2018). "Modeling of Lithium-Ion Battery Degradation for Cell Life Assessment." *IEEE Transactions on Smart Grid*, 9(2), 1138–1146. <https://doi.org/10.1109/TSG.2016.2578950>
3. Severson, K. A., Attia, P. M., Jin, N., Perkins, N., Jiang, B., Yang, Z., Herring, P. K., Aykol, M., Herring, P. K., Braatz, R. D., Ermon, S., Chueh, W. C. (2019). "Data-driven prediction of battery cycle life before capacity degradation." *Nature Energy*, 4, 383–391. <https://doi.org/10.1038/s41560-019-0347-4>
4. Attia, P. M., Grover, A., Jin, N., Severson, K. A., Markov, T. M., Liao, Y.-H., Chen, M. H., Cheong, B., Perkins, N., Yang, Z., Herring, P. K., Aykol, M., Braatz, R. D., Ermon, S., Chueh, W. C. (2020). "Closed-loop optimization of fast-charging protocols for batteries with machine learning." *Nature*, 578, 397–402. <https://doi.org/10.1038/s41586-020-1994-5>
