 **Context Dataset Creation**

**Version:** 1.0.0
**Layer:** Dataset Preparation Layer

---

## **Purpose**

This module converts extracted contexts into structured, labelled data suitable for machine learning.
Each context is written as a single **JSONL** entry for scalable training.

---

## **Input**

| Field          | Type  | Description                                     |
| -------------- | ----- | ----------------------------------------------- |
| `context_list` | Array | All extracted contexts from Step 1              |
| `label_rules`  | JSON  | Rule-based labels used for training supervision |

---

## **Output**

**JSONL dataset**

```
{"contextType":"attribute","flags":{...},"dynamic":{...},"attackClass":"event-handler","payloadType":"js-exec"}
```

---

## **Workflow**

1. Receive list of extracted contexts
2. Add rule-based labels for `attackClass` and `payloadType`
3. Clean fields (normalize booleans, text, missing values)
4. Append each entry to `dataset.jsonl`
5. Validate schema before saving

---

## **Dependencies**

| Module          | Role                           |
| --------------- | ------------------------------ |
| `bs4` output    | Source of static data          |
| Playwright logs | Source of dynamic features     |
| Rule Engine     | Provides initial attack labels |
| JSONL Writer    | Stores dataset entries         |

---

## **Exceptions**

| Error               | Trigger                           |
| ------------------- | --------------------------------- |
| `SchemaMismatch`    | Missing required fields           |
| `InvalidLabelError` | Rule engine fails to assign class |
| `WriteFailure`      | Disk write or permission issue    |

---

---