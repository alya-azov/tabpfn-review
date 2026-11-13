# TabPFN: Critical Review and Experiment

A critical review of the paper **"Accurate predictions on small data with a tabular foundation model"** (Hollmann et al., *Nature*, 2025) and a small experiment that tests TabPFN on data that does not look like its training data.

This was a research task for department selection at MIPT (autumn 2026).

## What is TabPFN

TabPFN is a transformer pre-trained on about 100 million synthetic tables. On a new table it does not train in the usual way: it takes the training rows as context and predicts the answers for new rows in one forward pass. The authors report that on datasets with up to 10,000 rows and 500 features it beats gradient boosting and tuned ensembles, often in seconds.

## Review: main points

- **The runtime comparison is incomplete.** TabPFN used a GPU while the other methods ran only on CPU, and the two weeks of pre-training are not counted. Prediction is also slower: about 0.2 s per row for TabPFN versus 0.0002 s for CatBoost, as the authors note themselves.
- **The test set is small and narrow:** 29 classification and 28 regression datasets, all with at most 10,000 rows.
- **The model is only as good as its synthetic prior.** Data with structure that the prior does not generate may be hard for it. The experiment below checks this.

## Experiment

**Data:** 800 RNA sequences of 24 nucleotides (`X0`–`X23`, values 0–3) from a machine learning course assignment. The target `y` is an efficiency index (regression). It mostly depends on codons, the triples of neighbouring nucleotides.

**Features:**
- raw nucleotides: 24 columns;
- codon counts: 64 columns, one for each codon.

**Protocol:** 80/20 train/test split, 5 random splits (seeds 0–4), the same rows for all models, metric RMSE.

| Model | Features | RMSE (mean ± std over 5 splits) |
|---|---|---|
| Predict the mean | — | 2.256 ± 0.104 |
| Linear regression | codon counts | **0.183 ± 0.018** |
| TabPFN | codon counts | 0.455 ± 0.050 |
| TabPFN | raw nucleotides | 2.135 ± 0.119 |

## Findings

- **On raw nucleotides TabPFN is barely better than predicting the mean** (it explains about 10% of the variance). It did not find the codon structure on its own, and marking the columns as categorical did not help.
- **Shuffling the column order did not change the result.** TabPFN does not use the order of features, so it cannot see that neighbouring columns form codons.
- **With codon features TabPFN works well, but a simple linear regression is still more accurate**, because the target is close to a sum of codon contributions.
- **Conclusion:** TabPFN is a strong default for typical tables, but feature engineering and knowledge of the domain still matter, as the authors themselves say in their user guide.

## Files

- `tabpfn_experiment.ipynb` — the experiment (Google Colab, GPU T4)
- `TabPFN_review_RU.pdf` — the full review (in Russian)
