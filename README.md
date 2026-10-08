<div align="center">

# 🎓 Bulk Certificate Generator & Issuance Engine

**An enterprise-grade, asynchronous backend API and interactive management portal designed for issuing, validating, and retrieving high-resolution PDF certificates in bulk.**

[![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688.svg?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Python](https://img.shields.io/badge/Python-3.10%20%7C%203.14-3776AB.svg?logo=python&logoColor=white)](https://www.python.org/)
[![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-2.0-D71F00.svg?logo=sqlalchemy&logoColor=white)](https://www.sqlalchemy.org/)
[![ReportLab](https://img.shields.io/badge/PDF_Engine-ReportLab-blue.svg)](https://www.reportlab.com/)
[![Tests](https://img.shields.io/badge/Tests-9%20Passed%20(100%25)-brightgreen.svg?logo=pytest&logoColor=white)](https://docs.pytest.org/)
[![License](https://img.shields.io/badge/License-MIT-purple.svg)](LICENSE)

<br/>

[Explore Architecture](#-system-architecture) •
[Quick Start](#-quick-start-in-3-steps) •
[API Reference](#-api-endpoints--usage) •
[Visual Tour](#-visual-tour--screenshots) •
[Interview Defenses](#-technical-interview-faq)

<br/>

---

</div>

## 🌟 What Problem Does This Project Solve?

Educational institutions, bootcamps, and conference organizers often need to issue **hundreds or thousands of certificates** immediately after an event concludes.

❌ **The Traditional Way**:
- Generating certificates one by one crashes browser tabs or freezes servers.
- A single typo or corrupt email address breaks the entire batch.
- Manual file organization requires hours of downloading individual files.

✅ **The Solution (Bulk Certificate Generator)**:
- **Asynchronous Execution**: Submit 500+ recipients in a single request; the server accepts it instantly (`HTTP 202 Accepted`) and processes it in the background without client timeouts.
- **Fault-Tolerant Isolation**: If Recipient #42 fails (e.g., malformed data), the system marks only that item as `FAILED` with diagnostics and **continues generating all remaining 499 certificates**.
- **Live Progress Tracking**: Monitor progress in real time (`Total`, `Completed`, `Failed`) with live progress bars.
- **Flexible Downloads**: Download individual recipient PDFs or grab everything at once in a single **compressed ZIP archive**.
- **Vector-Sharp Design**: Generates bespoke, archival-quality certificates with serif typography, heraldic crests, and verifiable digital seals.

---

## 📸 Visual Tour & Screenshots

### 1. Interactive Portal Dashboard (`/`)
> *Submit bulk batches, pick preset templates, monitor real-time generation progress, and download individual or zipped certificates.*

<div align="center">
  <img src="docs/images/dashboard_preview.png" alt="Interactive Dashboard Preview" width="850px" style="border-radius: 8px; box-shadow: 0 8px 30px rgba(0,0,0,0.3);" />
</div>

<br/>

### 2. High-Resolution Archival Certificate Outputs (Real Generated Samples)
> *Bespoke letter landscape certificates (11" x 8.5") generated from batch job `042a3794`, featuring archival framing, heraldic star crest, embossed gold foil authenticity seal, and cryptographic audit UUID.*

<div align="center">
  <img src="docs/images/Janardan_Kumar_sample.png" alt="Generated Certificate - Janardan Kumar" width="850px" style="border-radius: 8px; box-shadow: 0 8px 30px rgba(0,0,0,0.3); margin-bottom: 16px;" />
</div>

<br/>

<div align="center">
  <img src="docs/images/Ritika_Vaishnavi_sample.png" alt="Generated Certificate - Ritika Vaishnavi" width="850px" style="border-radius: 8px; box-shadow: 0 8px 30px rgba(0,0,0,0.3); margin-bottom: 16px;" />
</div>

<br/>

<div align="center">
  <img src="docs/images/Rishav_Kumar_sample.png" alt="Generated Certificate - Rishav Kumar" width="850px" style="border-radius: 8px; box-shadow: 0 8px 30px rgba(0,0,0,0.3);" />
</div>

<br/>

### 3. Interactive API Documentation (`/docs`)
> *Full OpenAPI (Swagger) interface for inspecting schema definitions, trying endpoints, and testing payloads.*

<div align="center">
  <img src="docs/images/swagger_preview.png" alt="Interactive Swagger Documentation" width="850px" style="border-radius: 8px; box-shadow: 0 8px 30px rgba(0,0,0,0.3);" />
</div>

---

## 🏗️ System Architecture & Workflow

Here is how data flows through the application:

```text
[ Client / Web Browser / API ]
             │
             │  1. POST /api/certificates/generate (Bulk JSON)
             ▼
┌──────────────────────────────────────────────┐
│ FastAPI Gateway                              │
│ • Validates schema using Pydantic v2         │
│ • Rejects blank names & invalid email syntax │
└──────────────────────┬───────────────────────┘
                       │
                       │  2. Creates Job & Recipient Rows (PENDING)
                       ▼
┌──────────────────────────────────────────────┐
│ Relational Database (SQLAlchemy 2.0)         │
│ • generation_jobs (Parent Table)             │
│ • certificate_items (Child Table)            │
└──────────────────────┬───────────────────────┘
                       │
                       │  3. Returns HTTP 202 Accepted { job_id }
                       │     Spawns Background Worker
                       ▼
┌──────────────────────────────────────────────┐
│ Background Processing Pipeline               │
│ • Loops through recipients with try/except   │
│ • ReportLab renders vector landscape PDF    │
│ • If item fails: records error, keeps going! │
│ • Updates live counters in Database          │
└──────────────────────┬───────────────────────┘
                       │
                       │  4. Poll status / Retrieve output
                       ▼
[ GET /api/certificates/jobs/{id} ] ──► Live Progress & Individual PDF URLs
[ GET /api/certificates/jobs/{id}/download-all ] ──► Instant ZIP Stream
```

---

## ⚡ Quick Start in 3 Steps

### Step 1: Open the Project
Open your terminal (VS Code Terminal, CMD, or PowerShell) and navigate to the folder:
```bash
cd bulk-certificate-generator
```

### Step 2: Install Dependencies
```bash
pip install -r requirements.txt
```

### Step 3: Start the Application
```bash
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload
```

🎉 **That's it! Open your web browser:**
- **Interactive UI Portal**: [http://127.0.0.1:8000/](http://127.0.0.1:8000/)
- **Interactive Swagger API Docs**: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
- **Alternative ReDoc Docs**: [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)

---

## 🔌 API Endpoints & Usage

### 1. Submit a Bulk Certificate Job
- **URL**: `POST /api/certificates/generate`
- **Status**: `202 Accepted`

```json
{
  "title": "Full-Stack Distributed Systems Masterclass",
  "issuer": "Open Software Foundation",
  "description": "For building fault-tolerant high-concurrency microservices",
  "issue_date": "October 2026",
  "recipients": [
    { "name": "Eleanor Vance", "email": "eleanor.vance@veritas.edu" },
    { "name": "Marcus Aurelius", "email": "marcus.a@veritas.edu" }
  ]
}
```

**Response**:
```json
{
  "job_id": "7fae2985-611f-4ff5-bda7-ffadce5bc4a9",
  "message": "Bulk certificate generation request accepted and queued for processing.",
  "status": "PENDING",
  "total_recipients": 2
}
```

---

### 2. Check Job Progress & Results
- **URL**: `GET /api/certificates/jobs/{job_id}`
- **Status**: `200 OK`

```json
{
  "id": "7fae2985-611f-4ff5-bda7-ffadce5bc4a9",
  "title": "Full-Stack Distributed Systems Masterclass",
  "status": "COMPLETED",
  "total_recipients": 2,
  "successful_count": 2,
  "failed_count": 0,
  "certificates": [
    {
      "id": "1c7b809a-8cbb-46eb-8a4a-14d23eb81a11",
      "recipient_name": "Eleanor Vance",
      "status": "SUCCESS",
      "download_url": "/api/certificates/1c7b809a-8cbb-46eb-8a4a-14d23eb81a11/download",
      "error_message": null
    }
  ]
}
```

---

### 3. Download Single Certificate PDF
- **URL**: `GET /api/certificates/{certificate_id}/download`
- **Output**: Streams binary `application/pdf` with `Content-Disposition: attachment; filename="Certificate_Eleanor_Vance_1c7b809a.pdf"`.

---

### 4. Download All Certificates as ZIP
- **URL**: `GET /api/certificates/jobs/{job_id}/download-all`
- **Output**: Streams an on-the-fly compressed `application/zip` containing all successful certificates.

---

## 🧪 Automated Testing

This project comes with a comprehensive test suite in `tests/test_api.py` running against an isolated in-memory SQLite database (`StaticPool`).

To execute all tests:
```bash
python -m pytest tests -v
```

### Coverage Highlights:
- ✅ **`test_health_check`**: Verifies application availability.
- ✅ **`test_dashboard_ui`**: Verifies interactive portal serving.
- ✅ **`test_create_generation_job_success`**: Confirms batch ingestion and HTTP 202 response.
- ✅ **`test_input_validation_empty_recipients`**: Validates 422 Unprocessable Entity on empty lists.
- ✅ **`test_input_validation_invalid_email_and_blank_name`**: Tests rejection of invalid emails and whitespace-only names.
- ✅ **`test_job_status_and_certificate_generation`**: End-to-end background rendering verification.
- ✅ **`test_retrieve_generated_certificate`**: Ensures output is valid, non-empty `%PDF` binary.
- ✅ **`test_handling_individual_certificate_failure`**: Proves a bad record fails gracefully without breaking the batch.
- ✅ **`test_bulk_download_zip`**: Validates dynamic ZIP compilation and stream integrity.

---

## 📁 Repository Structure

```
bulk-certificate-generator/
│
├── app/
│   ├── main.py                # FastAPI app initialization, middleware, routes
│   ├── config.py              # Path definitions & database configuration
│   ├── database.py            # SQLAlchemy 2.0 engine, sessions & base
│   ├── models.py              # GenerationJob & CertificateItem ORM models
│   ├── schemas.py             # Pydantic v2 schemas and validation logic
│   ├── api/
│   │   └── routes.py          # REST endpoints (submit, poll, download, zip)
│   ├── services/
│   │   ├── generator.py       # ReportLab PDF vector engine & seal styling
│   │   └── job_processor.py   # Asynchronous batch worker with failure isolation
│   └── templates/
│       └── index.html         # Interactive web portal UI
│
├── tests/
│   └── test_api.py            # Complete automated Pytest test suite
│
├── docs/
│   └── images/                # Visual previews & screenshots for documentation
│
├── generated_certificates/    # Local storage directory for generated PDF files
├── demo.py                    # One-click end-to-end simulation script
├── generate_report_pdf.py     # Script to generate printable project report PDF
├── requirements.txt           # Project dependencies
└── README.md                  # Project documentation
```

---

## 💡 Technical Interview FAQ & Design Defenses

#### Q1: Why FastAPI over Flask or Django for this project?
> **Answer**: FastAPI provides native asynchronous task management, automatic OpenAPI documentation (`/docs`), and compile-time data validation via Pydantic v2. It offers higher throughput with significantly lower memory overhead compared to Django, while being more structured than Flask.

#### Q2: How does the system handle thousands of recipients without running out of memory?
> **Answer**:
> 1. Jobs are enqueued asynchronously; the client gets an immediate HTTP 202 response.
> 2. PDF rendering is vector-based via ReportLab (using minimal memory per document).
> 3. ZIP archives are compressed on-the-fly and streamed directly to the client without buffering all files in RAM.
> 4. In a production cluster, the background function can be routed to a distributed queue like Celery or Redis without altering the API interface.

#### Q3: What makes this fault-tolerant?
> **Answer**: In `app/services/job_processor.py`, every certificate generation call runs inside a localized `try...except` block. A missing font, broken name string, or I/O failure is recorded directly on that specific `CertificateItem` record (`status = FAILED`, `error_message = ...`), while the loop continues uninterrupted for all other valid recipients.

---

<div align="center">
  <sub>Engineered with precision for modern backend environments.</sub>
</div>
