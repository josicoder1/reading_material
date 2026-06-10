# Tax Payment Demo — Simple Step-by-Step Guide

This guide explains **everything** in plain language. Follow it from top to bottom during your presentation.

---

## Part 0 — What is this project? (read this first)

Imagine a **tax office**:

1. The office sends a **bill (invoice)** to a taxpayer.
2. The taxpayer **pays** some or all of the bill.
3. The office records **every action** in a database so nothing is lost.

This project is a small **web application** that does exactly that.

### The 3 tools you will use

| Tool | What it is | What you do with it |
|------|------------|---------------------|
| **Spring Boot app** | The program running on your computer (port 8080) | It receives requests and saves data |
| **Postman** | A program to **send requests** to the app (like a browser for APIs) | You click "Send" and see the answer (JSON) |
| **pgAdmin** | A program to **look inside the database** (PostgreSQL) | You run SQL queries and see tables/rows |

### Simple picture

```
You (Postman)  →  sends request  →  Spring Boot App  →  saves/reads  →  Database (pgAdmin shows this)
```

### Important words (simple meanings)

| Word | Simple meaning |
|------|----------------|
| **API** | A set of URLs the app listens to. Example: `http://localhost:8080/api/invoices` |
| **Endpoint** | One URL + method. Example: `POST /api/invoices` = "create invoice" |
| **JSON** | Text format for data. Example: `{ "amount": 50 }` |
| **Invoice** | A tax bill. Has: principal, interest, penalty |
| **TIN** | Taxpayer ID number (like `123456789`) |
| **Payment** | Money paid against an invoice |
| **Idempotency key** | A unique ID for a payment. If you send the same key twice, you are **not charged twice** |
| **Audit** | A log of what happened step by step |
| **Transaction (ledger)** | A record of business events: invoice created, payment received, invoice voided |

### Payment rule (very important for demo)

When someone pays, money is used in this order:

```
1. Penalty first
2. Interest second  
3. Principal last
```

**Example:** Invoice has penalty=5, interest=15, principal=200 (total 220).  
Customer pays **50**:

- 5 goes to penalty (penalty done)
- 15 goes to interest (interest done)
- 30 goes to principal (30 of 200 paid)
- 0 left → still owes 170

---

## Part 1 — Setup before presentation (do this at home)

### 1.1 Start the database (PostgreSQL)

Open a terminal in the project folder and run:

```bash
cd /home/josi-coder/Documents/demo/tax-payment-trainee
docker compose up -d postgres
```

**What this does:** Starts PostgreSQL in Docker on port 5432.

If you already have PostgreSQL running with `tax_payment_db`, you can skip Docker.

---

### 1.2 Create database and user (only first time)

Open **pgAdmin** → connect to PostgreSQL → open **Query Tool** → paste and run:

```sql
CREATE DATABASE tax_payment_db;
CREATE USER tax_payment_user WITH PASSWORD 'secure_password_change_me';
GRANT ALL PRIVILEGES ON DATABASE tax_payment_db TO tax_payment_user;
```

Then select database `tax_payment_db` and run:

```sql
GRANT ALL ON SCHEMA public TO tax_payment_user;
```

**What this does:** Creates the database the app uses (see `application.yaml`).

---

### 1.3 Start the Spring Boot application

In terminal:

```bash
cd /home/josi-coder/Documents/demo/tax-payment-trainee
mvn spring-boot:run
```

**Wait until you see something like:** `Started TaxPaymentApplication`

**What this does:** Starts the API on `http://localhost:8080`

**If it fails:** Usually database is not running or wrong password.

---

### 1.4 Setup Postman

1. Open **Postman** application
2. Click **Import** (top left)
3. Click **Upload Files**
4. Choose file: `tax-payment-api.postman_collection.json` (in project folder)
5. Click **Import**

You should see a collection named **"Tax Payment API Demo"** on the left.

**Collection variables** (already set for you):

| Variable | Value | Meaning |
|----------|-------|---------|
| `baseUrl` | `http://localhost:8080` | Where the app runs |
| `invoiceId` | empty at start | Filled automatically after Create Invoice |
| `paymentId` | empty at start | Filled automatically after Pay Invoice |

To see variables: click collection name → **Variables** tab.

---

### 1.5 Setup pgAdmin connection

1. Open **pgAdmin**
2. Right-click **Servers** → **Register** → **Server**
3. **General** tab → Name: `Tax Payment Local`
4. **Connection** tab:

| Field | Value |
|-------|-------|
| Host | `localhost` |
| Port | `5432` |
| Database | `tax_payment_db` |
| Username | `tax_payment_user` |
| Password | `secure_password_change_me` |

5. Click **Save**

**To open Query Tool:** Expand server → Databases → `tax_payment_db` → right-click → **Query Tool**

---

### 1.6 Check everything works

In Postman, open folder **0. Health Check** → **Actuator Health** → click **Send**.

**You should see:**

```json
{
  "status": "UP"
}
```

If `UP` → you are ready for the demo.

---

## Part 2 — Database tables (show in pgAdmin at start)

In pgAdmin Query Tool, run:

```sql
SELECT table_name FROM information_schema.tables
WHERE table_schema = 'public'
ORDER BY table_name;
```

### What each table stores

| Table name | What is inside (simple) |
|------------|-------------------------|
| `invoices` | All tax bills |
| `invoice_status` | Status names: OPEN, PARTIALLY_PAID, PAID, VOIDED |
| `payments` | All payment records |
| `payment_audit` | Step-by-step log for each payment |
| `transactions` | Business ledger (created, paid, voided) |
| `outbox_events` | Events waiting to be published (advanced topic) |

**Say to audience:** "Every API call you will see in Postman also leaves data in these tables."

---

## Part 3 — The demo (do steps in this exact order)

---

## STEP 1 — Create Invoice

### What this step does
Creates a new tax bill for taxpayer `123456789`.

### Postman — exactly what to click

1. Left side: expand **Tax Payment API Demo**
2. Expand folder **Invoices**
3. Click **1. Create Invoice**
4. Make sure method is **POST** and URL is `{{baseUrl}}/api/invoices`
5. Click tab **Body** → should show **raw** and **JSON**
6. You will see this JSON (do not change for first demo):

```json
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
```

7. Click blue **Send** button

### What each field means

| Field | Meaning | Our value |
|-------|---------|-----------|
| `taxpayerTin` | Taxpayer ID | 123456789 |
| `taxTypeCode` | Type of tax | VAT |
| `taxYear` / `taxMonth` | Tax period | June 2026 |
| `principalAmount` | Main tax amount | 200.00 |
| `interestAmount` | Interest on late tax | 15.00 |
| `penaltyAmount` | Penalty for late payment | 5.00 |
| `currency` | Money type | USD |

**Total bill = 200 + 15 + 5 = 220 USD**

### What you should see in Postman (response)

Bottom panel → **Body** tab. Example:

```json
{
  "invoiceId": "some-uuid-here",
  "taxpayerTin": "123456789",
  "taxType": "VAT",
  "taxPeriod": "2026-06",
  "status": "OPEN",
  "principalAmount": 200.00,
  "interestAmount": 15.00,
  "penaltyAmount": 5.00,
  "paidPrincipal": 0.00,
  "paidInterest": 0.00,
  "paidPenalty": 0.00,
  "outstandingAmount": 220.00,
  "currency": "USD"
}
```

| Response field | Meaning |
|----------------|---------|
| `invoiceId` | Unique ID of this invoice (saved automatically in Postman) |
| `status` | OPEN = not paid yet |
| `outstandingAmount` | How much is still owed = 220 |
| `paidPrincipal/Interest/Penalty` | How much already paid = 0 |

**Status code:** Should be **200 OK** (green) at top right.

### What to say
> "We created an invoice. Status is OPEN. Total owed is 220 dollars."

### pgAdmin — verify data was saved

Run in Query Tool:

```sql
SELECT i.id, i.taxpayer_tin, ist.code AS status,
       i.principal_amount, i.interest_amount, i.penalty_amount
FROM invoices i
JOIN invoice_status ist ON i.status_id = ist.id
ORDER BY i.id DESC
LIMIT 3;
```

**You should see:** New row, status = `OPEN`, amounts 200, 15, 5.

Also run:

```sql
SELECT type, description, amount, created_at
FROM transactions
ORDER BY created_at DESC
LIMIT 3;
```

**You should see:** One row with `type = INVOICE_CREATED`.

**Say:** "Postman created the invoice, and pgAdmin proves it is in the database."

---

## STEP 2 — Get Invoice by ID

### What this step does
Loads one invoice using its ID.

### Postman

1. Click **2. Get Invoice by ID**
2. URL should be: `{{baseUrl}}/api/invoices/{{invoiceId}}`
   - `{{invoiceId}}` was saved automatically from Step 1
3. Click **Send**

### What you should see
Same invoice data as Step 1 (if you did not pay yet).

### What to say
> "We can always look up one invoice by its ID."

### pgAdmin

Copy `invoiceId` from Postman response, paste into this query:

```sql
SELECT * FROM invoices
WHERE id = 'PASTE-INVOICE-ID-HERE';
```

Compare columns with JSON in Postman.

---

## STEP 3 — List Invoices by TIN

### What this step does
Shows **all invoices** for one taxpayer (by TIN number).

### Postman

1. Click **3. List Invoices by TIN**
2. URL: `{{baseUrl}}/api/invoices?taxpayerTin=123456789`
3. Click **Send**

### What you should see
A **list** (array `[...]`) of invoices. Each item has: invoiceId, status, outstandingAmount.

### What to say
> "A taxpayer can have many invoices. This endpoint lists all invoices for TIN 123456789."

### pgAdmin

```sql
SELECT i.id, i.taxpayer_tin, ist.code AS status
FROM invoices i
JOIN invoice_status ist ON i.status_id = ist.id
WHERE i.taxpayer_tin = '123456789';
```

---

## STEP 4 — Pay Invoice (50 USD) — MOST IMPORTANT STEP

### What this step does
Customer pays 50 USD. System splits the 50 using penalty → interest → principal rule.

### Postman

1. Click **5. Pay Invoice (50 USD partial)** (in Payments folder)
2. Method: **POST**, URL: `{{baseUrl}}/api/payments`
3. Body:

```json
{
  "idempotencyKey": "{{lastIdempotencyKey}}",
  "invoiceId": "{{invoiceId}}",
  "amount": 50.00,
  "currency": "USD"
}
```

4. Click **Send**

### What each field means

| Field | Meaning |
|-------|---------|
| `idempotencyKey` | Unique key — if network fails and client retries, no double charge |
| `invoiceId` | Which invoice to pay |
| `amount` | How much money (50) |
| `currency` | Must match invoice (USD) |

### How 50 USD is split (draw this on board if needed)

```
Start with 50 USD to pay:

Step A — Penalty owes 5   → pay 5   → left: 45
Step B — Interest owes 15 → pay 15  → left: 30
Step C — Principal owes 200 → pay 30 → left: 0

Done. Paid: penalty=5, interest=15, principal=30
Still owes: 170 (220 - 50)
```

### What you should see in Postman

```json
{
  "paymentId": "some-uuid",
  "referenceNumber": "some-ref",
  "status": "SUCCESS",
  "failureReason": null,
  "createdAt": "2026-06-10T..."
}
```

| Field | Meaning |
|-------|---------|
| `paymentId` | ID of this payment (saved in Postman) |
| `status` | SUCCESS = payment worked |
| `referenceNumber` | Bank/gateway reference |

### Check invoice changed

Run **2. Get Invoice by ID** again:

- `paidPenalty` = 5.00
- `paidInterest` = 15.00
- `paidPrincipal` = 30.00
- `outstandingAmount` = 170.00
- `status` = **PARTIALLY_PAID**

### What to say
> "Payment of 50 was applied: first penalty, then interest, then principal. Invoice is now partially paid. 170 still owed."

### pgAdmin — 4 queries to run

**A) Payment row:**

```sql
SELECT id, amount, status, idempotency_key, created_at
FROM payments
ORDER BY created_at DESC
LIMIT 3;
```

**B) Invoice paid amounts:**

```sql
SELECT total_paid_penalty, total_paid_interest, total_paid_principal
FROM invoices
ORDER BY id DESC
LIMIT 1;
```

Should show: 5, 15, 30.

**C) Payment audit (the story of the payment):**

```sql
SELECT event_type, old_status, new_status, payload, created_at
FROM payment_audit
ORDER BY created_at DESC
LIMIT 5;
```

You should see events like:
- `REQUESTED` — payment started
- `SUCCESS` — payment finished, payload shows `penalty=5,interest=15,principal=30`

**D) Transaction ledger:**

```sql
SELECT type, amount, payment_id
FROM transactions
ORDER BY created_at DESC
LIMIT 5;
```

You should see `PAYMENT_RECEIVED`.

---

## STEP 5 — Idempotency (send same payment twice)

### What this step does
Proves that sending the **same payment again** does NOT charge twice.

### Postman

1. Click **6. Pay Invoice — DUPLICATE (idempotency demo)**
2. **Do not change the body** — same idempotency key as Step 4
3. Click **Send**

### What you should see
- Same `paymentId` as Step 4
- Status still SUCCESS
- **No new payment row** in database

### What to say
> "If the client sends the same request twice because of a network error, the system recognizes the idempotency key and does not charge again."

### pgAdmin

```sql
SELECT COUNT(*) AS how_many_payments
FROM payments
WHERE idempotency_key = (
  SELECT idempotency_key FROM payments ORDER BY created_at DESC LIMIT 1
);
```

**Result must be 1** (not 2).

```sql
SELECT event_type FROM payment_audit
ORDER BY created_at DESC
LIMIT 5;
```

You may see `DUPLICATE_REQUEST` — meaning "we saw this again, ignored it."

---

## STEP 6 — Payment Audit API

### What this step does
Shows audit log through the API (same data as `payment_audit` table).

### Postman

1. Click **7. Get Payment Audit Log**
2. URL: `{{baseUrl}}/api/payments/{{paymentId}}/audit`
3. Click **Send**

### What you should see
List of audit events with `eventType`, `oldStatus`, `newStatus`, `payload`.

### What to say
> "For compliance we keep a full audit trail. We can read it from the API or directly from the database."

### pgAdmin

```sql
SELECT * FROM payment_audit
WHERE payment_id = 'PASTE-PAYMENT-ID-HERE'
ORDER BY created_at;
```

Same data as Postman response.

---

## STEP 7 — Pay remaining 170 USD (finish the invoice)

### What this step does
Pays the rest so invoice becomes fully PAID.

### Postman

1. Click **8. Pay Invoice (170 USD — pay in full)**
2. Click **Send**

Body pays **170** with a **new** idempotency key (automatic).

### Then check invoice

Run **2. Get Invoice by ID**:

- `outstandingAmount` = **0**
- `status` = **PAID**

### What to say
> "Invoice is fully paid. Status changed to PAID."

### pgAdmin

```sql
SELECT ist.code FROM invoices i
JOIN invoice_status ist ON i.status_id = ist.id
ORDER BY i.id DESC LIMIT 1;
```

Should show `PAID`.

Optional — outbox event:

```sql
SELECT event_type, event_data, published
FROM outbox_events
ORDER BY created_at DESC
LIMIT 3;
```

---

## STEP 8 — List All Transactions

### What this step does
Shows the full business ledger.

### Postman

1. Click **9. List All Transactions**
2. Click **Send**

### What you should see
List with types:
- `INVOICE_CREATED`
- `PAYMENT_RECEIVED` (maybe 2 times if you paid twice)
- `INVOICE_VOIDED` (only if you voided)

### pgAdmin

```sql
SELECT type, description, amount, invoice_id, payment_id, created_at
FROM transactions
ORDER BY created_at DESC;
```

---

## STEP 9 — List Transactions by Invoice

### Postman

1. Click **10. List Transactions by Invoice**
2. Click **Send**

Shows only events for **your** invoice.

### pgAdmin

```sql
SELECT * FROM transactions
WHERE invoice_id = 'PASTE-INVOICE-ID-HERE'
ORDER BY created_at;
```

---

## STEP 10 — Void Invoice (use a NEW invoice first!)

### Important rule
You **cannot** void a PAID invoice.  
So: create a **new** invoice first, then void it.

### Postman

1. Run **1. Create Invoice** again (new invoiceId)
2. Click **4. Void Invoice**
3. Method: **PUT**, URL: `{{baseUrl}}/api/invoices/{{invoiceId}}/void`
4. No body needed
5. Click **Send**

### What you should see
- Status **200 OK**
- Empty response body (normal)

### What to say
> "Void cancels an unpaid invoice. No more payments allowed."

### pgAdmin

```sql
SELECT ist.code FROM invoices i
JOIN invoice_status ist ON i.status_id = ist.id
WHERE i.id = 'PASTE-NEW-INVOICE-ID';
```

Should show `VOIDED`.

```sql
SELECT type FROM transactions
WHERE invoice_id = 'PASTE-NEW-INVOICE-ID';
```

Should include `INVOICE_VOIDED`.

### Bonus — show error
Try to pay a voided invoice → app returns error. Good to show rules are enforced.

---

## Part 4 — All endpoints (quick reference)

| Step | Method | URL | What it does |
|------|--------|-----|--------------|
| Health | GET | `/actuator/health` | Is app running? |
| 1 | POST | `/api/invoices` | Create invoice |
| 2 | GET | `/api/invoices/{id}` | Get one invoice |
| 3 | GET | `/api/invoices?taxpayerTin=...` | List by taxpayer |
| 4 | PUT | `/api/invoices/{id}/void` | Cancel invoice |
| 5 | POST | `/api/payments` | Pay invoice |
| 6 | GET | `/api/payments/{id}/audit` | Payment audit log |
| 7 | GET | `/api/transactions` | All ledger entries |
| 8 | GET | `/api/transactions/by-invoice/{id}` | Ledger for one invoice |

Full URL example: `http://localhost:8080/api/invoices`

---

## Part 5 — If something goes wrong

| Problem | What to do |
|---------|------------|
| Postman: "Could not send request" | App not running → run `mvn spring-boot:run` |
| Postman: `invoiceId` is empty | Run Step 1 (Create Invoice) first |
| pgAdmin: cannot connect | Check PostgreSQL is running, password correct |
| Void fails | Invoice is already PAID → create new invoice |
| Pay fails "exceeds balance" | Amount too high → pay less or check outstanding |
| Pay fails on voided invoice | Expected — voided invoices cannot be paid |

---

## Part 6 — Short script for your presentation (copy and read)

**Opening (30 seconds):**
> "This is a tax payment system. We create invoices, accept payments, and store everything in PostgreSQL. I will use Postman to call the API and pgAdmin to show the database."

**After Create Invoice:**
> "Invoice created with status OPEN. Total 220 USD. You can see the same row in pgAdmin."

**After Pay 50:**
> "50 dollars applied: 5 penalty, 15 interest, 30 principal. Still owes 170. Status is PARTIALLY_PAID."

**After Idempotency:**
> "Same request sent twice. Only one charge. This protects against network retries."

**After Pay 170:**
> "Invoice fully paid. Status PAID."

**After Void:**
> "Invoice voided. Cannot pay anymore."

**Closing (30 seconds):**
> "Every action in Postman is stored in the database. We have invoices, payments, audit logs, and a transaction ledger. This is how real payment systems keep data safe and traceable."

---

## Part 7 — Print this one-page cheat sheet

```
SETUP:
  mvn spring-boot:run
  Postman: import tax-payment-api.postman_collection.json
  pgAdmin: tax_payment_db / tax_payment_user

DEMO ORDER:
  1. Create Invoice     → status OPEN, owed 220
  2. Get Invoice        → confirm details
  3. List by TIN        → all invoices for taxpayer
  4. Pay 50             → penalty 5 + interest 15 + principal 30, owed 170
  5. Duplicate pay      → same paymentId, no double charge
  6. Payment audit      → REQUESTED, SUCCESS
  7. Pay 170            → status PAID
  8. List transactions  → ledger
  9. Create NEW invoice → then Void → status VOIDED

PGADMIN QUICK CHECK:
  SELECT * FROM invoices ORDER BY id DESC LIMIT 3;
  SELECT * FROM payments ORDER BY created_at DESC LIMIT 3;
  SELECT * FROM payment_audit ORDER BY created_at DESC LIMIT 5;
  SELECT * FROM transactions ORDER BY created_at DESC LIMIT 5;
```

---

*You can keep this file open during the presentation and follow step by step.*
