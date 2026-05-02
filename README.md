# SubmitFlow

Production AI agent for submission intake at independent Property & Casualty insurance agencies. Reads ACORD forms, dec pages, and loss runs out of an inbox. Outputs a prefilled Excel workbook to the CSR before they expected to start triaging.

Built and operated solo. Anthropic Claude API. Python. Deployed to Railway.

## What it does

A submission email lands in an agency's monitored inbox. SubmitFlow:

1. Classifies the email (new business, renewal, COI request, marketing noise)
2. Parses every attached PDF — pdfplumber for native text, Claude Vision for scans
3. Extracts structured data with source tracing back to the page and section
4. Flags conflicts between documents and low-confidence fields
5. Generates an AMS-formatted Excel workbook (Applied Epic, AMS360, HawkSoft, or generic)
6. Sends the workbook to the CSR with a confirmation request
7. Watches for the reply, parses corrections, updates status, logs the audit trail

Human-in-the-loop on every submission. Zero fabrication. Missing fields are null, never guessed.

## Architecture

```
Inbound IMAP → Pre-classification filter → Claude classifier
                                                ↓
                  Document parser (pdfplumber + Claude Vision)
                                                ↓
                  Field extraction + source tracing + PII redaction
                                                ↓
                  Conflict detection + confidence scoring
                                                ↓
                  AMS-specific Excel writer (openpyxl)
                                                ↓
                  Resend HTTP transport → CSR inbox
                                                ↓
                  Reply monitor → confirmation parser → audit log
```

## Design decisions worth noting

- **Two-pass extraction.** Text first, vision fallback. Most ACORDs are native PDFs and pdfplumber is faster and cheaper than Vision. Vision only fires when text extraction returns below a quality threshold.
- **Source tracing on every field.** Document name, page number, section. Required for compliance defensibility and for the CSR to verify questionable fields in under 30 seconds.
- **Per-client isolation.** One client's failure cannot affect another. Per-client try/except in the main loop, per-client config, per-client API budget.
- **Resend over SMTP.** Railway blocks outbound SMTP. Resend HTTP transport is primary, SMTP is fallback for local dev only.
- **Persistent state on disk, not in a queue broker.** JSON files handle dedup, HITL state, and audit logs at this scale. Redis and SQS are deferred until they actually solve a problem.
- **No agent framework.** Considered the Claude Agent SDK and rejected it. Autonomous tool selection breaks the auditability requirement for regulated workflows.

## Tech stack

- Python 3.13
- Anthropic Claude API (Sonnet 4.6, text and vision)
- pdfplumber, pdf2image, Pillow, pytesseract
- openpyxl
- Native Python imaplib for inbound, Resend HTTP API for outbound
- Railway for deployment

## Compliance

Every client deployment includes a governance package: data processing agreement, AI system description, vendor risk assessment, audit trail. Sized for state insurance department market conduct examinations. More than half of US states have adopted the NAIC AI Model Bulletin or substantially similar guidance, and the NAIC AI Systems Evaluation Tool entered multistate pilot in January 2026.

## Repo status

Production code is private. This repo documents architecture and design. For technical discussion or walkthrough, reach out.

## Built by

Emmanuel Odetola. MBA Business Analytics. Based in DFW.
