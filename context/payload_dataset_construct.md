# **Payload Dataset Construction**

**Version:** 1.0.0
**Layer:** Payload Repository Layer

---

## **Purpose**

Collect and normalize XSS payloads into a single, consistent dataset with metadata.
Includes thousands of payloads from research sources, GitHub, bug bounty lists, and custom generators.

---

## **Inputs**

| Field         | Type      | Description                     |
| ------------- | --------- | ------------------------------- |
| `payload.txt` | Text file | Raw, unstructured payload lists |

---

## **Outputs**

Cleaned and structured payload dataset:

```json
{
  "attackClass": "event-handler",
  "payloadType": "js-exec",
  "payload": "onclick=alert(1)"
}
```

---

## **Workflow**

1. Read raw payload file
2. Clean unwanted characters, duplicates, invalid patterns
3. Extract metadata tags (e.g., `<ATTACK_CLASS>HTML_BREAKOUT</ATTACK_CLASS>`)
4. Validate payload format
5. Store into structured text/JSON format

---

## **Dependencies**

| Library    | Purpose                       |
| ---------- | ----------------------------- |
| Regex      | Payload cleanup & tag parsing |
| Normalizer | Ensures consistent formatting |

---

## **Exceptions**

| Error                  | Trigger                     |
| ---------------------- | --------------------------- |
| `TagMissingError`      | Payload missing attackClass |
| `InvalidPayloadSyntax` | Broken or malformed payload |

---

---