# Magic Wand — Number Recognition Model Evaluation

---

## 1. Baseline Model

The baseline model was trained on a large, pre-existing gesture dataset consisting of 8,811 samples across 10 digit classes (0–9). The dataset was split into 80% training, 10% validation, and 10% test. The model architecture is a small but effective Convolutional Neural Network (CNN) built with Keras, designed to classify 32×32 grayscale rasterized stroke images.

After training, the model was progressively compressed for microcontroller deployment:

| Model                     | Size           | Reduction              |
|---------------------------|----------------|------------------------|
| TensorFlow                | 171,722 bytes  | —                      |
| TensorFlow Lite           | 47,412 bytes   | reduced by 124,310 bytes |
| TensorFlow Lite Quantized | 15,928 bytes   | reduced by 31,484 bytes  |

### Performance

Evaluated on the held-out test set, the baseline model achieved high accuracy. In physical wand-drawing tests on the Arduino board, the model reliably recognized digits **0, 1, 2, 3, 6, 7, and 9**. These digits share a common trait: they are either single-stroke or follow widely consistent writing conventions across different people, making them easier for the model to generalize. Digits **4, 5, and 8**, which involve multi-stroke patterns or exhibit higher variability in how individuals write them, were less consistently recognized.

---

## 2. Fine-Tuned Model

The fine-tuned model was adapted from the baseline using a hand-collected dataset of approximately 100 personal gesture samples, gathered using a custom data collection tool. Rather than retraining from scratch, the baseline model's weights were used as a starting point — all layers were frozen except the last convolutional layer and all dense layers, which were unfrozen and retrained for 10 epochs using the Adam optimizer at a low learning rate (1e-3). This technique, known as fine-tuning, allows the model to adapt to personal writing style while preserving the general features learned from the large dataset.

### Performance

| Metric            | Value  |
|-------------------|--------|
| Overall Accuracy  | 38.38% |
| Macro F1-score    | 0.3586 |
| Weighted F1-score | 0.3627 |

**Per-class breakdown:**

| Digit | Precision | Recall | F1-Score | Support |
|-------|-----------|--------|----------|---------|
| 0     | 0.00      | 0.00   | 0.00     | 11      |
| 1     | 1.00      | 0.30   | 0.46     | 10      |
| 2     | 0.27      | 0.55   | 0.36     | 11      |
| 3     | 0.25      | 0.50   | 0.33     | 10      |
| 4     | 0.00      | 0.00   | 0.00     | 8       |
| 5     | 0.28      | 0.50   | 0.36     | 10      |
| 6     | 0.57      | 0.80   | 0.67     | 10      |
| 7     | 0.83      | 0.50   | 0.63     | 10      |
| 8     | 0.50      | 0.22   | 0.31     | 9       |
| 9     | 0.57      | 0.40   | 0.47     | 10      |

The low overall accuracy of 38% on the quantitative evaluation is expected given the small fine-tuning dataset (~10 samples per class). Despite this, the fine-tuned model showed improved responsiveness to the tester's personal gesture style during physical testing on the board, suggesting the adaptation had a meaningful practical effect even if not fully reflected in the metrics.

The model retained similar stroke-recognition challenges as the baseline — digits 4 and 0 remained difficult, with near-zero precision and recall. This is consistent with the confusion matrix, which shows digit 0 being confused broadly across multiple classes, and digit 4 being misclassified primarily as 2 and 3.

An important factor in recognition accuracy — for both models — is **stroke direction**. The IMU captures motion as a time-series, so the same digit drawn in different directions (e.g., drawing a "1" top-down versus bottom-up) produces a different sensor signature. Since the training data reflects a fixed set of stroke conventions, gestures that deviate from those conventions — even slightly — can result in misclassification. This sensitivity to directionality is a fundamental constraint of gesture-based recognition without explicit direction normalization, and likely accounts for much of the variability seen between digits and between users.

---

## Summary Comparison

| Aspect                  | Baseline Model                          | Fine-Tuned Model                          |
|-------------------------|-----------------------------------------|-------------------------------------------|
| Training Data           | Large pre-existing dataset (8,811 samples) | ~100 hand-collected personal samples    |
| Architecture            | Small CNN (Keras)                       | Same architecture, partial layer unfreeze |
| Quantized Size          | 15,928 bytes                            | Similar (int8 quantized)                  |
| Test Accuracy           | High (large dataset)                    | 38.38% (limited fine-tune data)           |
| Physical Test — Easy Digits | 0, 1, 2, 3, 6, 7, 9               | Similar, with improved personal gesture fit |
| Physical Test — Hard Digits | 4, 5, 8                            | 4, 0 remain difficult                     |
| Stroke Direction Sensitivity | Present                          | Present                                   |
| Best Use Case           | General digit recognition               | Personalized gesture recognition          |
