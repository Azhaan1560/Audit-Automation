# AuditFlow AI: Automated Internal Control & Substantive Testing Platform
An automated audit and compliance platform designed to streamline financial statement auditing by digitizing internal control matching (2-way through 5-way) and substantive transaction testing across major business cycles.

## Prototype Scope (November 2026)
The November prototype proves the approach on one cycle, end to end, using **synthetic documents only** .

**In scope**
* Purchases testing at **2-way, 3-way and 4-way** levels (PO, GRN, QA inspection, vendor invoice).
* A custom snipping add-in for Excel: view a document beside the sheet, snip a value into a cell, click a cell to see its evidence.
* Automated testing: classify documents, extract fields with AI, apply matching rules, create snips, and flag exceptions with reason codes.
* Evidence packs (highlighted, labelled PDFs) and a full audit log.

**Out of scope until after the pilot**
* 5-way matching. *(Definition to be confirmed against the firm's methodology: logistics and tax documents, as described below, or payment on the bank statement.)*
* Sales & Revenue, OPEX and NPL testing. NPL (non-performing loan) testing needs its own rules and data (loan files, ageing, ECL recalculation) and will reuse the snipping tool and audit trail.
* Real client data, user sign-in, the review workflow and the web portal.

## Core Features

### 1. Procure-to-Pay (P2P) Control Engine
Automates the validation of vendor disbursements, supply chain logistics, and tax compliance to ensure operational integrity.
* **2-Way Matching:** Cross-references Purchase Orders (POs) and Vendor Invoices (pricing and terms verification for services/subscriptions).
* **3-Way Matching:** Automatically reconciles POs, Goods Receipt Notes (GRNs), and Vendor Invoices for physical goods and inventory.
* **4-Way Matching:** Integrates Quality Assurance (QA) inspection and technical acceptance reports before invoice clearance for high-spec capital expenditures (CapEx).
* **5-Way Matching:** Validates logistics documentation (Bills of Lading, transit tracking) and regulatory tax compliance (e.g., e-invoicing, e-way bills, and government tax portal ledger matching).

### 2. Substantive Testing Modules
Simulates automated audit procedures for key financial statement line items (FSLIs):
* **Purchase & OPEX Testing:** Verifies expense validity, cut-off accuracy, and classification to prevent misstatements between capital and operating expenses.
* **Sales & Revenue Testing:** Analyzes transaction streams for occurrence, accuracy, and proper period cut-off under standard revenue recognition guidelines.
* **Non-Performing Loan (NPL) Testing:** Evaluates credit risk, loan aging brackets, collateral valuations, and expected credit loss (ECL) provisions (targeted at financial institution audits).

## System Architecture & Tech Stack
The system is a **modular monolith**: one FastAPI service, layered inside, with a worker process that runs the document pipeline and an Excel add-in as the user interface.

**Layers**
* **API layer:** FastAPI routes, authentication, input validation.
* **Application layer:** use cases such as *run test*, *create snip*, *build evidence pack*.
* **Domain layer:** matching rules, tolerances and the snip model. Pure Python: no AI, no database, no file access.
* **Infrastructure layer:** adapters for the database, file storage, OCR and AI providers.

**Document pipeline (worker):** Ingest -> OCR -> Classify -> Extract (the only AI step) -> Locate -> Match -> Write back.

**Principles**
* AI only *reads* values from documents. Matching decisions are made by deterministic rules in `config/tests.yaml`.
* Every value used in a decision must be found on the page and linked to its evidence.
* Every run is logged: document hashes, model and prompt versions, results.

| Area | Prototype choice |
| --- | --- |
| API | FastAPI, Pydantic v2, Python 3.12 |
| Database | PostgreSQL 16, SQLAlchemy 2.0, Alembic |
| Background jobs | PostgreSQL-backed job queue, one worker process |
| File storage | Local folder, files named by SHA-256 |
| PDF and OCR | pdfplumber, pypdfium2, pypdf, PaddleOCR (PP-OCRv5) |
| AI access | LiteLLM with fallback across providers, results cached |
| Rules | YAML per test, validated with Pydantic |
| Excel add-in | Office.js, TypeScript, React, Fluent UI, PDF.js |
| Test data | Generator seeded with Microsoft AdventureWorks purchasing data (Jinja2 + WeasyPrint) |
| Tooling | uv, Docker Compose, Ruff, pytest |

## Project Structure
```
snipper/
  service/     FastAPI app, worker, document generator, tests
  addin/       Excel add-in (Office.js)
  config/      tests.yaml (matching rules), llm.yaml (AI providers)
  data/        seed data, templates, generated document sets, sample workbook
  eval/        accuracy scripts, baseline timings, results
  scripts/     one-off helper scripts
  docs/        notes and learning log
  pitch/       slides, demo script, backup video
```

## Getting Started
*To be completed as the build progresses.*

1. Copy `service/.env.example` to `service/.env` and fill in the values.
2. Start the stack: `docker compose up --build`
3. Check the API: open http://localhost:8000/health and http://localhost:8000/docs

## Data and Privacy
This prototype uses synthetic documents only. No client data may be added to this repository or processed by this system until the AI route is approved by the DPO and Risk, and the firm has agreed the evidence format.
