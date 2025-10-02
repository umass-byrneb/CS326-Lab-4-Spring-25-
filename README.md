# Implementing *Deep Learning with Differential Privacy* on an Audio Dataset

This repository contains a PyTorch implementation of the methods described in the paper **“Deep Learning with Differential Privacy”** by Abadi et al. The goal of this project is to apply these state-of-the-art techniques to train a deep neural network on a sensitive **audio** dataset while providing **rigorous, mathematical privacy guarantees**.

Modern deep learning models can memorize and potentially expose fine details of their training data. This implementation offers protection against a strong adversary with full knowledge of the model’s architecture and parameters—appropriate for on-device deployment scenarios. We successfully trained effective audio classification models under a **modest (“single‑digit”) privacy budget**.

---

## Design & Implementation

The core training algorithm is **Differentially Private Stochastic Gradient Descent (DP‑SGD)** (Algorithm 1 in the source paper). Privacy is integrated directly into the training loop by modifying gradient updates at each step:

1. **Per‑example gradient computation** – For each example in a lot, compute its gradient. This enables the next steps and exposes each example’s influence.
2. **Per‑example gradient clipping** – Clip each gradient’s ℓ2 norm to the threshold **C** *before* averaging. This bounds any single example’s influence on the update.
3. **Noise addition** – After clipping and averaging within the lot, add calibrated Gaussian noise. The amount is scaled by the noise multiplier **σ** and the clipping bound **C**. This obscures precise contributions.
4. **Privacy accounting** – Track cumulative privacy loss **ε** across many steps using the **Moments Accountant** (via **Opacus**), which yields much tighter bounds than standard composition.

---

## Settings

**Dataset**: Custom audio dataset for keyword spotting, preprocessed into **MFCC** features.
- Training examples: **240**
- Test examples: **81**
- Number of classes: **4**

**Model**: Feed‑forward neural network with one hidden layer and ReLU; input dimension first reduced using a **Differentially Private PCA** layer to improve utility and performance.

**Initial Sweep Hyperparameters**:
- Batch size (lot): **16**
- Epochs: **20**
- Clipping norm **C**: **[0.5, 1.0]**
- Noise multiplier **σ**: **[0.5, 1.0, 2.0]**
- Learning rate: **0.1**
- Random seed: fixed for reproducibility

---

## Results

Non‑private baseline accuracy for this architecture: **79.01%**.

### Initial Parameter Sweep

A small grid search to inspect the privacy–utility trade‑off.

> **Table 1.** Results from the initial small‑grid hyperparameter sweep (δ = 1/240).

| C   | σ   | Accuracy (%) | ε    |
|-----|-----|--------------|------|
| 0.5 | 0.5 | 56.17        | 18.64 |
| 0.5 | 1.0 | 49.38        | 2.93  |
| 0.5 | 2.0 | 46.91        | 0.95  |
| 1.0 | 0.5 | 62.35        | 18.64 |
| 1.0 | 1.0 | 61.73        | 2.93  |
| 1.0 | 2.0 | 60.49        | 0.95  |

**Insights (consistent with the paper):**
- **Effect of noise (σ):** Increasing σ consistently and dramatically improves privacy (lowers ε). With **C = 1.0**, raising **σ** from **0.5 → 2.0** reduces **ε** from **18.64 → 0.95**.
- **Trade‑off:** Highest accuracy (**62.35%**) coincides with the worst privacy (**ε = 18.64**). A highly promising configuration is **C = 1.0, σ = 2.0**, yielding **60.49%** with **ε = 0.95**—meeting the *single‑digit* privacy target.

### Systematic Hyperparameter Tuning

Guided by the paper’s note that lot size impacts accuracy, we conducted a focused sweep.

> **Table 2.** Full results from the improved hyperparameter sweep (see `results/improved_sweep.csv`).

**Top configurations discovered:**
- **High‑utility model:** `[lot=32, clip=4.04, noise=1.0]` → **69.75%** accuracy, **ε = 4.43**.
- **Excellent trade‑off:** `[lot=32, clip=2.02, noise=2.0]` → **62.35%** accuracy, **ε = 1.43**.

### Regularization Effect of DP

Consistent with the paper, **DP‑SGD acts as a powerful regularizer**, preventing overfitting. Unlike non‑private training, DP models show a small train–test gap.
- *Figure 1.* The non‑private model (red) exhibits a widening train–test gap; the DP model (blue) keeps them close, indicating better generalization.

(Place your figure at `results/plots/train_vs_test_dp_vs_non_dp.png`.)

---

## Reproducing the Experiments

### Environment

```bash
python -m venv .venv && source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

**requirements.txt (example)**
```
torch
opacus
numpy
pandas
matplotlib
librosa
```

### Training (initial sweep)

```bash
python scripts/train.py   --dataset data/   --epochs 20   --lot 16   --clip 0.5 1.0   --noise 0.5 1.0 2.0   --lr 0.1   --seed 0   --out results/runs_initial.csv
```

### Training (focused sweep)

```bash
python scripts/train.py   --dataset data/   --epochs 20   --lot 32   --clip 2.02 4.04   --noise 1.0 2.0   --lr 0.1   --seed 0   --out results/improved_sweep.csv
```

### Plotting Accuracy vs ε

This repository’s plotting utility renders **accuracy vs ε** and labels each point as **`[lot,clip,noise]`** (numbers only; no letter prefixes). The x‑axis uses **integer** ε tick labels (display‑only rounding).

```bash
python scripts/plot_eps_acc.py   --csv results/improved_sweep.csv   --out results/plots/epsilon_vs_accuracy.png
```

> **Plot label rules**
> - Point label format: `[lot,clip,noise]` (e.g., `[32,2,2]`), **no** `L`, `C`, or `δ` in labels.
> - ε values shown as **integers** on the axis (rounded for display only). Raw ε stays precise in CSV.

---

## Repository Layout (suggested)

```
.
├─ data/                         # audio files or prepared MFCCs
├─ dp_utils/
│  ├─ privacy.py                 # moments accountant / ε helpers (Opacus)
│  └─ transforms.py              # DP-PCA, MFCC preprocessing
├─ scripts/
│  ├─ train.py                   # DP-SGD training & logging
│  └─ plot_eps_acc.py            # accuracy vs ε plotting
├─ results/
│  ├─ runs_initial.csv
│  ├─ improved_sweep.csv
│  └─ plots/
│     ├─ epsilon_vs_accuracy.png
│     └─ train_vs_test_dp_vs_non_dp.png
├─ requirements.txt
└─ README.md
```

---

## Takeaways

- **Privacy–utility is manageable:** We achieved **69.75%** accuracy (within ~10 points of the **79.01%** baseline) with **ε = 4.43**—a strong single‑digit guarantee.
- **Strong privacy with solid utility:** We trained a **useful** model at **ε = 0.95** with **60.49%** accuracy.
- **Hyperparameter tuning is critical:** Lot size, clip norm, and noise multiplier have a *large* impact and must be carefully tuned.

---

## Notes

- **DP‑PCA** can improve downstream utility; ensure its parameters and randomness are logged.
- All ε values are computed with **Opacus**’ moments accountant under **δ = 1/240** for the initial table.

## License

Choose an open‑source license (e.g., MIT) and place it in `LICENSE`.

## Citation

If you use this code or results, please cite the original paper and your repository.
