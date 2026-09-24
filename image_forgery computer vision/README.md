

# 🕵️ Image Forgery Detection using ELA + CNN

Detect whether an image is **authentic** or **tampered** using **Error Level Analysis (ELA)** and a **Convolutional Neural Network**, trained on the **CASIA v2.0** dataset. For images flagged as tampered, the project also highlights the suspected manipulated region.

![Python](https://img.shields.io/badge/Python-3.x-blue)
![TensorFlow](https://img.shields.io/badge/TensorFlow-Keras-orange)
![Accuracy](https://img.shields.io/badge/Test%20Accuracy-90.54%25-brightgreen)

---

## 📌 Overview

Image editing tools make it easy to splice, copy-move or remove objects in a photo. This project tackles that with a passive forensic approach:

1. Re-save the image as JPEG (quality 91) and compute the difference from the original → **ELA map**.
2. Feed the ELA map into a **CNN** that learns compression/noise inconsistencies.
3. Classify the image as `real` or `tampered`.
4. If tampered, threshold the ELA map to **highlight high-error regions**.

## 📊 Results

| Metric | Validation | Test |
|---|---|---|
| Accuracy | 90.47% | **90.54%** |
| Precision (authentic) | 96.16% | 95.03% |
| Recall (tampered) | 95.27% | 92.45% |
| Recall (authentic) | 86.95% | 89.36% |

Final training accuracy: **94.71%** (25 epochs). Test set = 1,248 images.

**Test confusion matrix** (0 = tampered, 1 = authentic): 441 tampered correctly caught, 36 tampered missed (predicted authentic), 82 authentic wrongly flagged as tampered, 689 authentic correctly identified.

<p align="center">
  <img src="assets/confusion_matrices.png" width="700" alt="Confusion matrices">
</p>

<p align="center">
  <img src="assets/training_curves.png" width="420" alt="Training curves">
</p>

> Validation loss starts rising after ~epoch 10 while training loss keeps falling, which is mild overfitting. See [Future Improvements](#-future-improvements).

## 🖼️ Visual Examples

**ELA at different JPEG quality levels**

<p align="center">
  <img src="assets/ela_quality_levels.png" width="650" alt="ELA at various quality levels">
</p>

**Authentic vs tampered image with their ELA maps**

<p align="center">
  <img src="assets/authentic_vs_tampered_ela.png" width="520" alt="Authentic vs tampered ELA">
</p>

**Suspected manipulated region (tampered prediction)**

<p align="center">
  <img src="assets/localization_output.png" width="320" alt="Localization output">
</p>

## 📂 Dataset

**CASIA v2.0** (Image Tampering Detection), two folders:

| Class | Label | Images |
|---|---|---|
| Authentic (`Au`) | 1 | 7,354 |
| Tampered (`Tp`) | 0 | 5,123 |
| **Total** | | **12,477** |

Split: **72% train (8,983) / 18% validation (2,246) / 10% test (1,248)**, `random_state=42`.

Dataset on Kaggle: search for "CASIA 2.0 image tampering detection dataset". The notebook expects it at `/kaggle/input/casia-dataset/CASIA2/`; change `Config.au` and `Config.tp` if your path differs.

## 🧠 Model Architecture

| Layer | Output Shape | Params |
|---|---|---|
| Conv2D (32, 5×5, ReLU) | 124×124×32 | 2,432 |
| Conv2D (32, 5×5, ReLU) | 120×120×32 | 25,632 |
| MaxPooling2D (2×2) | 60×60×32 | 0 |
| Dropout (0.25) | 60×60×32 | 0 |
| Flatten | 115,200 | 0 |
| Dense (256, ReLU) | 256 | 29,491,456 |
| Dropout (0.5) | 256 | 0 |
| Dense (2, Softmax) | 2 | 514 |

**Total parameters:** 29,520,034

**Training:** Adam (lr = 1e-4 with decay), binary cross-entropy, batch size 32, 25 epochs, input 128×128×3 ELA images.

## 🛠️ Tech Stack

- Python, TensorFlow / Keras
- Pillow, OpenCV (ELA computation)
- NumPy, scikit-learn, Matplotlib

## 🚀 Getting Started

### 1. Clone the repo
```bash
git clone https://github.com/veerarathi/image-forgery-detection.git
cd image-forgery-detection
```

### 2. Install dependencies
```bash
pip install tensorflow numpy matplotlib scikit-learn pillow opencv-python jupyter
```

> The notebook uses the legacy import `keras.utils.np_utils`. On newer Keras versions, replace it with `from keras.utils import to_categorical`.

### 3. Set up the dataset
Download CASIA v2.0 and update the paths in the `Config` class:
```python
class Config:
    au = "path/to/CASIA2/Au"
    tp = "path/to/CASIA2/Tp"
```

### 4. Run the notebook
```bash
jupyter notebook notebook.ipynb
```
Or upload it to Kaggle/Colab with the dataset attached and run all cells.

## 🔍 Usage (Predicting on a New Image)

```python
image = prepare_image("path/to/image.jpg")   # ELA + resize to 128x128 + normalise
image = image.reshape(-1, 128, 128, 3)
pred = model.predict(image)
class_names = ['tampered', 'real']
print(class_names[pred.argmax()], f"{pred.max()*100:.2f}%")
```

The trained model is saved as `model_casia_run1.h5`.

## 🗂️ Project Structure

```
image-forgery-detection/
├── notebook.ipynb        # Full pipeline: ELA, training, evaluation, localisation
├── model_casia_run1.h5   # Saved model (generated after training)
├── assets/               # Figures used in this README
└── README.md
```

## ⚠️ Limitations

- ELA relies on JPEG compression; heavily re-compressed, resized or lossless images can hide traces.
- Images are resized to 128×128, which may lose small tampered regions.
- Mild overfitting after ~epoch 10 (no early stopping used in the final run).
- Tested only on CASIA v2.0; not evaluated on AI-generated or other-source forgeries.
- Region highlighting is a simple threshold heuristic, not a trained segmentation model.

## 🔮 Future Improvements

- Early stopping / checkpointing on validation loss
- Transfer learning (Xception, ResNet) with higher resolution
- Data augmentation and class weighting
- Combine ELA with noise residuals and metadata cues
- Trained segmentation model for accurate tamper masks
- Deploy as a web app (Streamlit / Flask)

## 📚 References

- J. Dong, W. Wang, T. Tan, *CASIA Image Tampering Detection Evaluation Database*, 2013.
- N. Krawetz, *A Picture's Worth: Digital Image Analysis and Forensics*, Black Hat, 2007.

## 👤 Author

**Veera Rathi**
Reg No: 24BAI10699
GitHub: [@veerarathi](https://github.com/veerarathi)

---

⭐ If you found this project useful, consider giving it a star!