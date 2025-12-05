# JSON Parsing & Source Normalization

**Version:** 1.0.0  
**Layer:** Gateway Layer → Middleware (Stage-2)

## **Purpose**

This middleware converts raw HTTP responses into a structured and normalized JSON format.
It extracts relevant components from the HTML and produces a consistent, machine-readable representation for downstream modules such as Context Analysis and Payload Generation.

## **Input**

| Field               | Type          | Description                                              |
| ------------------- | ------------- | -------------------------------------------------------- |
| `raw_http_response` | Object/String | Complete response including raw HTML, status and headers |

## **Output**

Normalized JSON structure derived from the page:

```json
{
  "url": "",
  "title": "",
  "text": "",
  "links": [],
  "scripts": [],
  "forms": [],
  "raw_html": ""
}
```

This standardized representation becomes the official transferable format across modules.

## **Workflow**

1. Receive HTTP response from Source Retrieval Layer
2. Extract HTML body, headers, and metadata
3. Parse DOM → identify relevant elements
4. Normalize output into unified JSON schema
5. Forward JSON to Context Processing via Gateway

## **Dependencies**

| Module                    | Responsibility                        |
| ------------------------- | ------------------------------------- |
| Source Retrieval Layer    | Supplies raw response and page source |
| (Optional) Content Filter | Removes noise, minifies HTML          |

## **Error Handling**

| Error                   | Trigger                | Module Response                        |
| ----------------------- | ---------------------- | -------------------------------------- |
| `InvalidResponseFormat` | Non-HTML or empty data | Return structured error JSON           |
| `ParsingFailure`        | DOM extraction fails   | Return JSON with raw_html only         |
| `MissingCoreFields`     | Title, text absent     | Return minimal JSON — never block flow |

**Failure must degrade → not abort pipeline.**

## **Configuration**

```json
{
  "extract": ["scripts", "forms", "text", "links"],
  "strip_tags": true,
  "preserve_html": true,
  "max_char_limit": 150000
}
```

## **Notes**

* This module **does not interpret meaning**, only normalizes structure.
* Uniform output is essential so every next module works without restructuring.
* Raw HTML is preserved for reference unless explicitly disabled.
