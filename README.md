# VH_ANNUAL_REPORT

A two-stage agent workflow for OCR-first extraction and traceable, page-by-page analysis of annual reports and PDF decks (often image-based). The system produces a structured per-page extraction JSON and a consultant-style narrative analysis that preserves page boundaries and page order.

Overview

Many annual reports and presentation-style PDFs are image-heavy, which makes simple text extraction unreliable. This workflow addresses that by splitting the problem into two strictly separated stages:

Visual OCR extraction (Agent 1)
Converts each page into structured, machine-readable data without interpretation.

Page-preserving strategic analysis (Agent 2)
Writes a robust, traceable analysis section per page, then produces end-of-report recommendations and questions.

The core design goal is to produce analysis that is both readable and auditable: every section maps to a specific page object, and the system is explicitly robust to common OCR/pagination issues.

Key properties

OCR-first ingestion for image-based PDFs

Structured extraction output (JSON) with per-page artifacts:

verbatim OCR text

tables (rows)

charts (type, units, axes/legend, values)

key numbers (metric/value/unit)

extraction issues list

Strict separation of responsibilities:

Agent 1: transcription-only (no analysis)

Agent 2: analysis-only (no “fixing” the PDF or inventing facts)

Traceable output:

One section per page in exact order

Uses both the page number and an internal report index

Robustness to real-world extraction errors:

divider/section-marker pages

text leakage across pages

incorrect or inconsistent page numbering

Architecture
Agent 1 — Visual PDF reader (OCR transcription)

Role: Convert each page to structured JSON without interpretation.

Output schema (Agent1Schema) includes:

page_count

pages[] where each page contains:

page_number

title

verbatim_ocr (always present, may be empty string)

tables[] (title + rows)

charts[] (typed + values + axes/legend + unit)

key_numbers[] (metric/value/unit)

issues[] for uncertainty or extraction problems

Constraints:

Never guesses numbers

Preserves formatting exactly (separators, decimal commas/points, %, currency)

Uses issues[] to flag uncertainty instead of fabricating values

Agent 2 — Strategic report writer (page-by-page, robust)

Role: Produce a consultant-style analysis that never breaks page boundaries.

Core constraints:

Never skips pages that exist in pages[]

Never merges pages into free-form chapters

Always outputs sections in exact order

Uses:

page_number exactly as provided

an internal sequence index R{n} (1..N)

Output structure:

For each page:

## R{n} — Sida {page_number} — {rubrik}

Divider pages:

### Analys / Kommentar (2–4 sentences)

Data-heavy pages:

### Key Insights with 2–4 subheadings, each with implication-focused analysis

End sections:

## Rekommendationer framåt (synthesized bullets)

## Övergripande slutsatser

## Frågor att ta vidare

Anti-hallucination rules:

Only claim “shown/stated” if supported by extracted text/visual descriptions

Any inference must be explicitly marked as inference (“Detta kan tyda på…”)

How to use

Provide a PDF (annual report or slide deck).

The workflow outputs:

A per-page extraction JSON (OCR text, tables, charts, key numbers, issues)

A structured, page-by-page analysis plus final recommendations

Why this is useful

Converts image-heavy PDFs into structured data suitable for review, reporting, and downstream analytics

Produces a narrative analysis while preserving traceability to the original source pages

Handles noisy extraction outputs without collapsing the structure or inventing facts

Notes for reviewers

This repository emphasizes reliability over “pretty summarization”:

extraction is strictly transcription-only

analysis is forced to remain page-anchored and auditable

common OCR failure modes are handled explicitly via rules rather than hidden heuristics
