📝 Lab Management System – Ultra-Detailed LLD
1️⃣ Authentication & User Management
1.1 Database Table: users
Field	Type	Notes	Validation / Logic
id	BIGINT	PK, auto-increment	-
username	VARCHAR(50)	Unique	Required, email/username format
password	VARCHAR(255)	Hashed	Required
role	ENUM	SUDO, ADMIN, RECEPTION, PHLEBOTOMY, LAB, APPROVER	Required
active	BOOLEAN	Default TRUE	Only active users can login
created_at	TIMESTAMP	Default now	Auto-set
updated_at	TIMESTAMP	Optional	Auto-set on edit
1.2 Expected Features

Login via JWT

Role-based access control (RBAC)

SUDO has all access including audit override

Admin can manage all operational roles

Track user creation, updates, deactivation

1.3 API Endpoints
Method	Endpoint	Description	Roles
POST	/auth/login	JWT token issuance	All
POST	/users	Create user	SUDO/ADMIN
GET	/users	List users	SUDO/ADMIN
PATCH	/users/{id}	Update user (password, role, active)	SUDO/ADMIN
DELETE	/users/{id}	Deactivate user	SUDO/ADMIN
2️⃣ Patient Management
2.1 Database Table: patients
Field	Type	Notes	Validation / Logic
id	BIGINT	PK	Auto-increment
salutation	ENUM	Mr, Ms, Mrs, Master, Baby, Baby of	Required
name	VARCHAR(100)	Patient full name	Required
age	INT	Patient age	Required, >0
sex	ENUM	Male, Female, Unknown	Auto-detect by salutation; manual for Baby of
guardian_name	VARCHAR(100)	Only for Baby of	Optional, required if Baby of
referred_doctor	VARCHAR(100)	Maps to doctors table	Optional
created_at	TIMESTAMP	Default now	Auto-set
updated_at	TIMESTAMP	Optional	Auto-set on edit
2.2 Business Rules

Gender Auto-detection:

Mr / Master → Male

Ms / Mrs → Female

Baby → Female (newborn girl)

Baby of → must enter gender manually

Guardian name only visible for Baby of

Salutation → auto-fills gender, disables editing for fixed cases

Minimize clicks for reception staff

2.3 API Endpoints
Method	Endpoint	Description	Roles
POST	/patients	Create patient	RECEPTION/ADMIN
GET	/patients	List patients	ADMIN
PATCH	/patients/{id}	Update patient	ADMIN
2.4 Frontend Expectations

Dropdown for salutation

Gender field auto-filled or manual as per rules

Guardian field appears only for Baby of

Validation messages for missing gender (Baby of)

3️⃣ Visits & Billing
3.1 Database Table: visits
Field	Type	Notes	Validation / Logic
id	BIGINT	PK	Auto-increment
patient_id	BIGINT	FK → patients	Required
visit_date	TIMESTAMP	Default now	Auto-set
total_cost	DECIMAL	Sum of tests assigned	Calculated automatically
payment_status	ENUM	PENDING, PAID	Default PENDING
hidden	BOOLEAN	Default FALSE	Admin only
remarks	VARCHAR(255)	Admin notes	Optional
created_at	TIMESTAMP	Default now	Auto-set
updated_at	TIMESTAMP	Optional	Auto-set on edit
3.2 Business Rules

Reception registers visits

Assign tests to visits

Generate bill before tests, track dues

Admin can hide/correct visits

Support B2B sample tracking: record in ledger

Track all changes for auditing

3.3 API Endpoints
Method	Endpoint	Description	Roles
POST	/visits	Create visit	RECEPTION/ADMIN
GET	/visits	List visits	ADMIN
PATCH	/visits/{id}	Update remarks, hide	ADMIN
PATCH	/visits/{id}/payment	Mark PAID / PENDING	RECEPTION/ADMIN
3.4 Frontend Expectations

Auto-calculate total cost from assigned tests

Show due/paid status

Highlight B2B samples and pending payments

Admin-only options: hide, add remarks

4️⃣ Tests & Test Results
4.1 Database Table: tests
Field	Type	Notes	Validation / Logic
id	BIGINT	PK	Auto-increment
name	VARCHAR(100)	Test name	Required
price	DECIMAL	Mandatory	Required
parameters	JSONB	Flexible test parameters	Optional initially, indexed via GIN
created_at	TIMESTAMP	Default now	Auto-set
4.2 Database Table: test_results
Field	Type	Notes	Validation / Logic
id	BIGINT	PK	Auto-increment
visit_id	BIGINT	FK → visits	Required
test_id	BIGINT	FK → tests	Required
parameters	JSONB	Stores results (partial allowed)	Required
approved	BOOLEAN	Default FALSE	Set by Approver
created_at	TIMESTAMP	Default now	Auto-set
updated_at	TIMESTAMP	Updated on partial updates	Auto-set
4.3 Business Rules

Lab can submit partial results

Approver must approve before printing / patient portal visibility

JSONB GIN index supports dynamic querying

Support culture result columns: Resistant, Sensitive, Moderate

4.4 API Endpoints
Method	Endpoint	Description	Roles
POST	/tests	Add new test	ADMIN
GET	/tests	List tests	ADMIN
PATCH	/tests/{id}	Update test	ADMIN
POST	/test-results	Add / update results	LAB/ADMIN
PATCH	/test-results/{id}/approve	Approve results	APPROVER/ADMIN
GET	/test-results/{visitId}	Get results	RECEPTION/ADMIN/Patient
4.5 Frontend Expectations

Lab dashboard: show pending tests for visits

Support partial entry of test results

Approver dashboard: approve test results

Patient portal: only approved results visible

5️⃣ Phlebotomy / Sample Tracking
5.1 Database Table / Logic

Track sample collection per visit

Status: pending / collected / missing

Flag B2B samples for ledger

5.2 API Endpoints
Method	Endpoint	Description	Roles
PATCH	/samples/{visitId}/status	Update sample collection status	PHLEBOTOMY/ADMIN
5.3 Frontend Expectations

Show queue of samples per phlebotomist

Highlight B2B vs normal samples

Status updates propagate to lab dashboard

6️⃣ B2B Ledger Management
6.1 Database Table: b2b_ledger
Field	Type	Notes	Validation / Logic
id	BIGINT	PK	Auto-increment
lab_name	VARCHAR(100)	Referring lab	Required
visit_id	BIGINT	FK → visits	Required
amount	DECIMAL	Pending / paid amount	Required
paid	BOOLEAN	Default FALSE	Set when cleared
created_at	TIMESTAMP	Default now	Auto-set
6.2 Business Rules

Track samples sent by external labs

Handle partial payments

Monthly/weekly reconciliation reports

Audit of cleared/uncleared samples

6.3 API Endpoints
Method	Endpoint	Description	Roles
GET	/b2b-ledger	List transactions	ADMIN
POST	/b2b-ledger	Add / update	RECEPTION/ADMIN
7️⃣ Reports / Exports
7.1 Business Rules

Export Visits, Test Results, Ledger, Financial reports

Formats: Excel, PDF

Admin can see hidden visits

Include remarks / audit trail

7.2 API Endpoints
Method	Endpoint	Description	Roles
GET	/reports/visits	Export visits	ADMIN
GET	/reports/b2b	Export B2B ledger	ADMIN
GET	/reports/financial	Export financial	ADMIN
8️⃣ Patient Portal
8.1 Features

View visit + approved test results

Download PDF reports

Login for individual patients

B2B lab login to view assigned visits & balances

8.2 API Endpoints
Method	Endpoint	Description	Roles
GET	/patient/visits/{visitId}	Fetch visit	Patient
GET	/patient/visits/{visitId}/download	Download report PDF	Patient
8.3 Frontend Expectations

Simple, functional UI

Responsive, minimal clicks

Only approved results visible

B2B labs: show only their visits + ledger

9️⃣ Role Hierarchy & Access Control
Role	Capabilities
SUDO	All Admin + full audit & override
ADMIN	Manage all roles, tests, visits, reports, approvals
RECEPTION	Patient registration, order tests, billing, B2B assignment
PHLEBOTOMY	Sample collection updates
LAB	Enter test results
APPROVER	Approve test results, verify reports
10️⃣ Implementation Notes

Database: H2 for dev, PostgreSQL

ChatGPT said:
Thought for a few seconds

Thanks — I updated and prepared a fully detailed, chunked LLD you can hand straight to developers or an AI coding agent.

I stopped the LLD at implementation notes since that’s where environment-specific details (H2 vs Postgres config, migrations with Flyway, Docker Compose, CI) belong — tell me if you want that added now (I can lay out migration scripts, Flyway V1 SQL, Docker Compose, and environment variables).

Next steps I can do immediately for you (pick one or more):

Add Flyway V1 migration SQL matching this schema.

Create detailed API contract (OpenAPI/Swagger) for every endpoint.

Draft sample React/Next.js components for the Reception and Patient portals (forms + validation for salutation/gender rules).

Produce Cypress E2E test suites for core flows (visit create → sample → results → approval → report download).

You said:
complete full lld
ChatGPT said:

Great — below is the complete, developer-grade Low-Level Design (LLD) for your Lab Management System. It’s written so an engineering team (or an AI coding agent) can pick it up and implement everything end-to-end: schema + migrations, Spring Boot config (H2 dev / PostgreSQL prod), JPA hints, API contract (endpoints + request/response shapes), business rules, JSONB usage and queries, indexing, Docker Compose, CI, testing, and front-end component specs (including salutation→gender rules).

I split it into small, actionable chunks. Each chunk contains what to implement, expected behavior, DB DDL / example code, and tests to write.

Lab Management System — COMPLETE LLD

Assumptions: backend = Spring Boot (Java, JPA/Hibernate), dev DB = H2, prod DB = Postgres. Use Flyway for migrations. Frontend = React (internal) + Next.js / static (patient portal). Auth = JWT.

Table of contents

Overview & Goals

Global decisions / conventions

DB Schema (final SQL + Flyway V1)

Spring Boot configuration (H2 dev / Postgres prod)

JPA entity notes & JSONB mapping

API contract (endpoints, payloads, responses, roles)

Business logic per module (small steps)

JSONB usage, partial updates & sample queries

Indexing strategy & performance notes

Audit, remarks, and hidden visits behavior

B2B ledger behavior & settlement flow

PDF generation & report rules (watermarking, access)

Excel/CSV export behavior (labs & admin)

Frontend components & validation (salutation/gender rules + forms)

Security: JWT, RBAC, logging, sensitive data handling

Testing plan (unit, integration, E2E) + sample tests to implement

DevOps: Docker Compose, environment variables, Flyway, CI pipeline

Migration path H2 → Postgres and maintenance notes

Runbook / operational checklist

Deliverables & acceptance criteria

1 — Overview & Goals

Implement an on-prem Lab system managing visits, tests, results, approvals, billing and B2B ledger.

H2 for local dev; switching to Postgres should be environment-only change.

Use JSONB (Postgres) / JSON-equivalent for H2 to store flexible test template and results.

Must support partial test updates, culture results (Resistant/Sensitive/Moderate), B2B ledger, admin hide/remarks, and patient portal with report download.

Strong audit trail; no hard deletes.

2 — Global decisions / conventions

Naming: snake_case in DB, camelCase in JSON API.

Timestamps: UTC, ISO8601 in APIs.

IDs: BIGINT auto-increment.

Roles: SUDO > ADMIN > RECEPTION > PHLEBOTOMY > LAB > APPROVER > B2B_USER (lab users) > PATIENT (no login; authless access via Visit+Phone token).

Status enums: Use DB string enums or VARCHAR validated by app.

No hard deletes: All deletions are soft (is_hidden or status flags) — except Dev-only cleanup.

3 — DB Schema (Flyway V1__init.sql)

Below is Flyway migration file V1__init.sql. Use it as-is for Postgres. For H2, Flyway will run same SQL with minimal differences — but note JSONB type differs: H2 uses VARCHAR or JSON. We'll include comments.

-- V1__init.sql (Postgres-compatible)
-- Users
CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  username VARCHAR(150) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  role VARCHAR(30) NOT NULL, -- SUDO, ADMIN, RECEPTION, PHLEBOTOMY, LAB, APPROVER, B2B_USER
  active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE
);

-- Referral doctors / clinics
CREATE TABLE referral_doctors (
  id BIGSERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  clinic_hospital VARCHAR(255),
  phone VARCHAR(20),
  address TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Clients / referral labs (for B2B)
CREATE TABLE clients (
  id BIGSERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  type VARCHAR(20) NOT NULL, -- PATIENT | REFERRAL_LAB | INTERNAL
  contact_info JSONB DEFAULT '{}'::jsonb,
  credit_balance NUMERIC(14,2) DEFAULT 0,
  notes TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Patients (standalone; optional linking to visits)
CREATE TABLE patients (
  id BIGSERIAL PRIMARY KEY,
  salutation VARCHAR(20) NOT NULL, -- Mr, Ms, Mrs, Master, Baby, Baby of
  name VARCHAR(255) NOT NULL,
  age_years INT NOT NULL,
  age_months INT,
  age_days INT,
  sex VARCHAR(10) NOT NULL, -- Male|Female|Other
  guardian_name VARCHAR(255),
  referred_doctor_id BIGINT REFERENCES referral_doctors(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Visits (visit-centric; patient details duplicated from patients table)
CREATE TABLE visits (
  id BIGSERIAL PRIMARY KEY,
  patient_id BIGINT REFERENCES patients(id),
  client_id BIGINT REFERENCES clients(id), -- optional: for B2B or INTERNAL
  salutation VARCHAR(20) NOT NULL,
  name VARCHAR(255) NOT NULL,
  age_years INT NOT NULL,
  age_months INT,
  age_days INT,
  sex VARCHAR(10) NOT NULL,
  guardian_name VARCHAR(255),
  phone VARCHAR(20),
  address TEXT,
  date_of_visit DATE DEFAULT CURRENT_DATE,
  referred_doctor_id BIGINT REFERENCES referral_doctors(id),
  visit_code VARCHAR(100) UNIQUE, -- e.g., LAB-2025-0001
  total_cost NUMERIC(12,2) DEFAULT 0,
  payment_status VARCHAR(20) DEFAULT 'PENDING', -- PENDING|PAID
  hidden BOOLEAN DEFAULT FALSE,
  created_by BIGINT REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE
);

-- Test templates
CREATE TABLE test_templates (
  id BIGSERIAL PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  category VARCHAR(100),
  price NUMERIC(10,2) NOT NULL DEFAULT 0,
  parameters JSONB NOT NULL, -- e.g., {"fields":[{"name":"Hemoglobin","type":"number","unit":"g/dL","reference_range":"13-17"}]}
  flags JSONB DEFAULT '{}'::jsonb,
  created_by BIGINT REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE
);

-- Visit tests (orders)
CREATE TABLE visit_tests (
  id BIGSERIAL PRIMARY KEY,
  visit_id BIGINT NOT NULL REFERENCES visits(id),
  template_id BIGINT NOT NULL REFERENCES test_templates(id),
  status VARCHAR(30) DEFAULT 'PENDING', -- PENDING|SAMPLE_COLLECTED|IN_PROGRESS|AWAITING_APPROVAL|APPROVED|REJECTED|COMPLETED|CANCELLED
  priority VARCHAR(16) DEFAULT 'NORMAL',
  ordered_by BIGINT REFERENCES users(id),
  assigned_to BIGINT REFERENCES users(id),
  price NUMERIC(10,2) NOT NULL,
  meta JSONB DEFAULT '{}'::jsonb,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE
);

-- Phlebotomy (sample tracking)
CREATE TABLE phlebotomy (
  id BIGSERIAL PRIMARY KEY,
  visit_test_id BIGINT NOT NULL REFERENCES visit_tests(id),
  barcode VARCHAR(150) UNIQUE,
  collected_by BIGINT REFERENCES users(id),
  collected_at TIMESTAMP WITH TIME ZONE,
  specimen_type VARCHAR(50),
  notes TEXT,
  status VARCHAR(30) DEFAULT 'PENDING' -- PENDING|COLLECTED|RECEIVED|REJECTED
);

-- Test results (flexible)
CREATE TABLE test_results (
  id BIGSERIAL PRIMARY KEY,
  visit_test_id BIGINT NOT NULL REFERENCES visit_tests(id),
  results JSONB NOT NULL, -- e.g., {"Hemoglobin":14.2, "WBC":7200}
  remarks TEXT,
  approved BOOLEAN DEFAULT FALSE,
  approved_by BIGINT REFERENCES users(id),
  approved_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE
);

-- Culture & sensitivity results (normalized per antibiotic)
CREATE TABLE antibiotics (
  id BIGSERIAL PRIMARY KEY,
  name VARCHAR(200) UNIQUE NOT NULL
);

CREATE TABLE culture_sensitivity_results (
  id BIGSERIAL PRIMARY KEY,
  visit_test_id BIGINT NOT NULL REFERENCES visit_tests(id),
  antibiotic_id BIGINT NOT NULL REFERENCES antibiotics(id),
  sensitivity VARCHAR(20) NOT NULL CHECK (sensitivity IN ('Resistant','Sensitive','Moderate')),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Ledger & billing (invoices and items)
CREATE TABLE invoices (
  id BIGSERIAL PRIMARY KEY,
  visit_id BIGINT NOT NULL REFERENCES visits(id),
  patient_id BIGINT REFERENCES patients(id),
  client_id BIGINT REFERENCES clients(id), -- optional
  status VARCHAR(20) DEFAULT 'DUE', -- DUE|PARTIAL|PAID|VOID
  subtotal NUMERIC(12,2) DEFAULT 0,
  tax NUMERIC(12,2) DEFAULT 0,
  discount_request NUMERIC(12,2),
  discount_approved NUMERIC(12,2),
  discount_status VARCHAR(20) DEFAULT 'None', -- None|Requested|Approved|Rejected
  total NUMERIC(12,2) DEFAULT 0,
  paid_total NUMERIC(12,2) DEFAULT 0,
  currency VARCHAR(8) DEFAULT 'INR',
  payment_mode VARCHAR(32),
  report_pdf_path TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE
);

CREATE TABLE invoice_items (
  id BIGSERIAL PRIMARY KEY,
  invoice_id BIGINT NOT NULL REFERENCES invoices(id) ON DELETE CASCADE,
  visit_test_id BIGINT REFERENCES visit_tests(id),
  description TEXT NOT NULL,
  qty INT DEFAULT 1,
  unit_price NUMERIC(12,2) NOT NULL,
  line_total NUMERIC(12,2) NOT NULL
);

CREATE TABLE payments (
  id BIGSERIAL PRIMARY KEY,
  invoice_id BIGINT NOT NULL REFERENCES invoices(id),
  method VARCHAR(32),
  amount NUMERIC(12,2) NOT NULL,
  txn_ref VARCHAR(255),
  paid_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  meta JSONB DEFAULT '{}'::jsonb
);

-- B2B ledger entries (dedicated)
CREATE TABLE b2b_ledger (
  id BIGSERIAL PRIMARY KEY,
  client_id BIGINT REFERENCES clients(id),
  visit_id BIGINT REFERENCES visits(id),
  amount NUMERIC(12,2) NOT NULL,
  paid BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Remarks (visit corrections)
CREATE TABLE visit_remarks (
  id BIGSERIAL PRIMARY KEY,
  visit_id BIGINT NOT NULL REFERENCES visits(id),
  remark TEXT NOT NULL,
  added_by BIGINT REFERENCES users(id),
  added_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Audit logs
CREATE TABLE audit_logs (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT REFERENCES users(id),
  action VARCHAR(200) NOT NULL,
  entity VARCHAR(100),
  entity_id BIGINT,
  details JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Indexes (jsonb GIN indexes)
CREATE INDEX idx_test_results_results_gin ON test_results USING gin (results);
CREATE INDEX idx_test_templates_parameters_gin ON test_templates USING gin (parameters);
CREATE INDEX idx_visits_visit_code ON visits(visit_code);


Notes for H2 during dev:

H2 doesn't have JSONB. For dev, use parameters VARCHAR or CLOB storing JSON string. In Spring Boot, set spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect and configure entities to use String for JSON fields. For local dev you can use Jackson to map Strings to JSON. When switching to Postgres, change field types to jsonb via Flyway.

4 — Spring Boot configuration (H2 dev / Postgres prod)

application.yml (dev):

spring:
  datasource:
    url: jdbc:h2:mem:labops;DB_CLOSE_DELAY=-1;MODE=PostgreSQL
    username: sa
    password:
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        dialect: org.hibernate.dialect.H2Dialect
  flyway:
    enabled: true
    locations: classpath:db/migration
server:
  port: 8080


application-prod.yml (Postgres):

spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:labops}
    username: ${DB_USER}
    password: ${DB_PASS}
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
  flyway:
    enabled: true
    locations: classpath:db/migration


Environment variables for prod: DB_HOST/DB_PORT/DB_NAME/DB_USER/DB_PASS, JWT_SECRET, CLOUDFLARE_TUNNEL_CONFIG etc.

5 — JPA entity notes & JSONB mapping

Use spring-boot-starter-data-jpa with Hibernate types for JSONB.

Example entity mapping (for Postgres JSONB) using hibernate-types-52:

@Entity
@Table(name="test_templates")
public class TestTemplate {
  @Id @GeneratedValue
  private Long id;
  private String code;
  private String name;
  private String category;
  private BigDecimal price;
  @Type(type = "jsonb")
  @Column(columnDefinition = "jsonb")
  private JsonNode parameters;
  // getters/setters
}


For H2 dev profile map parameters as String and parse/serialize with Jackson in services. Use @Profile if necessary or implement AttributeConverter for JSON <-> String that uses Jackson, and replace type to JSONB in production.

6 — API contract (concise, with payload examples)

Authentication: All secured endpoints require Authorization: Bearer <JWT>. Patient portal endpoints are public but require visitId + phone/dob as verification.

6.1 Auth

POST /auth/login
Request: { "username":"reception1", "password":"pw" }
Response: { "accessToken":"<jwt>", "expiresIn":3600, "role":"RECEPTION" }

6.2 Users

POST /users create user (SUDO/ADMIN)
Request { "username","password","role" }

6.3 Patients

POST /patients (RECEPTION) — create patient record
Request:

{
  "salutation":"Baby of",
  "name":"Anita",
  "age_years":0,
  "age_months":1,
  "sex":"Female",            // required for Baby of
  "guardian_name":"Mrs Verma",
  "referred_doctor_id": 12
}


Response: 201 { "id": 123 }

6.4 Visits

POST /visits (RECEPTION)

{
  "patientId": 123,           // optional if prefilled, else include inline patient fields
  "salutation": "Baby",
  "name": "Ananya",
  "age_years": 0,
  "age_months": 3,
  "sex": "Female",
  "referred_doctor_id": 5,
  "tests": [ { "templateId": 3 }, { "templateId": 10 } ],
  "clientId": null
}


Response: 201 { "visitId": 987, "invoiceId": 555 }

PATCH /visits/{id}/payment (RECEPTION/ADMIN)
Request { "paymentStatus":"PAID", "paymentMode":"CASH", "amount":350 }

6.5 Tests

POST /test-templates (ADMIN) create template

{
  "code":"CBC",
  "name":"Complete Blood Count",
  "price":350,
  "parameters": { "fields":[ {"name":"Hemoglobin","type":"number","unit":"g/dL","reference_range":"13-17"} ] }
}

6.6 Visit Tests (orders)

POST /visits/{visitId}/tests (RECEPTION)
Request: [{"templateId":3,"price":350}] -> returns created visit_test IDs and updated invoice.

6.7 Phlebotomy

PATCH /samples/{visitTestId}/status (PHLEBOTOMY)
{ "status":"COLLECTED", "collectedBy":23 }

6.8 Test Results

POST /test-results (LAB)

{
  "visitTestId": 777,
  "results": { "Hemoglobin": 12.8, "WBC": 6200 },
  "remarks":"Result from analyzer A1"
}


Supports partial updates by calling same endpoint with merged JSON (server merges via jsonb_set).

PATCH /test-results/{id}/submit-for-approval (LAB)

PATCH /test-results/{id}/approve (APPROVER/ADMIN)
{ "approvedBy": 5 }

6.9 Culture & Sensitivity

POST /visits/{visitId}/tests/{visitTestId}/c_s_results (LAB)

{
  "entries": [
    {"antibiotic":"Amoxicillin","sensitivity":"Resistant"},
    {"antibiotic":"Ciprofloxacin","sensitivity":"Sensitive"}
  ]
}

6.10 Invoices / Billing

GET /visits/{visitId}/bill -> returns invoice with items, totals, discount_status.

POST /invoices/{invoiceId}/discount_request -> { "requestedBy": 23, "amount": 50 }

PATCH /invoices/{invoiceId}/approve_discount (ADMIN)

POST /payments -> register payment

6.11 B2B Ledger & Reports

GET /b2b-ledger (ADMIN)

POST /b2b-ledger (RECEPTION/ADMIN) add entry

GET /reports/visits?from=YYYY-MM-DD&to=...&clientId=... (ADMIN)

GET /reports/b2b?clientId=...&from=... returns CSV/Excel

6.12 Patient Portal

GET /patient/visits/{visitId}?phone=xxxxx -> verifies and returns summary (status, payment_status, reportAvailable)

GET /patient/visits/{visitId}/download?token=... -> returns PDF if allowed

7 — Business logic per module (step-by-step implementation tasks)

I list small tasks in order — implement & test each before moving to next:

Chunk A — Auth & User Management

Implement JWT auth filter, user entity, user repo, service.

Implement password hashing (BCrypt).

Create initial SUDO seed user via data.sql or Flyway baseline.

RBAC: method-level security with @PreAuthorize("hasRole('ADMIN')").

Tests: unit tests for auth service + integration test for login endpoint.

Chunk B — Patients & Salutation/Gender logic

Patient entity + DTO mapping.

Enforce salutation→gender mapping in service (Master/Mr->Male, Ms/Mrs/Baby->Female, Baby of->require sex).

Implement API + validation errors.

Frontend form implementing auto-map (see front-end chunk).

Tests: unit tests for mapping & API validation.

Chunk C — Test Templates & Index

TestTemplate entity (JSONB field).

Admin endpoints to CRUD templates.

Create GIN index in Flyway (only in Postgres). For H2 use string index.

Tests: CRUD APIs + schema validation tests.

Chunk D — Visit creation & invoice

Visit create flow: create visit row, create visit_tests rows, compute invoice and invoice_items.

Save invoice row with status DUE; return invoiceId.

B2B flow: if client.type == REFERRAL_LAB, create b2b_ledger row; otherwise normal invoice.

Tests: create visit -> assert invoice & ledger entries.

Chunk E — Phlebotomy & samples

Phlebotomy endpoints to mark sample collected.

Update visit_test.status accordingly.

Tests: status transitions and permissions.

Chunk F — Test results partial updates & approval

Implement test_results POST to upsert JSONB results; merge partials using jsonb_set.

When result is posted and contains out-of-range values, add a flag in visit_tests.meta (meta->'flags'->'outOfRange': true) for UI highlight.

Approver endpoints to approve results. Approval sets approved = true and approved_at & copies to invoice finalization process.

Tests: partial update test; approval flows; authorization.

Chunk G — Culture & sensitivity

CRUD for antibiotics master data.

Endpoint to add C/S entries per visit_test (normalized table).

Report generation includes C/S rows.

Tests: add entries, assert correctness.

Chunk H — Hiding visits & remarks & audit logs

Reception can add remark: visit_remarks.

Admin PATCH /visits/{id}/hide with hidden_reason; set hidden=true, hidden_by, hidden_at.

Audit log entry created on hide/unhide and important actions (payments, approvals).

Hidden visits excluded from B2B and patient views but included in Admin exports.

Tests: ensure labs can't see hidden visits, admin can.

Chunk I — Invoices, discounts approval, payments

Discount request stored on invoice (discount_request, discount_status='Requested').

Admin approves discount -> set discount_approved, recalc total.

Payment endpoint -> create payment record, update invoice.paid_total, status PAID if fully paid.

Report printing check: print allowed if (client.type == REFERRAL_LAB) OR (invoice.paid_total >= invoice.total) OR (client.type == INTERNAL => expense logic). If unpaid and patient -> block/return watermark.

Tests: discount request lifecycle, payment flows.

Chunk J — B2B ledger & exports

B2B ledger: when visit created with client.type REFERRAL_LAB, ledger entry created. If partial payment later, mark paid.

Labs login (B2B_USER) to portal; can export CSV (only their visits).

Admin exports include hidden flag & remarks.

Tests: B2B export contents, balances.

Chunk K — Report PDF generation & access

Template-driven PDFs (use openhtmltopdf or iText or Flying Saucer), provide HTML template + CSS.

PDF stored in file storage (local path) and path saved in invoice.report_pdf_path. For large scale later use S3.

Endpoint to download checks authorization & invoice payment rules. For patient portal issue time-limited signed token if necessary (JWT with short expiry).

Tests: PDF exists, content includes approved results, blocked if unpaid.

Chunk L — Frontend (internal) and Patient portal

Implement Reception UI, Phlebotomy UI, Lab UI, Approver UI, Admin UI as React modules.

Patient portal on Next.js: Visit ID + Phone form -> fetch visit -> show result + PDF download if allowed.

Implement salutation→gender logic and guardian display in forms.

Tests: Cypress E2E covering flows.

8 — JSONB usage, partial updates & sample queries

Partial update (Postgres): Add Platelet to results:

UPDATE test_results
SET results = jsonb_set(results, '{Platelet}', '"250000"', true),
    updated_at = now()
WHERE id = :id;


Merge object partial update:

UPDATE test_results
SET results = results || '{"NewField": "value"}'::jsonb
WHERE id = :id;


Query for Hemoglobin value:

SELECT (results->>'Hemoglobin')::numeric AS hemoglobin FROM test_results WHERE visit_test_id = :vtid;


Find results with Hemoglobin < 12:

SELECT tr.*
FROM test_results tr
WHERE (tr.results->>'Hemoglobin')::numeric < 12;


Use GIN index for performance.

9 — Indexing strategy & performance notes

GIN index on test_templates.parameters and test_results.results. (See Flyway file).

Index visits(visit_code) and visit_tests(status) for status queries.

Create expression indexes only when profiling shows a hot path (e.g., ((results->>'Hemoglobin')::numeric) — consider functional index in Postgres).

For heavy file storage (PDFs), use disk or object store; do not store large blobs in DB.

10 — Audit, remarks, and hidden visits behavior

All actions that change critical state must write to audit_logs:

create visit, update visit, hide/unhide, create invoice, approve discount, payment, add remark, approve result.

Hidden visits: set hidden = true. UI/backend filters must WHERE hidden = false for lab and B2B endpoints. Admin endpoints include hidden visits.

Remarks: visit_remarks appended; not editable by non-admins. UI shows chronological list.

11 — B2B ledger behavior & settlement flow

When client.type = REFERRAL_LAB for a visit:

Create b2b_ledger row with amount = invoice.total, paid=false.

Allow referral lab portal to download report immediately; update client's credit_balance as sum of ledger amounts (or maintain derived balance by summing).

Partial payment: create payments record with invoice_id and update b2b_ledger.paid at line-level or update separate b2b_payments.

Monthly settlement: admin exports reports/b2b?from=&to=; optionally admin can create a settlement record and mark ledger entries as settled.

12 — PDF generation & report rules

Use HTML templates for reports; CSS for print styles. Tools: openhtmltopdf or Flying Saucer + iText or wkhtmltopdf.

PDF content: header (lab details), patient details, visit details, test results (approved only), culture table, footer (disclaimer + signatory + lab license).

If invoice unpaid and client.type == PATIENT: either return HTTP 403 with message, or return a watermarked PDF with DUE overlay. Prefer: block final official report; allow "Doctor Copy" watermark if needed via a flag doctor_copy=true.

Save PDF path in invoices.report_pdf_path. Use standard naming: reports/{visit_code}_{timestamp}.pdf.

13 — Excel/CSV export behavior (labs & admin)

Implement server-side export with Apache POI (Java) or OpenCSV for CSV. For XLSX prefer Apache POI.

B2B export: GET /reports/b2b?clientId=...&from=...&to=... — only includes visits where client_id matches and hidden = false. Columns: VisitID, VisitCode, Date, PatientName, TestsCount, Amount, PaymentStatus, RemarkConcatenated.

Admin export: same plus hidden column and full audit info.

14 — Frontend components & validation (salutation/gender rules + forms)
Patient/Reception form fields

Salutation: select (Mr, Ms, Mrs, Master, Baby, Baby of)

Name: text

Age: integer years (if <1, use months/days fields)

Sex: auto-filled AND disabled for fixed salutations (Mr/Master/Ms/Mrs/Baby), empty & required for Baby of.

Guardian name: visible only for Baby of.

UI Behavior (pseudo-code)
onSalutationChange(salutation) {
  if (['Mr','Master'].includes(salutation)) { setSex('Male'); setSexDisabled(true); }
  else if (['Ms','Mrs','Baby'].includes(salutation)) { setSex('Female'); setSexDisabled(true); }
  else if (salutation === 'Baby of') { setSex(''); setSexDisabled(false); showGuardian(true); }
}

Component list

ReceptionCreateVisitForm (auto price calc, test chooser)

PhlebotomyQueue (list pending samples, search)

LabResultEntry (dynamic form builder from template.parameters)

CultureGrid (antibiotic dropdown + sensitivity)

ApproverDashboard (list awaiting approval)

AdminDashboard (users, tests, ledger, exports)

PatientPortal (Visit lookup and PDF download)

ReferralLabPortal (login + export CSV + list visits + balance)

15 — Security: JWT, RBAC, logging, sensitive data

Use JWT (HS256) with JWT_SECRET env var. Token payload includes userId, role, exp.

Enforce RBAC server-side via Spring Security method-level @PreAuthorize.

HTTPS enforced by Cloudflare Tunnel or reverse proxy.

Store password hashes with BCrypt.

Limit report downloads: check invoice and client type; require either invoice.paid or client.type != PATIENT. For patient downloads use visitId + phone + short-lived token.

Log all suspicious actions to audit_logs.

Backup schedule and encryption at rest for backups.

16 — Testing plan (unit, integration, E2E) — concrete items
Unit tests (JUnit + Mockito)

AuthService: login/invalid credentials/role extraction

PatientService: salutation→gender mapping, validation

TestTemplateService: CRUD & parameter JSON validation

InvoiceService: total calculation, discount request logic

Integration tests (SpringBootTest + Testcontainers)

H2 for simple, Testcontainers Postgres for integration mimic

Full create-visit -> add-tests -> create-invoice -> pay -> create-results -> approve -> generate-report

API contract tests (REST Assured)

Auth scenarios, RBAC checks

Ensure hidden visits don't appear to B2B users

E2E (Cypress)

Reception: create visit -> receive visit code

Phlebotomy: mark collected

Lab: partial results -> submit-for-approval

Approver: approve -> invoice paid -> patient portal download

Export/Excel tests

Verify CSV columns count & totals match invoice sums

17 — DevOps: Docker Compose, environment variables, Flyway, CI
Docker Compose (local dev)

docker-compose.yml

version: '3.8'
services:
  db:
    image: postgres:15
    restart: unless-stopped
    environment:
      POSTGRES_DB: labops
      POSTGRES_USER: labops
      POSTGRES_PASSWORD: labops
    volumes:
      - db_data:/var/lib/postgresql/data
  app:
    build: .
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/labops
      SPRING_DATASOURCE_USERNAME: labops
      SPRING_DATASOURCE_PASSWORD: labops
      JWT_SECRET: change-me
    ports:
      - "8080:8080"
    depends_on:
      - db
volumes:
  db_data:

Flyway

Put V1__init.sql in src/main/resources/db/migration.

CI (GitHub Actions)

Steps:

Checkout

Build (mvn -DskipTests package)

Run unit tests

Start postgres service and run integration tests (Testcontainers or docker-compose)

Lint / static analysis

Build Docker image on push to main

18 — Migration path H2 → Postgres and maintenance notes

Use Flyway from day 1, develop migrations compatible with Postgres. For H2, some JSONB features may not work; use CLOB fallback. Provide two migration files if needed: V1__init.sql (Postgres) + V1__init_h2.sql for H2 differences. In Spring Boot dev profile, set spring.flyway.url to H2 and point to alternate migrations folder.

Test migration steps by starting a clean Postgres and running Flyway.

For going live: stop writes, take DB backup, run migrations, start app with Postgres config.

19 — Runbook / operational checklist

Daily:

DB backup snapshot (cron job), verify backups.

Check b2b_ledger negative balances older than X days — export list.

Weekly:

Reconcile payments with bank statements.

Generate B2B reports and email to accounting.

On report generation failure:

Retry logs, regenerate HTML → PDF, check template data for missing approved results.

20 — Deliverables & Acceptance Criteria

Deliverables

Flyway migrations (V1 SQL) in repo

Spring Boot app with modules: auth, patients, visits, tests, results, invoices, b2b ledger, exports

React internal UI + Next.js Patient portal sample pages

Docker Compose for local dev

CI pipeline with tests

Postgres-compatible JSONB handling and GIN index

Acceptance Criteria

End-to-end automated tests pass: create visit → add tests → enter partial result → approve → generate PDF (blocked if unpaid for patients)

B2B labs can login, export their visits, and their exports exclude hidden visits.

Admin can hide visits, add remarks; hidden visits excluded from B2B and patient views but visible to Admin exports.

Salutation → gender mapping enforced exactly as: Mr/Master→Male (fixed), Ms/Mrs→Female (fixed), Baby→Female (fixed), Baby of→gender required manually.

JSONB partial updates work; GIN index present in Postgres.

Invoice discount flow requires admin approval.

Audit logs capture all hide/unhide/approve/payment actions.

Appendix A — Sample SQL queries & utilities

Recompute invoice totals (example):

UPDATE invoices i
SET subtotal = sub.sum_sub,
    total = sub.sum_sub + coalesce(i.tax,0) - coalesce(i.discount_approved,0)
FROM (
  SELECT invoice_id, sum(line_total) as sum_sub
  FROM invoice_items
  GROUP BY invoice_id
) sub WHERE i.id = sub.invoice_id;


Find visits that are hidden but referenced in exports:

SELECT v.id, v.visit_code, v.created_at, v.hidden_reason, a.details
FROM visits v
LEFT JOIN audit_logs a ON a.entity='visits' AND a.entity_id = v.id
WHERE v.hidden = true
ORDER BY v.hidden_at DESC;

Appendix B — Example OpenAPI snippet (for /visits POST)
paths:
  /visits:
    post:
      summary: Create a visit and invoice
      tags: [Visits]
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateVisitRequest'
      responses:
        '201':
          description: Created
components:
  schemas:
    CreateVisitRequest:
      type: object
      properties:
        patientId:
          type: integer
        salutation:
          type: string
        name:
          type: string
        age_years:
          type: integer
        sex:
          type: string
        referred_doctor_id:
          type: integer
        tests:
          type: array
          items:
            type: object
            properties:
              templateId:
                type: integer
              price:
                type: number
