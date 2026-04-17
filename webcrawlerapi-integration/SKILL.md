---
name: WebCrawlerAPI Integration
description: This skill should be used when the user asks to "integrate webcrawlerapi", "add webcrawlerapi to my project", "use webcrawlerapi in my code", "scrape a website with webcrawlerapi", "crawl pages using webcrawlerapi", "set up webcrawlerapi", "add web scraping to my app", or asks how to use the WebCrawlerAPI SDK in JavaScript, TypeScript, or Python. Also triggers when the user wants to scrape a single page, crawl multiple pages, extract structured data, or set up website change monitoring using WebCrawlerAPI.
version: 0.1.0
---

# WebCrawlerAPI Integration

WebCrawlerAPI is a web scraping and crawling service. The base API URL is `https://api.webcrawlerapi.com`.

Full documentation: **https://webcrawlerapi.com/docs**  
API reference: **https://webcrawlerapi.com/docs/api**

## Authentication

All requests require a Bearer token in the `Authorization` header:

```
Authorization: Bearer YOUR_API_KEY
```

API keys are created at https://dash.webcrawlerapi.com.

## Two Core Operations

### 1. Scrape a single page — `POST /v2/scrape`

Returns page content immediately (synchronous by default).

```bash
curl --request POST \
  --url https://api.webcrawlerapi.com/v2/scrape \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{"url": "https://example.com", "output_formats": ["markdown"]}'
```

Response fields: `success`, `status`, `markdown`, `html`, `cleaned`, `page_status_code`, `page_title`.

Add `?async=true` to get a job ID instead of waiting. Retrieve result via `GET /v2/scrape/{id}`.

### 2. Crawl multiple pages — `POST /v1/crawl`

Async by default — returns a job immediately and crawls in the background.

```bash
curl --request POST \
  --url https://api.webcrawlerapi.com/v1/crawl \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com",
    "items_limit": 10,
    "output_formats": ["markdown"],
    "max_depth": 2
  }'
```

Poll status: `GET /v1/job/{id}` — use `recommended_pull_delay_ms` from the response for polling interval.

Job statuses: `new`, `in_progress`, `done`, `error`.  
Item statuses: `new`, `in_progress`, `done`, `error`.

## Output Formats

Set via `output_formats: ["markdown"]` (array, can combine):

| Format | Description | Best for |
|--------|-------------|----------|
| `markdown` | Content with markdown formatting | LLMs, AI pipelines |
| `cleaned` | Plain text, no HTML noise | NLP, text processing |
| `html` | Raw HTML | Custom parsing |
| `links` | List of links found | Link extraction |

Default is `["markdown"]`.

## SDK Installation

### JavaScript / TypeScript
```bash
npm install webcrawlerapi-js
```

```javascript
import webcrawlerapi from "webcrawlerapi-js";
const client = new webcrawlerapi.WebcrawlerClient("YOUR_API_KEY");

// Synchronous (waits for completion)
const result = await client.crawl({ url: "https://example.com", output_formats: ["markdown"], items_limit: 10 });

// Async (returns immediately)
const job = await client.crawlAsync({ url: "https://example.com", output_formats: ["markdown"], items_limit: 10 });
const jobId = job.id;
let status = await client.getJob(jobId);
while (status.status === "in_progress") {
    await new Promise(r => setTimeout(r, status.recommended_pull_delay_ms));
    status = await client.getJob(jobId);
}
```

### Python
```bash
pip install webcrawlerapi
```

```python
from webcrawlerapi import WebCrawlerAPI

crawler = WebCrawlerAPI(api_key="YOUR_API_KEY")

# Synchronous
result = crawler.crawl(url="https://example.com", output_formats=["markdown"], items_limit=10)

# Async with polling
import time
job = crawler.crawl_async(url="https://example.com", output_formats=["markdown"], items_limit=10)
status = crawler.get_job(job.id)
while status.status == "in_progress":
    time.sleep(status.recommended_pull_delay_ms / 1000)
    status = crawler.get_job(job.id)
```

## Crawl Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `url` | string | Required. Seed URL |
| `items_limit` | int | Max pages (default: 10) |
| `output_formats` | array | Formats to generate |
| `max_depth` | int | Link depth from seed (0 = seed only) |
| `whitelist_regexp` | string | Only crawl matching URLs |
| `blacklist_regexp` | string | Skip matching URLs |
| `webhook_url` | string | POST result here when done |

## Structured Data Extraction (AI-powered)

Add `prompt` to `/v2/scrape` to extract structured data with AI. Add `response_schema` for strict JSON output.

```json
{
  "url": "https://example.com/product",
  "prompt": "Extract product name, price, and availability",
  "response_schema": {
    "type": "object",
    "properties": {
      "name": {"type": "string"},
      "price": {"type": "number"},
      "in_stock": {"type": "boolean"}
    },
    "required": ["name", "price", "in_stock"],
    "additionalProperties": false
  }
}
```

Returns `structured_data` field in response. Cost: $0.002/request extra.

## Webhooks

Provide `webhook_url` in any request. The server POSTs the full job object to that URL when done — no polling needed. Docs: https://webcrawlerapi.com/docs/async-requests

## Website Change Monitoring (Feeds)

Create a feed to monitor a website and receive updates when content changes.

```bash
curl --request POST \
  --url https://api.webcrawlerapi.com/v1/feeds \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --data '{
    "url": "https://example.com",
    "webhook_url": "https://yourserver.com/hook",
    "output_format": "markdown"
  }'
```

Docs: https://webcrawlerapi.com/docs/feeds

## Error Handling

Job-level errors (in `error_code`):
- `insufficient_balance` — top up at https://dash.webcrawlerapi.com
- `invalid_request` — check parameters
- `internal_error` — contact support@webcrawlerapi.com

Item-level errors (in `job_items[].error_code`):
- `website_access_denied` — 403 response
- `blocked_by_robots_txt` — robots.txt blocked
- `timeout_error` — retry; if persistent, site has anti-bot protection
- `webpage_non_success` — empty or blocked content
- `duplicate_item` — same content seen before; not charged

Full error reference: https://webcrawlerapi.com/docs/errors

## Rate Limits

- 10 parallel threads per account
- 10 parallel threads per target website (shared across all accounts)

Docs: https://webcrawlerapi.com/docs/rate-limits

## Other SDKs

- PHP: https://webcrawlerapi.com/docs/sdk/php
- .NET: https://webcrawlerapi.com/docs/sdk/dotnet
- Java: https://webcrawlerapi.com/docs/sdk/java
- LangChain: https://webcrawlerapi.com/docs/sdk/langchain
- n8n: https://webcrawlerapi.com/docs/sdk/n8n
- Zapier: https://webcrawlerapi.com/docs/sdk/zapier
- Make: https://webcrawlerapi.com/docs/sdk/make
- MCP: https://webcrawlerapi.com/docs/sdk/mcp
