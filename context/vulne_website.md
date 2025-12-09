
# **XSS Payload Execution Tester**

**Version:** 1.0.0
**Layer:** Execution Layer — Real Browser Testing

---

## **Purpose**

Verify whether a payload **actually executes** using a real browser environment.
Built with Selenium WebDriver for automated execution testing.

---

## **Inputs**

| Field      | Type   | Description                                             |
| ---------- | ------ | ------------------------------------------------------- |
| `payload`  | String | Payload to test                                         |
| `template` | String | Vulnerability template (e.g., attribute, event-handler) |

---

## **Outputs**

```json
{
  "payload": "",
  "executed": true,
  "evidence": "alert(1)",
  "dom_snapshot": "<html>...</html>"
}
```

---

## **Workflow**

1. Insert payload into a generated HTML template
2. Launch **Selenium ChromeDriver**
3. Simulate user events (click, mouseover, load)
4. Capture alert dialogs or DOM modifications
5. Write successful payloads to `working_payloads.txt`

---

## **Dependencies**

| Module               | Purpose                         |
| -------------------- | ------------------------------- |
| Selenium WebDriver   | Execute payload in real browser |
| Chrome               | XSS execution environment       |
| HTML Template Engine | Renders test pages              |

---

## **Exceptions**

| Error                    | Trigger                               |
| ------------------------ | ------------------------------------- |
| `AlertNotCapturedError`  | Payload executed but alert not logged |
| `BrowserCrash`           | Chrome failure                        |
| `TimeoutDuringExecution` | Payload does not execute in time      |

---

---