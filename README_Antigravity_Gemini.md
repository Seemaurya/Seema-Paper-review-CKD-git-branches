# XHAF-CKD: Explainable Hierarchical Attention-Fuzzy Network for CKD Diagnosis

**Target Journal:** Expert Systems with Applications (Elsevier)

This repository contains the complete execution pipeline for the proposed XHAF-CKD architecture. The overarching goal of this research is to bridge the gap between high-capacity Deep Learning and clinical transparency in nephrology by introducing a novel, multi-source, interpretable neuro-fuzzy network.

---

## 1. The Core Innovation (XHAF-CKD Architecture)
Traditional baseline models (Random Forest, XGBoost, standard MLPs) often achieve perfect accuracy by operating as opaque "black boxes" that overfit to deterministic GFR threshold equations. XHAF-CKD deliberately avoids this by forcing data through an interpretable hierarchy:

1. **Layer A (Symptom Attention):** Dynamically weights the importance of 8 free-text demographic/clinical symptoms (e.g., Hypertension, Weakness, Edema) per patient.
2. **Layer B (Biomarker Attention):** Dynamically weights the importance of 7 continuous laboratory biomarkers (e.g., GFR, Serum Creatinine).
3. **Layer C (ANFIS Fuzzy Fusion):** A mathematically robust, log-space mean fusion layer that mimics Adaptive Neuro-Fuzzy Inference Systems (ANFIS). It logically combines the symptom and biomarker streams into 32 unified fuzzy rules via learnable Gaussian distributions.
4. **Layer D (MC Dropout Classifier):** Outputs the 5-class KDIGO CKD Stage prediction alongside **calibrated Uncertainty Bounds** using Monte Carlo stochastic forward passes, ensuring doctors know exactly when the AI is mathematically unsure.

---

## 2. Robust Training Strategy & Data Segregation
A massive flaw in existing CKD literature is the lack of real-world clinical validation. To prove true generalization for Elsevier, this project utilizes a strict 4-part data strategy:

* **Training & Validation Base (n=970):** The model is trained *exclusively* on publicly available data—the canonical UCI CKD dataset augmented perfectly by NHANES 2017-2018 population data.
* **SMOTE Balancing:** Because early-stage CKD is rare in public datasets, Synthetic Minority Over-sampling Technique (SMOTE) was applied *only to the training set* to ensure the neural network learned Stage 1/2 profiles equally alongside Stage 4/5.
* **Zero-Shot External Testing (Hospital Data):** A completely isolated, separate cohort of private real-world hospital patients was kept completely hidden from the model during training. The model achieved **86.6% zero-shot accuracy** on these unseen clinical patients, proving it does not memorize data but rather learns true biological CKD pathology.

---

## 3. Execution Pipeline

### `step1_data_pipeline.py`
Aggregates the disparate datasets (UCI, NHANES, Hospital). Unifies them via KDIGO GFR formulas into a strict 15-feature schema (7 biomarkers + 8 symptoms). Applies SMOTE scaling to the training set.

### `step2_eda.py`
Generates publication-ready figures proving the statistical significance of the chosen features (Pearson Correlation Matrices, Stage Distribution plots).

### `step3_model.py`
The core engine. Trains the XHAF-CKD network (500 epochs, Cosine Annealing learning rate, Label Smoothing). 
* **Baselines:** The script rigorously benchmarks XHAF-CKD against 6 baselines: Classical ML (Random Forest, SVM, XGBoost) and three recent literature proxy architectures (ABPNN-ANFIS, SACNN-SOA, MLFIS).
* **Metrics:** Generates precision, recall, F1, Cohen's Kappa, and confusion matrices across both internal Validation and external Hospital testing.

### `step4_shap_explainability.py`
Generates the explainability artifacts required for clinical trust:
* **Global SHAP Summary:** Proves which features drive macro-level network reasoning.
* **Local SHAP Waterfall:** Deconstructs exactly how a specific patient's vital signs pushed the model to its diagnosis.
* **Attention Heatmaps:** Visualizes the dynamic shifting of Layer A and Layer B attention weights across the external hospital patients.
* **Clinical Explanation Report:** A patient-by-patient table outputting exactly why the model made a decision (e.g., *Patient 15: Stage 1 | 97.0% Conf | 6.9% Uncert | Driving Features: GFR, Weakness, BP*).

### `step5_compile_paper.py`
An automated aggregation script designed to gather all CSV tables, confusion matrices, and PNG figures scattered throughout the pipeline and pack them cleanly into a single `Paper_Submission_Assets` directory for immediate journal upload.

---
*Developed for peer-reviewed submission in Clinical Artificial Intelligence.*
