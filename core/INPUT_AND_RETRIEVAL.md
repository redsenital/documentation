# Target Input & Source Retrieval

**Version:** 1.0.0  
**Layer:** Gateway Layer — Request Entry Point

## Purpose

This module receives incoming client requests, validates target URLs, fetches the webpage content, and forwards the structured output to the next micro-service in the pipeline.

## Inputs

| Parameter     | Type   | Description                         |
| ------------- | ------ | ----------------------------------- |
| `website_url` | String | The target URL to fetch and inspect |

## Outputs

| Output            | Description                                                           |
| ----------------- | --------------------------------------------------------------------- |
| `Response Object` | Contains HTTP status, headers, raw page source or parsed JSON version |

## Workflow

1. Accept request from client.
2. Validate URL format and accessibility.
3. Fetch the webpage from the provided input URL.
4. Parse the source into predefined JSON format.
5. Forward processed/output content to **Context Module** via `POST /endpointforcontextmicroservice`.

## Exceptions

| Exception                  | Trigger                                     |
| -------------------------- | ------------------------------------------- |
| `InvalidURLException`      | URL format incorrect or malformed           |
| `WebsiteNotFoundException` | Remote resource unreachable or non-existent |
