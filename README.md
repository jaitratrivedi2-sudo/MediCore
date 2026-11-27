# FinCore – Digital Wallet, Ledger & Payment Processing Platform

FinCore is an enterprise-grade digital finance, ledger, and payment processing simulator implementing balanced double-entry bookkeeping, multi-tenant digital wallets, high-precision monetary calculations, real-time risk analytics, and hybrid vector search on **ChromaDB Cloud**.

---

## 1. Architecture Overview
- **Frontend**: Next.js 14 App Router, React 18, TypeScript, Tailwind CSS, Lucide icons.
- **Backend API Gateway**: Node.js 20+, Express, JWT with RBAC, Idempotency middleware, Wallet Concurrency Locking.
- **Python Service**: Python 3.13, FastAPI, Pydantic, pytest, Decimal monetary arithmetic with Banker's Rounding (`ROUND_HALF_EVEN`).
- **Database Layer**: ChromaDB Cloud hosted at `api.trychroma.com` (configured via environment variables), utilizing 11 domain-sharded collections, deterministic record IDs, 16 KiB document chunking, and reciprocal rank fusion hybrid search.

---

## 2. 15 Mandatory Assessments Coverage
1. **Assessment 01 (Frontend)**: Customer and Operations dashboards with live balances, transfer execution, beneficiaries, dispute management, and statement generation.
2. **Assessment 02 (REST APIs)**: REST endpoints for customers, wallets, beneficiaries, transfers, payment intents, refunds, reversals, and disputes.
3. **Assessment 03 (Double-Entry Ledger)**: Chart of accounts, journal entries with balanced `sum(DEBIT) === sum(CREDIT)`, hash chaining, and immutable audit logs.
4. **Assessment 04 (Identity & Auth)**: Password hashing with bcrypt, JWT token sessions, RBAC (Customer vs Operations), and sensitive-action re-authentication (`X-Reauth-Password` / `X-MFA-Code`).
5. **Assessment 05 (Python Financial Calculations)**: Decimal calculations, fee schedules, interest formulas (Actual/365), and Banker's rounding.
6. **Assessment 06 (FastAPI Risk Analytics)**: Velocity checks, sudden amount spikes, high-risk beneficiary scoring, and decision-support risk indicators.
7. **Assessment 07 (Transfer Consistency Engine)**: Concurrency lock per wallet, idempotency keys, and multi-step state machine.
8. **Assessment 08 (Payment & Webhooks)**: HMAC-SHA256 signature verification, replay attack prevention, and reconciliation.
9. **Assessment 09 (Containerization)**: Multi-stage non-root Dockerfiles and Docker Compose configuration.
10. **Assessment 10 (Governance)**: Git structure, migration documentation, and branching conventions.
11. **Assessment 11 (CI/CD)**: GitHub Actions workflow for test automation and build validation.
12. **Assessment 12 (Ledger Testing)**: Automated test suites testing insufficient funds, duplicate transfers, reversals, and balancing.
13. **Assessment 13 (Contracts)**: Postman collection (`postman/FinCore.postman_collection.json`) and API specification.
14. **Assessment 14 (Deployment & Recovery)**: Runbook in `docs/deployment.md` detailing Chroma Cloud backup and recovery.
15. **Assessment 15 & Capstone**: Controlled failure simulation engine demonstrating root-cause investigation for duplicate webhooks, mismatched settlements, and unbalanced ledger attempts.

---

## 3. Quick Start & Installation

### Prerequisites
- Node.js >= 20.x
- Python >= 3.10
- Git

### Step 1: Clone and Environment Setup
```bash
cp .env.example .env
```
Ensure your Chroma Cloud credentials are set in `.env`:
```env
CHROMA_HOST=api.trychroma.com
CHROMA_API_KEY=your_chroma_api_key_here
CHROMA_TENANT=your_tenant_id_here
CHROMA_DATABASE=your_database_name_here
```

### Step 2: Python Virtual Environment
```bash
# Windows:
py -m venv venv
venv\Scripts\activate
pip install -r requirements.txt

# Linux/macOS:
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Step 3: Seed Database in Chroma Cloud
```bash
python database/seed_data.py
```

### Step 4: Install Dependencies & Run Services
```bash
# Terminal 1: Python Microservice (Port 8000)
venv\Scripts\activate
python run_python_service.py
# Or directly via uvicorn:
# uvicorn main:app --app-dir python-service --port 8000 --reload

# Terminal 2: Backend API Gateway (Port 5000)
cd backend
npm install
npm start

# Terminal 3: Frontend Dashboard (Port 3000)
cd frontend
npm install
npm run dev
```

Open your browser at `http://localhost:3000` to access the interactive customer and operations dashboards.

---

## 4. Running Automated Tests

```bash
# Run Python pytest tests:
pytest

# Run Backend Jest & Capstone tests:
npm test --prefix backend
```

---

## 5. Seed Accounts for Testing
Initial synthetic test accounts configured in database seed (`database/seed_data.py`):
- **Customer Account**: `alice@fincore.internal` (Default Wallet: `wal_alice_usd`)
- **Customer Account**: `bob@fincore.internal` (Default Wallet: `wal_bob_usd`)
- **Operations Staff**: `ops@fincore.internal`

> **Security Note:** In production, credentials and sensitive authorization tokens must always be supplied via secure environment variables (`.env`). Default initial test passwords can be customized directly in `database/seed_data.py` prior to running the seed script. Never commit production credentials, secret keys, or passwords to version control.
