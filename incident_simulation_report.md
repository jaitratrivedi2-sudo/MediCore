# MediCore Healthcare Incident Simulation & Remediation Report
**Assessment 15 Implementation Evidence**

This clinical incident simulation drill investigates, reproduces, diagnoses, and patches five distinct operational failures within the MediCore ecosystem.

---

### Incident 01: Unauthorized Cross-Patient Record Access (IDOR Vulnerability)
- **Reproduction**: An authenticated patient session (John Doe, MRN-2026-0001) attempted to directly query `/api/v1/patients/55555555-5555-5555-5555-555555555502` belonging to Jane Roe.
- **Log Inspection**:
  ```json
  {"level":"ERROR","message":"Access denied: You are not authorized to view another patient clinical record","url":"/api/v1/patients/55555555-5555-5555-5555-555555555502","statusCode":403}
  ```
- **Root Cause**: Reliance on client-side routing guards without record-level authorization checks in the repository layer.
- **Remediation & Fix**: Implemented an explicit user ownership match check inside `patient.service.js` and Row Level Security (RLS) policies on `public.patients`.
- **Regression Test**: Automated test `backend/tests/run_tests.js` (Test 6) asserts HTTP 403 Forbidden on foreign record access attempts.
- **Prevention Strategy**: Zero-trust authorization middleware enforcing tenant/profile matching at the API gateway and database RLS.

---

### Incident 02: Appointment Booking Concurrency Collision (Double Booking)
- **Reproduction**: Two asynchronous requests submitted identical 30-minute booking requests for Dr. Marcus Smith at `2026-12-05T14:00:00.000Z` simultaneously.
- **Log Inspection**:
  ```json
  {"level":"ERROR","message":"Doctor booking collision: Doctor is already committed to an appointment at 2026-12-05T14:00:00.000Z","statusCode":409}
  ```
- **Root Cause**: Asynchronous reads checking doctor schedule overlap before writes could commit without transactional serialization.
- **Remediation & Fix**: Added doctor-scoped asynchronous mutual exclusion locks in `database.js` and PostgreSQL range exclusion constraints (`btree_gist`).
- **Regression Test**: `backend/tests/test_concurrency_and_interoperability.js` asserts exactly one `fulfilled` and one `rejected` (409 Conflict) response.
- **Prevention Strategy**: Strict database-level isolation level and scheduling mutex locks.

---

### Incident 03: Notification Delivery Storm & Duplication
- **Reproduction**: Network retry glitch fired five identical appointment confirmation dispatches within 200 milliseconds.
- **Log Inspection**:
  ```json
  {"level":"INFO","message":"[Notification Sandbox] Duplicate dispatch ignored by Idempotency Key: appointment_confirm_66666666_001"}
  ```
- **Root Cause**: Notification dispatcher lacked client-side idempotency keys and state tracking.
- **Remediation & Fix**: Built `notification.service.js` with an in-memory & database `idempotencyStore`, returning cached status on re-dispatch.
- **Regression Test**: `backend/tests/test_concurrency_and_interoperability.js` (Test 2) asserts `DUPLICATE_IGNORED` status.
- **Prevention Strategy**: Enforce UUID idempotency keys on all external webhooks, email triggers, and SMS dispatches.

---

### Incident 04: Primary Database Interruption & Graceful Degradation
- **Reproduction**: Simulated temporary network timeout or PostgREST 403 authorization lock on cloud Supabase endpoints.
- **Log Inspection**:
  ```json
  {"level":"WARN","message":"[Supabase Config] PostgREST schema cache or table unavailable. Switching to high-resilience local clinical data store."}
  ```
- **Root Cause**: Monolithic database reliance blocking clinical read/write operations during maintenance windows.
- **Remediation & Fix**: Implemented the dual-layer `DatabaseRepository` architecture which seamlessly falls back to local clinical operational storage without dropping requests.
- **Regression Test**: Backend and frontend health endpoints continue serving active sessions with 200 OK.
- **Prevention Strategy**: Circuit breakers and local read replicas for critical inpatient vital displays.

---

### Incident 05: Malformed Laboratory Result Unit Inversion
- **Reproduction**: Ingested synthetic laboratory test record specifying `Serum Creatinine: 1.1 kg/L` instead of `mg/dL`.
- **Log Inspection**:
  ```json
  {"test_name":"Serum Creatinine","flag":"FLAG_UNIT_ERROR","validation_status":"UNIT_MISMATCH"}
  ```
- **Root Cause**: Direct ingestion of unstructured third-party laboratory files without unit and LOINC metadata validation.
- **Remediation & Fix**: Integrated `LabDataProcessor` in Python Pandas which cross-validates measurement units against `STANDARD_REFERENCE_RANGES`.
- **Regression Test**: `python-service/tests/test_lab_processing.py` asserts `FLAG_UNIT_ERROR` on unstandardized measurement units.
- **Prevention Strategy**: Mandatory schema normalization and LOINC-aligned automated unit sanitizers.
