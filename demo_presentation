<div align="center">
💰 Tax Payment Demo
A Hands-On Guide to Building Payment Systems
https://spring.io/
https://www.postgresql.org/
https://www.postman.com/
Learn how tax payment processing works — from invoice creation to payment allocation, idempotency, and audit trails.
</div>
📋 Table of Contents
🎯 What is this project?
🛠️ Setup Guide
🗄️ Database Overview
🎬 Live Demo Steps
📚 API Quick Reference
🔧 Troubleshooting
📝 Presentation Script
🚀 One-Page Cheat Sheet
🎯 What is this project?
Imagine a tax office that needs to:
📨 Send bills to taxpayers
💵 Process payments securely
📊 Record every action for compliance
This is a production-ready Spring Boot application that demonstrates real-world payment processing patterns including idempotency, audit trails, and ledger accounting.
Architecture Overview
plain
┌─────────────┐     HTTP/JSON      ┌─────────────────┐     JDBC      ┌──────────────┐
│   Postman   │ ═══════════════► │  Spring Boot    │ ═══════════► │  PostgreSQL  │
│  (Client)   │                  │   (Port 8080)   │              │   (pgAdmin)  │
└─────────────┘                  └─────────────────┘              └──────────────┘
Key Concepts
Table
Term	Meaning
Invoice	A tax bill with principal, interest, and penalty components
TIN	Taxpayer Identification Number (e.g., 123456789)
Idempotency Key	Prevents duplicate charges on network retries
Payment Allocation	Penalty → Interest → Principal (in that order!)
Audit Trail	Immutable log of every system action
Transaction Ledger	Business events: created, paid, voided
⚡ Payment Rule: Money is always applied in this order:
plain
1️⃣ Penalty first → 2️⃣ Interest second → 3️⃣ Principal last
🛠️ Setup Guide
Prerequisites
Java 17+
Maven
Docker & Docker Compose
Postman
pgAdmin 4
Step 1: Start PostgreSQL
bash
cd /home/josi-coder/Documents/demo/tax-payment-trainee
docker compose up -d postgres
💡 Tip: If you already have PostgreSQL running with tax_payment_db, you can skip Docker.
Step 2: Create Database & User
Open pgAdmin → Query Tool → Execute:
sql
-- Create database and user
CREATE DATABASE tax_payment_db;
CREATE USER tax_payment_user WITH PASSWORD 'secure_password_change_me';
GRANT ALL PRIVILEGES ON DATABASE tax_payment_db TO tax_payment_user;

-- Grant schema permissions
\c tax_payment_db;
GRANT ALL ON SCHEMA public TO tax_payment_user;
Step 3: Start the Application
bash
cd /home/josi-coder/Documents/demo/tax-payment-trainee
mvn spring-boot:run
Wait for: Started TaxPaymentApplication in X.XXX seconds
✅ API is live at http://localhost:8080
Step 4: Import Postman Collection
Open Postman → Click Import (top left)
Select tax-payment-api.postman_collection.json
Collection "Tax Payment API Demo" appears in sidebar
Collection Variables (pre-configured):
Table
Variable	Value	Purpose
baseUrl	http://localhost:8080	API base address
invoiceId	(auto-filled)	Created invoice ID
paymentId	(auto-filled)	Created payment ID
Step 5: Configure pgAdmin Connection
Table
Setting	Value
Name	Tax Payment Local
Host	localhost
Port	5432
Database	tax_payment_db
Username	tax_payment_user
Password	secure_password_change_me
✅ Health Check
In Postman, run 0. Health Check → Actuator Health
JSON
{
  "status": "UP"
}
Green light! You're ready to demo. 🚀
🗄️ Database Overview
Schema Diagram
plain
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│    invoices     │────►│ invoice_status  │     │    payments     │
├─────────────────┤     ├─────────────────┤     ├─────────────────┤
│ id (PK)         │     │ id (PK)         │     │ id (PK)         │
│ taxpayer_tin    │     │ code            │     │ invoice_id (FK) │
│ status_id (FK)  │     │ description     │     │ amount          │
│ principal_amount│     └─────────────────┘     │ status          │
│ interest_amount │                              │ idempotency_key │
│ penalty_amount  │     ┌─────────────────┐     │ created_at      │
│ ...             │     │  payment_audit  │     └─────────────────┘
└─────────────────┘     ├─────────────────┤              │
         │              │ id (PK)         │              │
         │              │ payment_id (FK) │◄─────────────┘
         │              │ event_type      │
         │              │ old_status      │     ┌─────────────────┐
         │              │ new_status      │     │  transactions   │
         │              │ payload         │     ├─────────────────┤
         │              │ created_at      │     │ id (PK)         │
         │              └─────────────────┘     │ type            │
         │                                      │ amount          │
         └─────────────────────────────────────►│ invoice_id (FK) │
                                                │ payment_id (FK) │
                                                │ created_at      │
                                                └─────────────────┘
Table Reference
Table
Table	Purpose
invoices	All tax bills with amount breakdowns
invoice_status	Status dictionary: OPEN, PARTIALLY_PAID, PAID, VOIDED
payments	Payment records with idempotency keys
payment_audit	Step-by-step payment processing log
transactions	Immutable business ledger
outbox_events	Event publishing queue (advanced)
🎬 Live Demo Steps
🎤 Pro Tip: Keep pgAdmin Query Tool open alongside Postman. Show the database after every API call to prove data persistence.
STEP 1: Create Invoice
Postman: Invoices → 1. Create Invoice (POST)
JSON
{
  "taxpayerTin": "123456789",
  "taxTypeCode": "VAT",
  "taxYear": 2026,
  "taxMonth": 6,
  "principalAmount": 200.00,
  "interestAmount": 15.00,
  "penaltyAmount": 5.00,
  "currency": "USD"
}
Response:
JSON
{
  "invoiceId": "550e8400-e29b-41d4-a716-446655440000",
  "status": "OPEN",
  "outstandingAmount": 220.00,
  "paidPrincipal": 0.00,
  "paidInterest": 0.00,
  "paidPenalty": 0.00
}
Table
Field	Value	Meaning
status	OPEN	Not paid yet
outstandingAmount	220.00	Total owed (200 + 15 + 5)
🗣️ Say: "We created a VAT invoice for taxpayer 123456789. Status is OPEN, total owed is $220."
pgAdmin Verify:
sql
SELECT i.id, i.taxpayer_tin, ist.code AS status,
       i.principal_amount, i.interest_amount, i.penalty_amount
FROM invoices i
JOIN invoice_status ist ON i.status_id = ist.id
ORDER BY i.id DESC LIMIT 1;
STEP 2: Get Invoice by ID
Postman: Invoices → 2. Get Invoice by ID (GET)
URL: {{baseUrl}}/api/invoices/{{invoiceId}}
🗣️ Say: "We can always retrieve an invoice by its unique ID for verification."
STEP 3: List Invoices by TIN
Postman: Invoices → 3. List Invoices by TIN (GET)
URL: {{baseUrl}}/api/invoices?taxpayerTin=123456789
🗣️ Say: "A taxpayer can have multiple invoices. This endpoint returns all invoices for a given TIN."
STEP 4: Pay Invoice — $50 Partial Payment ⭐
Postman: Payments → 5. Pay Invoice (50 USD partial) (POST)
JSON
{
  "idempotencyKey": "{{lastIdempotencyKey}}",
  "invoiceId": "{{invoiceId}}",
  "amount": 50.00,
  "currency": "USD"
}
💡 Allocation Logic:
plain
Payment: $50.00
├── Penalty:  $5.00  (fully paid ✅)
├── Interest: $15.00 (fully paid ✅)
└── Principal: $30.00 (partially paid, $170 remaining)

Remaining Balance: $170.00
Response:
JSON
{
  "paymentId": "660e8400-e29b-41d4-a716-446655440001",
  "status": "SUCCESS",
  "referenceNumber": "REF-2026-001"
}
Verify with Get Invoice:
paidPenalty = 5.00
paidInterest = 15.00
paidPrincipal = 30.00
outstandingAmount = 170.00
status = PARTIALLY_PAID
🗣️ Say: "$50 applied in order: penalty first, then interest, then principal. Invoice is now partially paid with $170 remaining."
pgAdmin Verify:
sql
-- Payment record
SELECT id, amount, status, idempotency_key FROM payments ORDER BY created_at DESC LIMIT 1;

-- Audit trail
SELECT event_type, old_status, new_status, payload FROM payment_audit ORDER BY created_at DESC LIMIT 3;

-- Ledger entry
SELECT type, amount, payment_id FROM transactions ORDER BY created_at DESC LIMIT 1;
STEP 5: Idempotency Demo — No Double Charges 🛡️
Postman: Payments → 6. Pay Invoice — DUPLICATE (POST)
⚠️ Do not change the body — same idempotency key as Step 4
Result:
Same paymentId returned
No new payment row in database
Status remains SUCCESS
pgAdmin Verify:
sql
SELECT COUNT(*) AS payment_count
FROM payments
WHERE idempotency_key = 'THE_KEY_FROM_STEP_4';
-- Result: 1 (never 2!)
🗣️ Say: "Network failures happen. If a client retries the same payment, our idempotency key ensures they are never charged twice. This is critical for financial systems."
STEP 6: Payment Audit API
Postman: Payments → 7. Get Payment Audit Log (GET)
URL: {{baseUrl}}/api/payments/{{paymentId}}/audit
Response:
JSON
[
  { "eventType": "REQUESTED", "oldStatus": null, "newStatus": "PENDING" },
  { "eventType": "SUCCESS", "oldStatus": "PENDING", "newStatus": "SUCCESS" }
]
🗣️ Say: "For regulatory compliance, every payment state transition is logged immutably. We can query this via API or inspect it directly in the database."
STEP 7: Pay Remaining $170 — Invoice Paid in Full ✅
Postman: Payments → 8. Pay Invoice (170 USD — pay in full) (POST)
Verify with Get Invoice:
outstandingAmount = 0.00
status = PAID
paidPrincipal = 200.00
🗣️ Say: "Invoice fully settled. Status automatically transitions to PAID when outstanding balance reaches zero."
STEP 8: List All Transactions
Postman: Transactions → 9. List All Transactions (GET)
Expected entries:
Table
Type	Amount	Description
INVOICE_CREATED	220.00	Invoice generated
PAYMENT_RECEIVED	50.00	Partial payment
PAYMENT_RECEIVED	170.00	Final payment
🗣️ Say: "This is our immutable business ledger. Every financial event is recorded for auditing and reconciliation."
STEP 9: List Transactions by Invoice
Postman: Transactions → 10. List Transactions by Invoice (GET)
🗣️ Say: "We can also filter the ledger to show only events for a specific invoice."
STEP 10: Void Invoice 🚫
⚠️ Important: You cannot void a PAID invoice. Create a new invoice first.
Postman:
Run Invoices → 1. Create Invoice again
Run Invoices → 4. Void Invoice (PUT)
URL: {{baseUrl}}/api/invoices/{{invoiceId}}/void
Result: Status = VOIDED
🗣️ Say: "Voiding cancels an unpaid invoice. Any attempt to pay a voided invoice will be rejected — our business rules enforce data integrity."
Bonus: Try to pay the voided invoice → receive error response.
📚 API Quick Reference
Table
#	Method	Endpoint	Description
0	GET	/actuator/health	Health check
1	POST	/api/invoices	Create new invoice
2	GET	/api/invoices/{id}	Retrieve invoice
3	GET	/api/invoices?taxpayerTin={tin}	List by taxpayer
4	PUT	/api/invoices/{id}/void	Cancel invoice
5	POST	/api/payments	Process payment
6	GET	/api/payments/{id}/audit	Payment audit log
7	GET	/api/transactions	Full ledger
8	GET	/api/transactions/by-invoice/{id}	Invoice ledger
🔧 Troubleshooting
Table
Symptom	Cause	Solution
❌ "Could not send request"	App not running	mvn spring-boot:run
❌ invoiceId is empty	Skipped Step 1	Run Create Invoice first
❌ pgAdmin connection fails	PostgreSQL down	docker compose up -d postgres
❌ Void fails with 400	Invoice already PAID	Create new invoice, then void
❌ "Exceeds outstanding balance"	Amount too high	Check outstandingAmount first
❌ "Invoice is voided"	Business rule	Expected — voided invoices cannot be paid
📝 Presentation Script
Opening (30 seconds)
"Today I'll demonstrate a tax payment processing system built with Spring Boot and PostgreSQL. We'll create invoices, process payments with automatic allocation, enforce idempotency for reliability, and maintain a complete audit trail. Every API call will be verified in the database using pgAdmin."
During Demo
Table
Step	Key Message
Create Invoice	"Invoice OPEN, total owed $220."
Pay $50	"Applied: penalty $5, interest $15, principal $30. Remaining: $170."
Idempotency	"Same request twice = one charge. Network-safe by design."
Pay $170	"Fully paid. Status automatically becomes PAID."
Void	"Void cancels unpaid invoices. Business rules prevent paying voided bills."
Closing (30 seconds)
"Every action is persisted and auditable. We have invoices, payments, state-machine-driven status changes, immutable audit logs, and a transaction ledger. This is how production payment systems ensure data integrity and regulatory compliance."
🚀 One-Page Cheat Sheet
Startup Commands
bash
# Terminal 1: Database
docker compose up -d postgres

# Terminal 2: Application
mvn spring-boot:run

# Postman: Import collection
tax-payment-api.postman_collection.json

# pgAdmin: Register server
localhost:5432 / tax_payment_db / tax_payment_user
Demo Sequence
plain
1. Create Invoice    → OPEN, owed $220
2. Get Invoice       → Verify details
3. List by TIN       → All taxpayer invoices
4. Pay $50           → PARTIALLY_PAID, owed $170
5. Duplicate pay     → Same ID, no double charge
6. Payment audit     → REQUESTED → SUCCESS
7. Pay $170          → PAID, owed $0
8. List transactions → Full ledger
9. Create NEW invoice → Then Void → VOIDED
Essential pgAdmin Queries
sql
-- Latest invoices
SELECT * FROM invoices ORDER BY id DESC LIMIT 3;

-- Latest payments
SELECT * FROM payments ORDER BY created_at DESC LIMIT 3;

-- Payment audit trail
SELECT * FROM payment_audit ORDER BY created_at DESC LIMIT 5;

-- Transaction ledger
SELECT * FROM transactions ORDER BY created_at DESC LIMIT 5;

-- Check invoice status
SELECT i.id, ist.code 
FROM invoices i 
JOIN invoice_status ist ON i.status_id = ist.id 
ORDER BY i.id DESC LIMIT 1;
<div align="center">
Made with ❤️ for learning payment systems
Keep this README open during your presentation and follow step by step.
</div>
