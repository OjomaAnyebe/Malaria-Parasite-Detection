# Malaria Blood-Cell Image Classifier

A Jupyter notebook project for auditing labeled blood-cell microscopy images, preparing reproducible train/validation/test splits, and training an experimental binary image classifier for **Uninfected** and **Parasitized** cell images.

> **Research and education only.** This project is not a medical device, diagnostic tool, or patient-level malaria-risk assessment. Do not use its outputs for diagnosis or treatment decisions. A qualified health professional and appropriate laboratory testing are required.

## Current status

The dataset audit and image split have been completed. **The classifier has not yet been trained or evaluated.** There is no trained model checkpoint, test accuracy, ROC-AUC, or prediction report in the current project artifacts. The notebook contains cells to train and evaluate the model once its dependencies are installed and the notebook is run.

## Dataset audit

The notebook scans the `data_set` and `cell_images` folders and derives a label only when an image is inside a folder named `Parasitized` or `Uninfected`. It checks that images can be opened, hashes file contents to remove byte-identical copies, and stops if an identical image appears under conflicting class labels.

Verified audit results for the current folders:

| Audit result | Count |
|---|---:|
| Image files discovered | 49,328 |
| Exact duplicate files removed | 21,770 |
| Unique labeled images retained | 27,558 |
| Uninfected | 13,779 |
| Parasitized | 13,779 |
| Unlabeled or unreadable images excluded | 0 |
| Identical images with conflicting labels | 0 |

These are **folder-derived labels**; they were not independently reviewed or clinically verified.

### Train / validation / test split

The split is stratified and group-aware. Files with the same filename-derived capture group are kept together, and the notebook asserts that group IDs and image hashes do not overlap across splits.

| Split | Images | Uninfected | Parasitized | Groups |
|---|---:|---:|---:|---:|
| Train | 19,795 | 9,912 | 9,883 | 1,003 |
| Validation | 3,929 | 1,984 | 1,945 | 206 |
| Test | 3,834 | 1,883 | 1,951 | 199 |

## Run the notebook

1. Open [`malaria_training.ipynb`](malaria_training.ipynb) in Jupyter or VS Code.
2. Select a Python kernel. Python 3.10+ is recommended.
3. Run the dependency cell if the kernel does not already have the required packages:

   ```text
   torch torchvision scikit-learn matplotlib pandas pillow
   ```

4. Run all cells in order. The first model setup may download pretrained ResNet-18 weights, requiring internet access.
5. After training and test evaluation complete, run the **Manually check random held-out test images** section to view random predictions. Set `RANDOM_TEST_SEED` to an integer to reproduce the same sample.

The notebook uses CUDA if available and otherwise uses CPU. Training on CPU may take considerably longer.

## Method

- Labels come exclusively from the `Parasitized` and `Uninfected` folder names.
- SHA-256 hashes remove exact duplicate image files before splitting.
- A fixed random seed and `StratifiedGroupKFold` create the group-aware splits.
- Images are resized to 224 × 224; training uses mild augmentation.
- The planned classifier fine-tunes ImageNet-pretrained ResNet-18.
- Validation loss drives checkpoint selection, learning-rate reduction, and early stopping.
- The test set is reserved for final evaluation and is not used to select training epochs.

## Outputs

The notebook writes generated outputs under `artifacts/malaria_classifier/`:

- `train_manifest.csv`, `validation_manifest.csv`, `test_manifest.csv`
- `dataset_split/` — copied image files arranged by split and class
- `class_counts.png`, `sample_images.png`
- After training: `best_resnet18_malaria.pt`, `training_history.csv`, and `training_curves.png`
- After evaluation: `test_predictions.csv`, `classification_report.csv`, `test_metrics.png`
- After random test sampling: `random_test_predictions.csv` and `random_test_predictions.png`
- `summary.json` after successful evaluation

The project also includes:

- [`Malaria_Project_Brief.pdf`](Malaria_Project_Brief.pdf) — project overview, verified audit/split counts, a class-balance chart, and current evaluation status
- [`Malaria_Code_Line_by_Line_Explanation.pdf`](Malaria_Code_Line_by_Line_Explanation.pdf) — explanations for notebook code

The local `sample_images.png` preview contains dataset images and is excluded from this public repository because redistribution permission has not been established. The class-count chart contains counts only, not cell photographs.

## Dataset source and redistribution

The local folder structure and filename pattern strongly resemble the NIH/NLM malaria-cell image collection described in the [original research paper](https://pmc.ncbi.nlm.nih.gov/articles/PMC5907772/) and on the [archived NLM/LHNCBC dataset page](https://web.archive.org/web/20181106123201/https://ceb.nlm.nih.gov/repositories/malaria-datasets/). This is a likely provenance match, not proof that every local file is the same release.

The local files contain no dataset-specific license, and the sources above do not establish explicit permission to mirror the image archive in this repository. Therefore, **the image binaries and image-containing sample preview are not included in this repository by default**. Use the original source link to obtain the dataset and follow its terms. Add the dataset files only after confirming the applicable redistribution permission and attribution requirements.

The notebook expects the downloaded data to be organized under `data_set/` and/or `cell_images/` as described above; these local data folders are excluded from Git by `.gitignore`.

## Licensing

No license is supplied for the dataset images, and this repository does not grant rights to the images. The project code and documentation also have no license file yet; absent a license, standard copyright applies. Do not assume that the notebook, PDFs, or dataset are covered by a license that is not present in the repository.
