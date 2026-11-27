# MediCore — Hospital, Appointment & Patient Care Management Platform

MediCore is an enterprise-grade, full-stack hospital management and clinical care coordination platform built with **Next.js (TypeScript)**, **Express.js (Node.js)**, **FastAPI (Python 3)**, and **Supabase PostgreSQL**.

---

## 🔒 Mandatory Security Notice: Public Repository Safety

This repository strictly complies with zero-plaintext security guidelines:
- **No secrets, private keys, passwords, or actual database credentials exist in this repository.**
- All environment examples use **placeholders only**.
- Actual credentials are kept solely in local `.env` files that are strictly excluded by `.gitignore`.
- Row Level Security (RLS) is enabled on all PostgreSQL database tables.

---

## 🏛️ System Architecture

```
                    ┌───────────────────────────────────────────┐
                    │       Next.js 14 App Router (Frontend)     │
                    │  - 7 Role Dashboards: Patient, Reception,  │
                    │    Doctor, Nurse, Lab, Billing, Admin     │
                    └─────────────────────┬─────────────────────┘
                                          │ REST / JSON
                                          ▼
                    ┌───────────────────────────────────────────┐
                    │      Node.js / Express Core Backend       │
                    │  - Auth / JWT / RBAC / Rate Limiter       │
                    │  - Clinical APIs & FHIR R4 Interop        │
                    │  - Concurrency Locking & Audit Logging    │
                    └──────────────┬──────────────────┬─────────┘
                                   │                  │
                         Internal  │                  │  Redis
                         REST API  ▼                  ▼  Queues
                    ┌──────────────────────┐    ┌───────────────┐
                    │  Python FastAPI Svc  │    │ Redis Service │
                    │  - Pandas Processing │    └───────────────┘
                    │  - Lab Analytics     │
                    └──────────────┬───────┘
                                   │ Supabase Client
                                   ▼
                    ┌───────────────────────────────────────────┐
                    │         Supabase PostgreSQL DB           │
                    │  - Normalized schema, RLS policies,       │
                    │    Exclusion constraints, Audit Triggers  │
                    └───────────────────────────────────────────┘
```

---

## 👥 7 Role-Based Healthcare Portals

1. **Patient Portal**: Schedule doctor appointments, view diagnostic lab results, review active prescriptions, inspect invoices, and manage privacy consents.
2. **Reception Desk**: Patient registration, appointment check-ins, and doctor schedule lookup.
3. **Doctor Suite**: Active clinical encounters, ICD-10 diagnosis recording, electronic prescriptions, and laboratory test ordering.
4. **Nursing Station**: Patient triage, vital signs entry (Blood Pressure, Heart Rate, SpO2, Temperature), and ward monitoring.
5. **Laboratory Workbench**: Clinical specimen analysis, quantitative result verification, and critical value flagging.
6. **Billing & Finance**: Invoice generation, itemized charges, and payment receipt reconciliation.
7. **Hospital Administration**: Clinical audit log streams, system metrics, and population analytics.

---

## 🚀 Getting Started

### 1. Prerequisites
- **Node.js**: v18+ or v20+
- **Python**: v3.11+
- **Git**

### 2. Environment Configuration
Copy the provided `.env.example` template into a new `.env` file:

```bash
cp .env.example .env
```

Edit `.env` and fill in your Supabase project credentials:
```env
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url_here
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_publishable_anon_key_here
SUPABASE_URL=your_supabase_project_url_here
SUPABASE_SERVICE_ROLE_KEY=your_supabase_secret_service_role_key_here
JWT_SECRET=your_secure_random_jwt_secret_min_32_chars
```

### 3. Setting Up the Dedicated Python Virtual Environment

```powershell
# Windows
cd python-service
python -m venv venv
.\venv\Scripts\activate
pip install -r requirements.txt
cd ..
```

### 4. Running the Backend REST API

```powershell
cd backend
npm install
npm run dev
# Running on http://localhost:5000
```

### 5. Running the Python FastAPI Analytics Microservice

```powershell
cd python-service
.\venv\Scripts\activate
uvicorn app.main:app --port 8000 --reload
# Running on http://localhost:8000
```

### 6. Running the Frontend Application

```powershell
cd frontend
npm install
npm run dev
# Running on http://localhost:3000
```

---

## 🧪 Automated Testing

MediCore includes extensive multi-tier test suites:

```powershell
# 1. Run Node.js API, Security & IDOR Tests
node backend/tests/run_tests.js

# 2. Run Concurrency & FHIR Interoperability Tests
node backend/tests/test_concurrency_and_interoperability.js

# 3. Run Python Data Processing & FastAPI Analytics Tests
cd python-service
.\venv\Scripts\pytest.exe -v
```

---

## 🐳 Docker Deployment

To launch the full containerized environment (Frontend, Backend, FastAPI, Redis):

```bash
docker-compose up --build -d
```

---

## 📄 License & Disclaimer
MediCore uses strictly **synthetic patient data** for development, demonstration, and clinical assessment purposes. Does not generate medical diagnoses or treatment recommendations.
