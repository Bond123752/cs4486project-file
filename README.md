# Mechanisms of Action prediction

CS4486 mini project using the [Kaggle Mechanisms of Action competition](https://www.kaggle.com/competitions/lish-moa). The [executed notebook](MoA_Kaggle_Submission.ipynb) predicts a probability for each of 206 mechanisms per sample from 872 gene-expression and cell-viability measurements plus treatment time and dose.

## Reproduce the experiment

**On Kaggle.** Import `MoA_Kaggle_Submission.ipynb` into a Kaggle notebook, attach the competition dataset, and run all cells. The last cell writes `/kaggle/working/submission.csv`.

**On the group's Windows computer.** The notebook detects the official files at `C:/cs4486/lish-moa/` automatically.

**On another computer.** Place the extracted files in a `data/` folder next to the notebook. The five required files are `train_features.csv`, `train_targets_scored.csv`, `train_drug.csv`, `test_features.csv`, and `sample_submission.csv`. For data stored elsewhere, set the environment variable `MOA_DATA_DIR` to the extracted folder before starting Jupyter. Get the files from the [competition data page](https://www.kaggle.com/competitions/lish-moa/data).

Install the dependencies in a Python 3.11 environment, then launch from the folder containing the notebook.

```sh
python -m pip install notebook numpy pandas "scikit-learn>=1.6" torch
python -m notebook MoA_Kaggle_Submission.ipynb
```

Select **Run All Cells**. The notebook records the installed package versions in its first output. It uses CUDA when available. The local output directory is `moa_submission/`. Set `MOA_OUTPUT_DIR` to choose another directory.

## Reading the experiment

Each of five shuffled folds (seed 42) holds out entire treated drugs. The models learn on treated samples only. Vehicle controls receive zero probabilities and are spread across the held-out folds. Preprocessing and label frequencies are fit separately on each training fold. The notebook aggregates log loss over sample–target pairs and also reports the treated-only loss.

The supplied network has hidden widths of 256 and 128, 25% dropout and a separate sigmoid probability for each target. We retain the original epoch counts of 4, 2, 10, 4 and 3. We evaluate the smoothed frequency baseline, the original network and a fixed 90% network / 10% frequency blend on identical folds. This blend uses label frequencies from the training fold.

## Results on held-out drugs

| Method | All-sample log loss | Treated-only log loss |
|---|---:|---:|
| Frequency baseline | 0.022193 | 0.024080 |
| Original MLP | 0.020452 | 0.022191 |
| 90% MLP + 10% training frequency | **0.020094** | **0.021802** |

The fixed blend reduced all-sample log loss by **1.75%** relative to the original MLP. It helped on four of the five folds. Fold 3 accounted for most of the gain (0.024242 to 0.022431), while fold 5 worsened slightly (0.022800 to 0.022924). The treated-only score also improved. These results apply to this five-fold split. An official Kaggle score or a ranking under a shared course split requires separate evaluation.

The saved submission uses the frequency blend. To produce the original ensemble's predictions, set `SUBMISSION_METHOD = "mlp"` before running the notebook. The recorded run used Python 3.11.0, NumPy 2.3.2, pandas 2.3.2, scikit-learn 1.9.1 and PyTorch 2.15.0 on CPU (four threads).

## Files produced by a run

| File | Contents |
|---|---|
| `submission.csv` | The five fold models' average test probabilities, aligned to the Kaggle sample submission |
| `cv_summary.csv` | Pooled held-out log loss for all samples and for treated samples |
| `fold_scores.csv` | Per-fold losses, sample counts and training epochs |
| `fold_assignments.csv` | Each training sample's held-out fold, identified by `sig_id` |

## AI assistance in this revision

OpenAI Codex reviewed the supplied submission notebook, added held-out scoring and a frequency-blend comparison, made the data path portable, checked the exported predictions, and prepared this README. The neural architecture, fitting procedure, epoch schedule, grouped split and control handling came from the supplied work. Group members should describe their individual contributions and any earlier assistance in the course report.
