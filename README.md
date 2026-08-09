# AD vs MCI Classifier — ADNI Dataset

This project uses deep learning to classify brain MRI scans as either **Alzheimer's Disease (AD)** or **Mild Cognitive Impairment (MCI)** — an early stage that may progress to AD. Early and accurate detection of the transition between these two stages has significant clinical value.

We fine-tuned a ResNet-18 convolutional neural network on 2D MRI slices from the [Alzheimer's Disease Neuroimaging Initiative (ADNI)](http://adni.loni.usc.edu/) dataset, achieving 75% validation accuracy and 71.4% test accuracy on a held-out set of 28 subjects.

---

## What is this project?

MRI scans produce hundreds of image slices per patient. This model looks at those slices and predicts whether the patient shows signs of Alzheimer's Disease or Mild Cognitive Impairment. Rather than diagnosing from a single slice, the model scores every slice for a given patient and uses a majority vote across all of them to make a final subject-level prediction.

---

## Dataset

This project uses the **ADNI (Alzheimer's Disease Neuroimaging Initiative)** dataset, which is not publicly available for direct download. To access it you must apply at [adni.loni.usc.edu](http://adni.loni.usc.edu/).

Once approved, the dataset used here consists of:
- **183 subjects** total (AD and MCI diagnoses, 24-month visit scans)
- **~35,000 2D DICOM slices** derived from 3D MRI volumes
- Split at the **subject level**: 70% train / 15% validation / 15% test

> ⚠️ Data files are not included in this repository due to ADNI's data sharing restrictions.

---

## Project Structure

---

## Model

- **Architecture:** ResNet-18 pretrained on ImageNet, with the final layer replaced by a 2-class head
- **Training strategy:** Freeze-then-unfreeze — backbone frozen for 3 epochs to stabilise the new head, then full fine-tuning
- **Regularisation:** Dropout 0.6, early stopping on validation loss, weighted cross-entropy loss
- **Data handling:** Central fraction filter (70%) to remove blank skull slices, weighted random sampler to correct class imbalance (31% AD / 69% MCI), 20 slices sampled per volume per epoch
- **Augmentation:** Horizontal flip, random rotation, colour jitter

---

## Results

| Model | Validation Accuracy | Test Accuracy |
|---|---|---|
| Baseline | 57.14% | — |
| + Weighted sampler | 72.7% | — |
| + Central fraction filter | 71.2% | — |
| + Freeze-unfreeze + Dropout | **75.0%** | **71.4%** |

**Test set performance (28 subjects, majority vote):**

| Class | Precision | Recall | F1 |
|---|---|---|---|
| AD (n=8) | 0.67 | 0.25 | 0.36 |
| MCI (n=20) | 0.76 | 0.95 | 0.84 |

ROC-AUC: 0.43

MCI classification was strong, while AD recall remained low — reflecting both the small number of AD test subjects and the inherent difficulty of separating early-stage AD from MCI on structural MRI.

---

## How to Run

### Requirements

- Python 3.8+
- A Google account (the notebooks are designed to run in Google Colab)
- Access to the ADNI dataset

### Steps

1. **Clone this repository**
```bash
   git clone https://github.com/yourusername/adni-classifier.git
```

2. **Open in Google Colab**
   Upload the notebooks to [colab.research.google.com](https://colab.research.google.com) or open them directly from your Google Drive.

3. **Add your data**
   Place your ADNI DICOM files in the expected directory structure and update the file paths in the notebook to match your Google Drive setup.

4. **Run the dataset creation notebook first**
   `Copy_of_dataset_creation.ipynb` processes the raw DICOM files and generates the CSV split files needed for training.

5. **Run the classifier notebook**
   `resnet18_adni_classifier.ipynb` handles training, validation, and test evaluation. A GPU runtime is strongly recommended — in Colab, go to **Runtime → Change runtime type → T4 GPU**.

---

## Limitations

- The test set contains only 8 AD subjects, making AD metrics sensitive to individual predictions
- The 2D slice approach treats each slice independently, missing volumetric spatial relationships
- A 3D CNN was designed to address this but could not be fully evaluated due to hardware constraints

---

## Acknowledgements

Data used in preparation of this project were obtained from the Alzheimer's Disease Neuroimaging Initiative (ADNI) database. The ADNI is funded by the National Institute on Aging and the National Institute of Biomedical Imaging and Bioengineering.
