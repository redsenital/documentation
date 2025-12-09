

# **Context Extraction System**

**Version:** 1.0.0
**Layer:** Context Module — Static + Dynamic Extraction Layer

---

## **Purpose**

This component extracts all potential XSS-relevant contexts from a webpage.
It gathers **static HTML structure** using BeautifulSoup and **dynamic DOM behavior** using Playwright.
The goal is to identify every possible injection point where malicious payloads may execute.

---

## **Inputs**

| Field             | Type   | Description                                      |
| ----------------- | ------ | ------------------------------------------------ |
| `raw_html`        | String | Page source returned from fetcher                |
| `browser_runtime` | Object | Playwright browser instance for dynamic analysis |

---

## **Outputs**

### **Context Object (per injection point)**

```json
{
  "contextType": "attribute | html | script | event-handler",
  "tag": "",
  "attribute": "",
  "value": "",
  "flags": {
    "has_html_entities": false,
    "has_url_encoding": false,
    "has_js_escape": false
  },
  "dynamic": {
    "innerHTML_sink_used": false,
    "outerHTML_sink_used": false,
    "has_event_listeners": false,
    "created_by_js": false
  }
}
```

---

## **Workflow**

1. Parse HTML using **BeautifulSoup (bs4)**
2. Enumerate tags, attributes, JS URLs, event handlers
3. Launch Playwright browser
4. Capture dynamic event listeners & DOM mutations
5. Merge static + dynamic properties
6. Output full list of contexts for ML processing

---

## **Dependencies**

| Library/Module          | Purpose                                 |
| ----------------------- | --------------------------------------- |
| BeautifulSoup (bs4)     | Static DOM parsing                      |
| Playwright              | Dynamic DOM + event listener extraction |
| HTML Sanitization Flags | Detect escaping & encoding              |
| Context Schema          | Ensures consistent output format        |

---

## **Exceptions**

| Exception            | Trigger                                |
| -------------------- | -------------------------------------- |
| `HTMLParseFailure`   | Invalid or corrupted HTML              |
| `DynamicScanTimeout` | JS execution exceeds timeout           |
| `MissingDOM`         | Page fails to load in headless browser |

---

---
