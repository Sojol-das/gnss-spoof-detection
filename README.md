![Python](https://img.shields.io/badge/Python-3.9-blue)
![Machine Learning](https://img.shields.io/badge/Machine%20Learning-Scikit--Learn-orange)
![Task](https://img.shields.io/badge/Task-GNSS%20Spoof%20Detection-green)
# GNSS Spoofing Detection using Machine Learning

## 1. Problem Statement

Global Navigation Satellite Systems (GNSS) are critical for navigation, aviation, logistics, telecommunications, and financial systems.

However, GNSS signals are vulnerable to spoofing attacks, where malicious transmitters broadcast fake signals that cause receivers to compute incorrect position or timing.

The objective of this project is to design an **AI-based spoof detection
system** capable of identifying anomalies in GNSS signal measurements and
detecting spoofing attempts.

    
## 2. Dataset Description

The dataset contains GNSS signal measurements collected from multiple satellite channels at each time step.

### Key Features

| Feature | Description |
|--------|-------------|
| PRN | Satellite ID |
| Carrier_Doppler_hz | Frequency shift caused by relative motion |
| Pseudorange_m | Distance between receiver and satellite |
| Carrier_phase | Signal phase measurement |
| EC / LC / PC | Early / Late / Prompt correlator outputs |
| CN0 | Carrier-to-noise ratio (signal strength) |
| TCD | Code delay |
| PIP / PQP | Signal quality indicators |

Spoofing attacks often introduce inconsistencies in these measurements,
including abnormal Doppler shifts, distorted correlation peaks,
or unrealistic signal strength patterns.


### Target Variable

This is a **binary classification problem**:

- **0 → Genuine GNSS signal**
- **1 → Spoofed GNSS signal**

Evaluation metric used in this competition:

**Weighted F1 Score**

---

## 3. Data Preprocessing

The following preprocessing steps were applied:

- Converted channel identifiers (`ch0`, `ch1`, etc.) into numeric format
- Converted signal columns to numeric values
- Removed rows containing missing values
- Removed potential leakage features such as `RX_time` and `TOW`
- Ensured consistent feature format for training and testing data


## 4. Feature Engineering

To capture temporal signal behavior and multi-satellite consistency, several
engineered features were introduced.

### Time Aggregated Features

Mean values across satellites for each time step:

- CN0_mean_time
- doppler_mean_time
- pseudo_mean_time

Standard deviation features capturing signal variability:

- CN0_std_time
- doppler_std_time
- pseudo_std_time

### Correlator Difference Feature

corr_diff = EC − LC

This captures distortion in the correlation peak, which may occur during
spoofing attempts.

These features help detect anomalies in signal strength, Doppler behavior,
and signal geometry across satellites.

### 5. Important Engineered Features

The model relies heavily on aggregated satellite features computed at each time step:

- **pseudo_mean_time** → Average pseudorange across satellites
- **doppler_mean_time** → Average Doppler shift across satellites
- **CN0_mean_time** → Average signal strength across satellites
- **pseudo_std_time** → Variation in pseudorange measurements
- **doppler_std_time** → Variation in Doppler shift
- **CN0_std_time** → Variation in signal strength

These features help detect inconsistencies across satellites which are often
introduced during GNSS spoofing attacks.

    
## 6. Model Architecture

The final model used is:

HistGradientBoostingClassifier

This gradient boosting model is well suited for large tabular datasets and
captures non-linear relationships between GNSS signal features.


### Reason for choosing this model

- Effective for tabular datasets
- Captures non-linear relationships
- Handles large datasets efficiently
- Robust to feature scaling

### Model Parameters
max_iter = 300
learning_rate = 0.05
max_depth = 8


## 7. Training Methodology

The dataset contains imbalanced classes where genuine signals dominate.

To ensure proper evaluation:

- A stratified train-validation split was used.
- Balanced sample weights were applied during training.

This prevents the model from biasing heavily toward the majority class.

## Handling Class Imbalance

Spoofing events are rare compared to genuine GNSS signals.

To address this imbalance, balanced sample weights were computed using:

compute_sample_weight(class_weight="balanced")

This ensures spoof events contribute proportionally during model training.


## 8. Prediction Strategy

Predictions are generated for each satellite channel.

To obtain a final spoof decision for each time step:

1. Predict spoof probability per channel
2. Aggregate probabilities by time using mean probability
3. Apply a spoof detection threshold

Threshold used: 0.05

    
## Feature Importance

Permutation feature importance analysis shows that the most influential
features for spoof detection are:

- **pseudo_std_time**
- **pseudo_mean_time**
- **doppler_mean_time**
- **doppler_std_time**
- **CN0_mean_time**

These aggregated satellite features capture inconsistencies in pseudorange,
Doppler behavior, and signal strength across satellites, which are strong
indicators of GNSS spoofing.

## Results

Validation Performance:

Weighted F1 Score ≈ 0.97

This indicates strong performance in distinguishing spoofed and genuine GNSS signals.



## 9. Submission Format

The final output file follows the required format:

time,Spoofed,Confidence
111402,0,0.000001
111403,0,0.000001
111404,0,0.000001

Where:

- **time** → Time step identifier
- **Spoofed** → Predicted class (0 or 1)
- **Confidence** → Prediction confidence value


## 10. How to Reproduce the Project

### Install Dependencies
pip install pandas numpy scikit-learn

### Run Training

Execute the training pipeline inside the notebook:
final_pipeline.ipynb

### Generate Predictions

Run the prediction pipeline to create the submission file:
submission_final.csv


## 11. Repository Structure

gnss-spoof-detection
│
├── notebooks
│ ├── exploration.ipynb
│ └── final_pipeline.ipynb
│
├── submission_final.csv
├── requirements.txt
└── README.md
gnss-spoof-detection
│
├── notebooks
│ ├── exploration.ipynb
│ └── final_pipeline.ipynb
│
├── submission_final.csv
├── requirements.txt
└── README.md


## 12. Notes

The dataset is not included in this repository and should be downloaded from the official competition dataset link.


## 13. Future Improvements

Possible improvements for future work include:

- Sequence-based models such as LSTM or Transformers for temporal modeling
- Satellite geometry consistency checks
- Unsupervised anomaly detection approaches for spoof detection


## 14. Project Workflow

The machine learning pipeline used in this project follows the workflow below:

Raw GNSS Dataset  
⬇  
Data Preprocessing  
(Channel encoding, numeric conversion, missing value handling)  

⬇  
Feature Engineering  
(corr_diff, CN0_mean_time, doppler_mean_time, pseudo_mean_time,  
CN0_std_time, doppler_std_time, pseudo_std_time)

⬇  
Train–Validation Split  
(Stratified sampling)

⬇  
Model Training  
(HistGradientBoostingClassifier with balanced sample weights)

⬇  
Validation  
(Weighted F1 Score evaluation)

⬇  
Test Prediction  
(Probability prediction per satellite channel)

⬇  
Aggregation by Time  
(Mean probability across channels)

⬇  
Spoof Detection  
(Threshold = 0.05)

⬇  
Submission File  
(time, Spoofed, Confidence)
