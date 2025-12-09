
# **C-Script Construction System**

**Version:** 1.0.0
**Layer:** Hybrid Rule + ML Layer

---

## **Purpose**

Combine rule-based validation and ML predictions into a **Unified C-Script** used by the payload generator.
This script becomes the “blueprint” for generating a working exploit.

---

## **Inputs**

| Field            | Type   | Description                   |
| ---------------- | ------ | ----------------------------- |
| `context`        | Object | Extracted context from Step 1 |
| `ml_predictions` | Object | attackClass + payloadType     |

---

## **Outputs**

**C-Script JSON Structure**

```json
{
  "attackClass": "",
  "payloadType": "",
  "riskScore": 0.92,
  "sink": "",
  "allowedChars": "",
  "isDynamic": true
}
```

---

## **Workflow**

1. Accept context object
2. Run ML classifier → predict attackClass + payloadType
3. Apply rule-based corrections if prediction contradicts DOM evidence
4. Assign riskScore (0–1 float)
5. Produce final C-Script for payload matching

---

## **Dependencies**

| Module      | Purpose                        |
| ----------- | ------------------------------ |
| ML Models   | Provide primary classification |
| Rule Engine | Validates ML output            |
| Risk Engine | Computes risk score            |

---

## **Exceptions**

| Error                 | Trigger                                |
| --------------------- | -------------------------------------- |
| `InvalidContextShape` | Missing fields needed for construction |
| `MLFailure`           | Model cannot load or predict           |

---

---