# AI Ethics Digest | Automated Content Curation Pipeline

An automated Make (Integromat) scenario that monitors AI ethics and governance news, uses an LLM to analyze each article, and delivers a curated digest by email, built as a personal research tool to stay current on a fast-moving field.

## Problem

Keeping up with AI ethics and governance developments means scanning dozens of sources daily, most of which aren't relevant enough to justify the time. Manually triaging this volume doesn't scale.

## Solution

An automated pipeline that:
1. Watches a Google Alerts RSS feed for new AI ethics / governance articles
2. Sends each article through an LLM with a domain-expert prompt to assess its content and relevance
3. Emails a formatted, pre-filtered digest - so only the "is this worth reading" decision remains manual

```
Google Alerts (RSS) → Make AI Toolkit (LLM analysis) → Gmail (digest) 
                              ↓ (on failure)
                         Retry (3x, 5 min apart)
```

## Architecture

| Module | Function |
|---|---|
| **RSS - Watch feed items** | Polls a Google Alerts feed (used as a lightweight RSS aggregator) for new articles matching AI ethics / governance search terms |
| **Make AI Toolkit - Simple Text Prompt** | Sends each article's title, description, and URL to an LLM with a structured prompt (see below) |
| **Gmail - Send an email** | Delivers the formatted analysis as a digest email |
| **Flow Control - Retry** | Catches failed executions (e.g. API errors) and retries automatically: 3 attempts, 5 minutes apart, before flagging for manual review |

## The prompt

The core of the pipeline is a domain-expert prompt that turns a raw article into a structured, skimmable analysis:

```
You are an expert in AI ethics and AI governance. Analyze the article below, which you found online.

Title: {{1.title}}
Description/Abstract: {{1.description}}
Link: {{1.url}}

Prepare a brief summary for the user:

What is this article about? (max 2 sentences)

What specific problem, bias, or impact assessment method does it describe?

Recommendation: Is it worth reading the whole article? (Scale 1–5 with a brief explanation).

Formatting rules:
1. Do NOT use markdown asterisks (**) for bolding.
2. Use ONLY HTML <strong> tags
3. Every section and answer MUST be separated by <br><br> tags to ensure proper line breaks.
```

Two details worth noting:
- The prompt asks for a **1–5 relevance rating**, not just a summary. The goal is triage, not just compression.
- **Formatting rules are dictated by the downstream consumer** (Gmail renders HTML, not markdown) - a small but deliberate integration detail that avoids broken formatting in the final email.

## Error handling

Failed executions (e.g. a transient Gmail or LLM API error) are caught by a **Flow Control – Retry** module: up to 3 automatic retries, 5 minutes apart. If all retries fail, the execution is flagged as incomplete for manual review rather than silently dropped.

## Source

Articles are aggregated via a **Google Alerts RSS feed**, using Google's own alerting as a lightweight, zero-maintenance aggregation layer instead of integrating with multiple news sources directly.

## Known limitations

- Relies on Google Alerts' own relevance matching upstream; the LLM step filters *within* what Alerts surfaces, not from a broader source set
- Digest currently goes to a single recipient (personal research use); not yet built for multi-subscriber distribution
- Relevance scoring is model-judged, not benchmarked against a labeled dataset

## Stack

Make (Integromat) · Google Alerts (RSS) · Make AI Toolkit (LLM) · Gmail

## Screenshots

- full scenario diagram -> `screenshots/make.png` 
- Flow Control retry configuration -> `screenshots/Retry.png` 
