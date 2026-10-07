# HRM Melt-Curve Classification of Bacterial Species

**Random Forest (scikit-learn, 200 trees) classifying 10 bacterial species from 18,281 digital HRM melt curves (Langouche et al., 2020), evaluated on a stratified random 80/20 split of individual curves: accuracy 99.6%, macro F1 0.995. Each species' curves come from a single chip, so species and chip are confounded; the score measures discrimination within this dataset, not generalisation to new chips, instruments or datasets.**

## At a glance
| | |
|---|---|
| Task | Identify 10 bacterial species (10 classes) from melt curves |
| Model | Random Forest, 200 trees (baselines: majority class, 1-NN Euclidean) |
| Data | 18,281 curves x 160 points (0.1 C steps), Langouche et al., *Bioinformatics* 2020 |
| Split | Stratified random 80/20 at the curve level (14,624 train / 3,657 test, `random_state=42`) |
| Extra checks | Duplicate-curve check, 5-fold stratified CV, baselines, shuffled-label control, top-5 feature selection, train/test loss curves (notebook sections 6 and 10 to 12) |

## Results (held-out test set, 3,657 curves)
| Metric | Score |
|---|---|
| Accuracy | 0.9962 |
| Macro F1 | 0.9952 |
| Weighted F1 | 0.9962 |

## Cross-validation and baselines (5-fold stratified, mean +/- sd)
| Model | Accuracy | Macro F1 |
|---|---|---|
| Majority class (dummy) | 0.1239 +/- 0.0000 | 0.0220 +/- 0.0000 |
| 1-NN (Euclidean) | 0.9969 +/- 0.0006 | 0.9958 +/- 0.0007 |
| Random Forest (200 trees) | 0.9963 +/- 0.0008 | 0.9954 +/- 0.0013 |

The score is stable across folds and matches the single split above. A plain 1-nearest-neighbour classifier performs just as well as the Random Forest, so the classes are almost perfectly separable by simple curve distance and the Random Forest adds no measurable advantage. This agrees with the original paper, where all four tested classifiers exceed 99.5%.

For context, the original paper reports accuracy above 99.5% for four classifiers on this dataset over five train/test splits, so this score is in line with the published benchmark.

## Shuffled-label control
Training the same Random Forest on shuffled species labels gives 0.1148 test accuracy, close to the 0.1077 expected by chance. The pipeline therefore does not leak label information.

## Top-5 feature selection
Features are the signal at single temperatures. Selection used the training set only: rank by ANOVA F-score, then skip any temperature with |r| >= 0.80 against one already chosen. A plain top 5 would give five adjacent temperatures (87.3 to 87.7 C), so the filter was needed.

| | |
|---|---|
| Selected temperatures | 87.0, 87.5, 87.9, 89.2, 89.6 C (all inside the melt-peak region) |

| Random Forest (200 trees) | Test accuracy | Test log loss | Train log loss |
|---|---|---|---|
| All 160 features | 0.9962 | 0.0468 | 0.0148 |
| Top 5 features | 0.9377 | 0.2173 | 0.0423 |

Five single temperatures are enough for about 94% accuracy (chance is about 11%), but the remaining accuracy needs the finer shape of the full curve. Training accuracy is 1.0 in both cases, which is normal for a Random Forest and is not by itself evidence of a problem; the held-out numbers above are the meaningful ones.

## Loss curves (5 features, log loss)
- **Random Forest:** test loss drops steeply over the first 20 trees and is almost flat by 100 to 200 trees (about 0.22 at 200), so more trees would not help. Training loss is flat from about 5 trees. The Random Forest curves never rise as trees are added.
- **Gradient boosting (HistGradientBoosting, 300 iterations, default settings):** training loss falls to about 0.0001, while test loss bottoms out at 0.1844 around iteration 38 and then climbs to 0.3726 at iteration 300. This is the typical overfitting pattern in loss (the model becomes overconfident on the curves it gets wrong). Test accuracy stays at 0.9382, about the same as the 5-feature Random Forest (0.9377), so the 5 temperatures cap accuracy near 94% for both models. The iteration with the lowest test loss was found on the test set, so it is descriptive only and would need a separate validation split (early stopping) to be used for tuning.

## Limitations (please read before citing the number)
- **Species and chip are confounded.** According to the dataset paper, the curves for each species come from a single chip. A random curve-level split puts curves from the same chip in both train and test, so the model may partly use chip-specific noise or temperature shifts instead of species-specific melt shape. A leave-one-chip-out split is not possible because each chip is one class.
- **No external validation.** Performance on new chips, instruments, dyes, primers or labs has not been tested and may be lower.
- **Not classic overfitting, but not proof of generalisation.** The test curves were never used for training and exact duplicates were ruled out, but a held-out score from the same runs does not show how the model behaves on a new dataset.
- Summary-table values (peak temperatures, amplicon lengths, DTW) are copied from Table 1 of the paper, not computed here.

## Pipeline
1. Load `0.csv` ... `9.csv` (one file per species; each row is one smoothed melt curve with 160 points)
2. Class overview and representative melt curves
3. Peak melting temperature summary
4. Duplicate-curve sanity check
5. Stratified 80/20 train/test split
6. Random Forest (200 trees)
7. Evaluation: accuracy, macro/weighted F1, per-class report, confusion matrix
8. Robustness checks: 5-fold CV, baselines, shuffled-label control
9. Correlation matrix and top-5 feature selection (training set only), then retraining with train/test log-loss curves (Random Forest and gradient boosting)

## Species
0 Citrobacter koseri, 1 Escherichia coli, 2 Enterococcus faecium, 3 Streptococcus group B (GBS), 4 Haemophilus influenzae, 5 Listeria monocytogenes, 6 Staphylococcus aureus (MSSA), 7 Streptococcus gallolyticus, 8 Streptococcus sanguinis, 9 Streptococcus pneumoniae

## Run it
```bash
pip install -r requirements.txt
# download 0.csv ... 9.csv from the Data folder of the authors' repository (link below)
# and place them in the same folder as the notebook
jupyter notebook hrm_bacterial_classification.ipynb
```

## Data
Dataset not included (about 55 MB; see `.gitignore`). The melt curves come from the supporting data of:

> Langouche L, Aralar A, Sinha M, Lawrence SM, Fraley SI, Coleman TP. *Data-driven noise modeling of digital DNA melting analysis enables prediction of sequence discriminating power.* Bioinformatics 36(22-23), 2020, 5337-5343. https://doi.org/10.1093/bioinformatics/btaa1053

Authors' repository: https://github.com/lenlan/dHRM-noise-modeling

This repository is an independent re-analysis and is not affiliated with the original authors.

## License
GPL-3.0, matching the license of the original data repository. See `LICENSE`.
