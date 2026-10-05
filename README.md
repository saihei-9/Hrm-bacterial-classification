# HRM Melt-Curve Classification of Bacterial Species

Random Forest classifier that identifies 10 bacterial species from high-resolution melt (HRM) curves.

## Pipeline
1. Load `0.csv` ... `9.csv` (one file per species; each row is a normalized melt curve with 160 temperature points)
2. Class overview and representative melt curves
3. Peak melting temperature summary by species
4. Duplicate-curve sanity check
5. Stratified 80/20 train/test split
6. Random Forest (200 trees)
7. Evaluation: accuracy, macro/weighted F1, per-class report, confusion matrix

## Results (held-out test set, 3,657 curves)
| Metric | Score |
|---|---|
| Accuracy | 0.9962 |
| Macro F1 | 0.9952 |
| Weighted F1 | 0.9962 |

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

## Notes
- Results come from a single random stratified split (random_state=42), without cross-validation.
- Peak temperatures, amplicon lengths and DTW values in the summary table are hard-coded, not computed in the notebook.

## License
GPL-3.0, matching the license of the original data repository. See `LICENSE`.
