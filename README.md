# SubmitFlow

A production AI agent that automates document intake for independent insurance agencies. Built on the Anthropic Claude API.

## What it does

SubmitFlow takes inbound submission documents (typically PDFs sent via email), classifies them, extracts structured data with confidence scoring, and routes the output to the right destination with human review built in for low-confidence cases.

The goal: replace the manual intake work that agencies do every day, accurately enough to trust in production, with guardrails for the cases where AI shouldn't be trusted alone.

## Architecture
Inbound Email (IMAP)
↓
Document Parser (pdfplumber for text PDFs, Claude Vision API for scans)
↓
Anthropic Claude API (Sonnet 4.6) — extraction, classification, confidence scoring
↓
Validation Layer — JSON schema enforcement, confidence thresholds
↓
┌────────────────────────────┐
│  High confidence outputs   │ → Structured Excel (openpyxl) → Outbound delivery (Resend)
└────────────────────────────┘
┌────────────────────────────┐
│  Low confidence outputs    │ → Human-in-the-loop review queue
└────────────────────────────┘
↓
Audit log + monitoring
## Key features

- **Two-pass extraction.** Text-first extraction via pdfplumber for native PDFs, Claude Vision API fallback for scans. Cheaper and more accurate when text is extractable.
- **Confidence scoring per field.** Low-confidence fields route to human review instead of silently failing.
- **Reusable prompt framework.** Prompts are templated and version-controlled, not hardcoded.
- **Reliability patterns.** Retry with exponential backoff, rate limiting, dead-letter recovery, audit logging.
- **PII handling.** Sensitive data is logged with care and bounded by access controls.
- **Always-on deployment.** Runs as a 24/7 worker process on Railway.

## Tech stack

- **Language:** Python 3.13
- **AI:** Anthropic Claude API (Sonnet 4.6 + Vision)
- **Document processing:** pdfplumber, Claude Vision API
- **Output:** openpyxl for structured Excel
- **Email:** IMAP (inbound), Resend API (outbound)
- **Deployment:** Railway (24/7 worker)
- **Version control:** Git, GitHub
- **AI-assisted development:** Claude Code

## Status

In production. Currently being piloted with independent insurance agencies. Iterating on accuracy, edge case handling, and expansion to additional document types based on operational feedback.

## Built by

Emmanuel Odetola, AI builder, MBA in Business Analytics. Based in Dallas, TX.
