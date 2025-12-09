# **Machine Learning Context Classifier**

**Version:** 1.0.0
**Layer:** ML Classification Layer

---

## **Purpose**

Train a machine learning model to classify:

* **attackClass** (event-handler, attribute-context, innerHTML, JS-URL, etc.)
* **payloadType** (js-exec, html-breakout, dom-manip, url-js, etc.)

The model learns from contextual features extracted in Steps 1 & 2.

---

## **Inputs**

| Field           | Type  | Description                    |
| --------------- | ----- | ------------------------------ |
| `dataset.jsonl` | JSONL | Full dataset used for training |

---

## **Outputs**

| File                    | Description                                     |
| ----------------------- | ----------------------------------------------- |
| `model_attackClass.pkl` | Random Forest model predicting attack type      |
| `model_payloadType.pkl` | Random Forest model predicting payload category |
| `encoders.pkl`          | Label encoders for categorical fields           |
| `model_features.pkl`    | Saved list of final ML feature columns          |

---

## **Workflow**

1. Load JSONL dataset into pandas dataframe
2. Preprocess → One-Hot Encoding via pandas
3. Split into train/test
4. Train **Random Forest Classifier** (best accuracy)
5. Evaluate with classification report & accuracy
6. Save final models with joblib

---

## **Dependencies**

| Library       | Purpose                               |
| ------------- | ------------------------------------- |
| Pandas        | Data preprocessing                    |
| Scikit-Learn  | RandomForest, train/test, evaluations |
| Joblib        | Model serialization                   |
| OneHotEncoder | Feature encoding                      |

---

## **Exceptions**

| Error                  | Trigger                           |
| ---------------------- | --------------------------------- |
| `FeatureMismatch`      | Missing columns at inference time |
| `ModelTrainingFailure` | Invalid data or missing labels    |

---

---