# Hospital Knowledge and Appointment Assistant

A complete, college-project-friendly backend system built with **FastAPI**, **PostgreSQL**, **SQLAlchemy**, and **FAISS**.

The assistant allows hospitals to manage departments, clinicians, patients, and appointment schedules while offering an intelligent, grounded **Retrieval-Augmented Generation (RAG)** chatbot that answers patient inquiries from verified hospital documents without hallucinations or fabrications.

---

## 1. Project Overview

Patients and visitors frequently have common questions regarding hospital departments, OPD hours, visiting restrictions, and appointment booking procedures. 

This project delivers:
1. **Administrative REST API**: Complete CRUD capabilities for departments, doctors, patients, and appointments with Role-Based Access Control (Admin, Staff, and User).
2. **Knowledge Document Ingestion**: Ingestion of PDF, DOCX, TXT, and Markdown documents with text extraction, word-level chunking, and dense vector embeddings.
3. **FAISS Vector Search**: Fast similarity search over knowledge chunks without external vector database infrastructure.
4. **Dual-Mode RAG Chatbot**:
   - `retrieval_only` (**Default**): Works 100% locally with zero external API keys or subscriptions.
   - `groq`: Optional integration with Groq Cloud LLMs (e.g. `llama-3.1-8b-instant`) for natural responses strictly grounded in retrieved hospital documents.
5. **Emergency Safety Guard**: Immediately intercepts acute medical emergencies (chest pain, shortness of breath, heavy bleeding, unconsciousness) and redirects patients to emergency care without delay.

---

## 2. Key Features

- **Authentication & Security**: Secure user registration, password hashing via `bcrypt`, and stateless JWT tokens.
- **Role-Based Authorization**:
  - `Admin`: Manage users, departments, doctors, patients, appointments, documents, and vector indexing.
  - `Staff`: View departments/doctors/patients, manage appointments, upload knowledge files.
  - `User`: Register, log in, browse departments and doctors, schedule appointments, and chat.
- **Hospital Management**:
  - Departments with active statuses and clinician mappings.
  - Doctor profiles linked to specialized departments.
  - Patient profiles.
  - Appointment scheduling with duplicate booking conflict prevention.
- **Document Processing Pipeline**:
  - Supported formats: `.pdf`, `.docx`, `.txt`, `.md`.
  - Maximum upload size: 10 MB with secure filename sanitization.
- **RAG & Vector Search**:
  - Dense semantic embeddings generated with `sentence-transformers/all-MiniLM-L6-v2`.
  - Local vector indexing using Meta's `FAISS` library.
  - Explicit citations and chunk references returned with every answer.
- **Built-in Web Interface**: Unified single-page shell at `http://127.0.0.1:8000/` or `http://127.0.0.1:8000/ui` (`/ui/index.html`) featuring RAG Chat, appointments booking/management, doctors & departments directory, patient records, document ingestion, and 1-click demo login.
- **Interactive Documentation**: Auto-generated Swagger UI available at `/docs` and ReDoc at `/redoc`.

---

## 3. Technology Stack

- **Language**: Python 3.10+
- **Web Framework**: FastAPI, Uvicorn, Starlette
- **Data Validation & Settings**: Pydantic v2, Pydantic-Settings
- **Database & ORM**: PostgreSQL, SQLAlchemy 2.0
- **Database Migrations**: Alembic
- **Embeddings**: Sentence-Transformers (`all-MiniLM-L6-v2`)
- **Vector Search Engine**: FAISS (`faiss-cpu`)
- **Document Loaders**: `pypdf`, `python-docx`
- **Authentication**: `python-jose` (JWT), `passlib` with `bcrypt`
- **Optional LLM Provider**: Groq SDK
- **Automated Testing**: `pytest`, `pytest-asyncio`, Starlette TestClient

---

## 4. Folder Structure

```text
hospital-ai-assistant/
├── README.md                     # Project documentation
├── PROJECT_CONCEPTS.md           # College concept study guide
├── requirements.txt              # Project dependencies
├── .env.example                  # Template environment configuration
├── .gitignore                    # Git ignore file
├── Dockerfile                    # Container build configuration
├── docker-compose.yml            # Multi-container setup (API + PostgreSQL)
├── alembic.ini                   # Alembic migration configuration
│
├── alembic/
│   ├── env.py                    # Alembic migration runner
│   └── versions/                 # Database migration versions
│       └── 001_initial.py        # Baseline schema migration
│
├── app/
│   ├── main.py                   # FastAPI entrypoint, lifespan, /health
│   ├── api/
│   │   ├── deps.py               # Auth & RBAC dependencies
│   │   └── v1/
│   │       ├── router.py         # API router aggregator
│   │       └── endpoints/        # Route handlers
│   │           ├── auth.py       # Login and registration
│   │           ├── users.py      # User management
│   │           ├── departments.py# Department CRUD
│   │           ├── doctors.py    # Doctor CRUD
│   │           ├── patients.py   # Patient CRUD
│   │           ├── appointments.py # Appointment scheduling
│   │           ├── documents.py  # Knowledge uploads & indexing
│   │           ├── chat.py       # RAG chat endpoint & WebSocket
│   │           └── health.py     # Liveness & readiness probes
│   │
│   ├── core/
│   │   ├── config.py             # Application settings & validation
│   │   ├── logging.py            # Centralized logging setup
│   │   ├── security.py           # Password hashing & JWT helpers
│   │   └── exceptions.py         # Exception handling
│   │
│   ├── db/
│   │   ├── base.py               # Declarative Base
│   │   └── session.py            # SessionLocal & get_db generator
│   │
│   ├── models/                   # SQLAlchemy ORM models
│   │   ├── user.py
│   │   ├── department.py
│   │   ├── doctor.py
│   │   ├── patient.py
│   │   ├── appointment.py
│   │   ├── knowledge_document.py
│   │   ├── knowledge_chunk.py
│   │   └── enums.py
│   │
│   ├── schemas/                  # Pydantic request/response models
│   │   ├── auth.py
│   │   ├── user.py
│   │   ├── department.py
│   │   ├── doctor.py
│   │   ├── patient.py
│   │   ├── appointment.py
│   │   ├── document.py
│   │   └── chat.py
│   │
│   ├── crud/                     # Database access layer
│   │   ├── user.py
│   │   ├── department.py
│   │   ├── doctor.py
│   │   ├── patient.py
│   │   ├── appointment.py
│   │   └── knowledge_document.py
│   │
│   ├── services/                 # Core business & AI logic
│   │   ├── document_loader.py    # Text extraction (PDF, DOCX, TXT, MD)
│   │   ├── chunking.py           # Overlapping text chunker
│   │   ├── embedding.py          # Sentence-transformers embedder
│   │   ├── vector_store.py       # FAISS vector store
│   │   ├── retriever.py          # Query embedding & top-k retrieval
│   │   ├── prompt_builder.py     # Grounded context prompt assembler
│   │   ├── rag_service.py        # End-to-end RAG orchestrator
│   │   ├── chat_service.py       # Chat controller
│   │   ├── emergency_guard.py    # Acute emergency detection
│   │   └── medical_guard.py      # Non-diagnostic disclaimers
│   │
│   ├── llm/                      # LLM Providers
│   │   ├── base.py               # Provider abstract interface
│   │   ├── factory.py            # Provider selection factory
│   │   ├── retrieval_only.py     # Extractive offline provider
│   │   └── groq_provider.py      # Groq Cloud LLM provider
│   │
│   └── static/                   # Web dashboard interface
│       ├── index.html            # Main single-page application shell
│       ├── app.js                # Shell client controller
│       └── styles.css            # Styling and theme
│
├── data/
│   ├── knowledge_base/           # Ingested hospital knowledge documents
│   ├── storage/                  # Uploaded files
│   └── vector_index/             # Persisted index.faiss & metadata.json
│
├── scripts/
│   ├── ingest_knowledge_base.py  # Document ingestion & FAISS indexer
│   ├── create_admin.py           # Initial admin bootstrapper
│   └── check_local_setup.py      # Environment & dependency validator
│
├── sample_data/
│   └── knowledge/
│       ├── hospital_services.txt # Demo hospital departments & OPD
│       ├── visiting_hours.txt    # Demo visiting policies & ICU rules
│       └── appointment_policy.txt# Demo booking & cancellation rules
│
└── tests/
    ├── conftest.py               # Pytest fixtures and mock embeddings
    ├── unit/
    │   ├── test_chunking.py      # Text chunking unit tests
    │   ├── test_emergency_guard.py # Emergency detection tests
    │   └── test_security.py      # Hashing and JWT verification tests
    │
    └── integration/
        ├── test_auth.py          # Register, login, protected route tests
        ├── test_departments.py   # Department CRUD tests
        ├── test_doctors.py       # Doctor CRUD tests
        ├── test_patients.py      # Patient CRUD tests
        ├── test_appointments.py  # Appointment CRUD & conflict tests
        └── test_chat.py          # Grounded RAG & emergency chat tests
```

---

## 5. Prerequisites

- **Python**: Version 3.10 or higher
- **PostgreSQL**: Version 14 or higher (or Docker)
- **Git**

---

## 6. Local Installation

### 1. Clone Repository & Open Directory
```bash
git clone <repository_url>
cd hospital-ai-assistant
```

### 2. Create and Activate Virtual Environment
**Windows (PowerShell):**
```powershell
python -m venv .venv
.\.venv\Scripts\activate
```

**Linux / macOS:**
```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

---

## 7. Environment Configuration

Copy the sample environment file to create `.env`:
```bash
cp .env.example .env
```
*(On Windows: `copy .env.example .env`)*

### Key Variables Explained:

| Variable | Default Value | Description |
| :--- | :--- | :--- |
| `APP_NAME` | `Hospital AI Assistant` | Name of the application |
| `DATABASE_URL` | `postgresql+psycopg2://postgres:postgres@localhost:5432/hospital_ai` | SQLAlchemy connection string |
| `JWT_SECRET_KEY` | *(Random Secret)* | Secret key used to sign JWT tokens |
| `ACCESS_TOKEN_EXPIRE_MINUTES`| `60` | Token validity period in minutes |
| `LLM_PROVIDER` | `retrieval_only` | `retrieval_only` (default, no API key required) or `groq` |
| `GROQ_API_KEY` | `""` | Optional Groq API key for LLM generation |
| `ADMIN_EMAIL` | `admin@hospital.com` | Email for initial administrator account |
| `ADMIN_PASSWORD` | `change-this-password` | Password for initial administrator account |
| `VECTOR_INDEX_PATH` | `data/vector_index/index.faiss`| File path for persisted FAISS index |
| `VECTOR_METADATA_PATH`| `data/vector_index/metadata.json`| File path for vector chunk metadata |

---

## 8. Database Setup & Migrations

Ensure your local PostgreSQL server is running. Create a database named `hospital_ai`:
```sql
CREATE DATABASE hospital_ai;
```

Run database migrations to generate all tables:
```bash
alembic upgrade head
```

To create new migrations in the future:
```bash
alembic revision --autogenerate -m "describe changes"
```

---

## 9. Creating the Admin User

Bootstrap the default administrator account:
```bash
python scripts/create_admin.py
```

---

## 10. Knowledge Base Ingestion

Process the sample hospital documents and build the FAISS vector index:
```bash
python scripts/ingest_knowledge_base.py
```

Expected output:
```text
Documents processed: 3
Chunks created: 27
Vector index updated successfully.
```

---

## 11. Starting the Server

Launch the development server:
```bash
uvicorn app.main:app --reload
```

- **API Root / Dashboard**: `http://127.0.0.1:8000/` (redirects directly to `/ui/index.html`)
- **Health Check**: `http://127.0.0.1:8000/health`
- **Swagger Documentation**: `http://127.0.0.1:8000/docs`

---

## 12. Classroom Demonstration Steps

1. **Check System Setup**:
   ```bash
   python scripts/check_local_setup.py
   ```
2. **Health Check**:
   Open `http://127.0.0.1:8000/health` (Returns `{"status": "ok"}`).
3. **Register a User**:
   In Swagger (`/docs`), call `POST /api/v1/auth/register` with email, password, and full name.
4. **Log In & Authorize**:
   Call `POST /api/v1/auth/login`. Copy the returned `access_token` and click Swagger's **Authorize** button (or paste `Bearer <token>` in headers).
5. **Create Department**:
   Call `POST /api/v1/departments` with name `Cardiology`.
6. **Create Doctor**:
   Call `POST /api/v1/doctors` assigning the doctor to the department.
7. **Create Patient**:
   Call `POST /api/v1/patients` with name and date of birth.
8. **Book Appointment**:
   Call `POST /api/v1/appointments` linking patient and doctor with date and time.
9. **Duplicate Booking Check**:
   Attempt to book the same doctor at the same date/time. Notice the `409 Conflict` response.
10. **Ask a Grounded Question**:
    Call `POST /api/v1/chat` (or use the web dashboard at `/`):
    ```json
    {
      "question": "What are the visiting hours for the ICU?"
    }
    ```
    Observe the grounded answer and cited document chunk.
11. **Ask an Unrelated Question**:
    ```json
    {
      "question": "Who won the World Cup?"
    }
    ```
    Notice the assistant states the information is not in the hospital knowledge base.
12. **Ask an Emergency Question**:
    ```json
    {
      "question": "I am having severe chest pain. What should I do?"
    }
    ```
    Observe the immediate emergency safety response.

---

## 13. Running Automated Tests

Run the test suite with pytest:
```bash
pytest -q
```
All 49 unit and integration tests run in-memory without affecting your production database.

---

## 14. Troubleshooting

- **PostgreSQL Connection Refused**:
  Check if PostgreSQL service is running and verify `DATABASE_URL` credentials in `.env`.
- **ModuleNotFoundError: No module named 'faiss'**:
  Install CPU FAISS via `pip install faiss-cpu`.
- **FAISS Index Missing**:
  Run `python scripts/ingest_knowledge_base.py` to create `index.faiss`.
- **Invalid Email in Registration**:
  Use standard domains (e.g. `user@hospital.com`) as email-validator enforces RFC-compliant TLDs.
