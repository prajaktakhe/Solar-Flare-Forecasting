Solar Flare Forecasting with Machine Learning

Overview

This project investigates machine-learning approaches for forecasting solar flares using active-region observations from the SDO/HMI SHARP dataset.

The analysis focuses on:

* Exploratory analysis of solar active-region parameters
* Handling severe class imbalance in flare observations
* Preventing information leakage using active-region-based data splitting
* Feature analysis and correlation with flare occurrence
* Decision-tree classification
* Comparison of class-weighting and random undersampling strategies

The project was conducted as part of Astronomy 101 by Equinox — The Astronomy Club of IIT Guwahati.

Year: 2026

⸻

Dataset

The dataset contains observations of solar active regions, including magnetic-field and active-region parameters used to investigate whether machine-learning models can distinguish flare-producing regions from non-flaring regions.

The main variables include:

* NOAA_AR — NOAA active-region identifier
* HARPNUM — HMI Active Region Patch identifier
* USFLUX — Total unsigned magnetic flux
* TOTUSJH — Total unsigned current helicity
* TOTUSJZ — Total unsigned vertical current
* MEANALP — Mean magnetic twist parameter
* R_VALUE — Magnetic flux near the polarity inversion line
* TOTPT — Total magnetic energy proxy
* SAVNCPP — Total unsigned vertical current per polarity
* AREA_ACR — Active-region area
* ABSNJZH — Absolute net current helicity

The original dataset was provided as part of the Astronomy 101 course and is not redistributed in this repository.

⸻

Data Exploration and Preprocessing

Initial exploration examined the dataset shape, data types, missing values, duplicate observations, target distribution, feature distributions, and relationships between the features and flare occurrence.

The dataset contained 200,842 observations and 13 columns.

No missing values were found.

The target variable was highly imbalanced:

Class	Count
N	123,840
padding	65,808
P	11,194

Padding observations

The dataset contained artificial padding observations. These rows had magnetic-field features equal to zero and were distinguishable from real observations.

These artificial padding rows were removed before model training.

The flare target was then converted into a binary variable:

* 0 — Non-flare (N)
* 1 — Flare (P)

After removing padding observations, the dataset contained 135,034 real observations.

⸻

Preventing Information Leakage

Solar active-region observations are repeated over time, meaning that randomly splitting individual observations could place measurements from the same active region into both the training and validation sets.

This would allow the model to effectively see the same active region during training and validation.

To avoid this, the data was split using GroupShuffleSplit, with HARPNUM used as the grouping variable.

This resulted in:

* Training observations: 95,470
* Validation observations: 39,564
* Active-region overlap between training and validation: 0

This provides a more realistic evaluation of how the model performs on previously unseen active regions.

⸻

Class Imbalance

The flare-producing class is much rarer than the non-flaring class. This creates a major challenge for classification.

A baseline decision-tree model achieved approximately 89.4% accuracy, but its recall for the flare class was only 3.1%.

This demonstrates why accuracy alone is not an appropriate metric for this problem: a model can achieve high accuracy while detecting very few actual flares.

Therefore, particular attention was given to:

* Flare-class recall
* Flare-class precision
* F1-score
* ROC-AUC
* PR-AUC

⸻

Imbalance Handling

Two approaches were investigated.

1. Class Weighting

A decision tree was trained using:

class_weight="balanced"

This increases the effective importance of the minority flare class during training without creating or removing observations.

Results:

Metric	Baseline	Class Weighting
Flare Recall	3.1%	37.1%
Flare Precision	15.2%	15.7%
Accuracy	89.4%	75.7%
ROC-AUC	0.661	0.594
PR-AUC	0.168	0.139

Class weighting substantially improved flare detection, increasing recall from 3.1% to 37.1%, while keeping precision approximately unchanged.

The reduction in overall accuracy is expected because the model becomes more willing to classify observations as potential flares rather than favoring the majority class.

⸻

2. Random Undersampling

Random undersampling was also investigated by reducing the number of majority-class training observations while leaving the validation set unchanged.

A 3:1 majority-to-minority ratio was used for the training data.

Results:

Metric	Baseline	Random Undersampling
Flare Recall	3.1%	27.1%
Flare Precision	15.2%	19.7%
Accuracy	89.4%	83.0%
ROC-AUC	0.661	0.592
PR-AUC	0.168	0.144

Undersampling improved both flare recall and precision compared with the baseline, although its recall improvement was smaller than that obtained through class weighting.

⸻

Key Findings

The analysis shows that:

1. Accuracy is misleading for this highly imbalanced classification problem.
2. Group-based splitting is important because multiple observations can originate from the same solar active region.
3. Removing artificial padding observations is necessary before model training.
4. Class weighting produced the largest improvement in flare recall, increasing it from 3.1% to 37.1%.
5. Random undersampling produced a better flare precision than class weighting, reaching 19.7%.
6. The results demonstrate a trade-off between detecting more potential flares and generating false alarms.
7. Magnetic-field-related features show strong correlations with one another, indicating substantial feature redundancy.

⸻

Models

The project currently evaluates Decision Tree classifiers with a fixed maximum depth of 8.

The following approaches were compared:

* Baseline decision tree
* Decision tree with balanced class weighting
* Decision tree with random undersampling

⸻

Limitations

Several limitations should be considered:

* The dataset is highly imbalanced.
* The analysis uses a relatively simple tree-based classifier.
* Class-weighting and undersampling involve different trade-offs between recall and precision.
* The available observations may contain temporal dependencies that are not fully captured by the current splitting strategy.
* The dataset provided for the course is not redistributed with this repository.

⸻

Repository Contents

solar-flare-forecasting/
├── README.md
└── solar_flare_forecasting.ipynb

The Jupyter notebook contains the complete analysis, including data exploration, preprocessing, feature analysis, model training, and evaluation.

⸻

Acknowledgements

This project was completed as part of Astronomy 101, conducted by Equinox — The Astronomy Club of IIT Guwahati.

The dataset used in this analysis was provided by the Astronomy 101 team.
