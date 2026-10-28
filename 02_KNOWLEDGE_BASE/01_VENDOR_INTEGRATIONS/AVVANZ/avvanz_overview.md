---
title: AVVANZ Overview
type: vendor_overview
status: verified
owner: Thang Nguyen
last_updated: 2026-10-28
vendor: AVVANZ
---

# Avvanz API Middleware — Knowledge Base

> **Purpose:** Single source of truth for an AI agent that helps developers and operators use the Avvanz API Middleware (background-check / screening case API for the Avvanz **ScreenGlobal** platform).
> **Built from:** (1) `Avvanz_API_Middleware_v1_8.pdf` (documentation, cover version 1.0.1, authors Cris Bibal & Jeric Macalawa, created July 2025, PDF generated 14 Oct 2025; the second uploaded PDF is a byte-identical duplicate), and (2) `Avvanz_BGC_postman_collection.json` (Postman collection "Avvanz BGC", based on API Middleware v1.0.1).
> **Owner of the API:** Avvanz Inc.
> **Conventions in this file:** "**Documented**" = stated in the PDF or Postman collection. "**Observed**" = visible only in a sample response. "**Inferred**" = reasoned from samples, not stated. "**Unknown**" = not covered by any source — the agent must say so and not guess.

---

## 1. Quick Facts

| Item | Value |
|---|---|
| API style | REST, JSON responses |
| Production base URL | `https://middleware.avvanz.com` |
| Staging base URL | `https://staging-middleware.avvanz.com` |
| Auth | Client credentials → Bearer token (`Authorization: Bearer <access_token>`) |
| Credentials | `client_id` + `client_secret`, issued by Avvanz Inc. (not self-service) |
| Rate limit | **60 requests per minute**; exceeding returns **HTTP 429 Too Many Requests** |
| API version prefix | `/api/v1/` |
| Primary workflow | Auth → discover packages/branches → create case → track status → fetch final report → reconcile invoices |
| Webhooks / callbacks | **Unknown** (none documented; use polling) |
| Token lifetime / refresh | **Unknown** (none documented; re-authenticate on 401) |

---

## 2. Endpoint Index

| # | Name | Method & Path | Auth | Purpose |
|---|---|---|---|---|
| 1 | Get Token | `POST /api/v1/auth/token` | No | Exchange client credentials for a bearer token |
| 2 | Get Packages | `GET /api/v1/package` | Yes | List company packages (sets of checks) |
| 3 | Get Branches | `GET /api/v1/company-branches` | Yes | List company branches |
| 4 | Create Case | `POST /api/v1/case` | Yes | Create a screening case for a candidate/employee |
| 5 | Get All Cases | `GET /api/v1/case` | Yes | List cases (date filter, sort, pagination) |
| 6 | Update Candidate Info | `PUT /api/v1/case/{case_id}/candidate` | Yes | Update candidate details on an existing case |
| 7 | Get Case Status | `GET /api/v1/case/{case_id}` | Yes | Case status, candidate details, checks |
| 8 | Get Case Checks Status | `GET /api/v1/case/{case_id}/checks-status` | Yes | Status/remarks of every check in the case |
| 9 | Get Case Price | `GET /api/v1/case/{case_id}/price` | Yes | Itemised price and totals for the case |
| 10 | Cancel Case | `PUT /api/v1/case/{case_id}/cancel` | Yes | Cancel a case (only in certain statuses) |
| 11 | Final Report — PDF Stream | `GET /api/v1/case/final-report-pdf-stream/{case_id}` | Yes | Binary PDF of the final report |
| 12 | Final Report — Base64 | `GET /api/v1/case/final-report-base64/{case_id}` | Yes | Final report PDF as Base64 string in JSON |
| 13 | Final Report — JSON | `GET /api/v1/case/final-report-json/{case_id}` | Yes | Structured report data |
| 14 | Get Paid Invoices | `GET /api/v1/invoice` | Yes | List paid invoices (date filter, sort, pagination) |

Note: the PDF's table of contents lists only some of these (Get Token … Get Packages); Get Branches, Get All Cases, Update Candidate Info, Get Case Price and Get Paid Invoice appear in the body of the PDF and in the Postman collection.

---

## 3. Getting Access (Onboarding)

1. **Register the company** on the Avvanz ScreenGlobal web app: click *Register* → choose **Company Registration** ("I represent a Company and intend to screen our employees/contractors") → fill in all required fields. (The other option, "I am an individual screening myself", is *not* for API use.)
2. **Contact Avvanz Inc. for package creation.** Packages (the sets of checks applied to a case) are created by Avvanz, not through the API.
3. **Contact Avvanz Inc. for API credentials** (`client_id` and `client_secret`). There is no self-service API account creation.
4. Start in **staging**, then switch `base_url` to production.

Company registration identifies the company and links the packages used in verification.

---

## 4. Common Request Conventions

### 4.1 Headers
| Header | Value | When |
|---|---|---|
| `Accept` | `application/json` | Always (use `application/pdf` for the PDF stream endpoint) |
| `Content-Type` | `application/json` or `application/x-www-form-urlencoded` or `multipart/form-data` | Depends on body encoding (see §6.1, §6.4, §6.6) |
| `Authorization` | `Bearer <access_token>` | All endpoints except Get Token |

### 4.2 Response envelope
Every JSON response uses the same envelope:

```json
{
  "success": true,
  "message": "Human readable message.",
  "data": { },
  "status_code": 200,
  "status_text": "OK"
}
```
- `data` is absent on some responses (e.g., Cancel Case, Update Candidate, Get Token).
- Get Token returns `token_type` and `access_token` at the top level (not inside `data`).

### 4.3 HTTP status codes documented
| Code | Meaning in this API |
|---|---|
| 200 OK | Successful read/update/cancel/login |
| 201 Created | Case created |
| 401 | Invalid credentials at token step (flowchart: "Invalid Credential 401") |
| 422 | Invalid request / validation failure on case creation (flowchart: "Invalid Request 422") |
| 429 | Rate limit exceeded (60 requests/minute) |

Error response body schema: **Unknown** (not documented). *Assumption to verify:* errors keep the same envelope with `success: false`.

### 4.4 Pagination and filtering (list endpoints)
Applies to **Get All Cases** and **Get Paid Invoices**; all parameters optional, sent as query string.

| Param | Type (per docs) | Description |
|---|---|---|
| `date_from` | String(10) | Created-date lower bound. Examples use `YYYY-MM-DD` |
| `date_to` | String(10) | Created-date upper bound. Examples use `YYYY-MM-DD` |
| `sort` | String(3) | `ASC` or `DESC` |
| `offset` | String(100) | Zero-based start index (starts at 0) |
| `limit` | String(10) | Max records per request |

Unknown: default/maximum `limit`, whether a total count is returned (none appears in samples), whether date bounds are inclusive. To page through results, increase `offset` by `limit` until a page returns fewer than `limit` items (inferred).

---

## 5. Authentication

### 5.1 Get Token
- **Endpoint:** `POST /api/v1/auth/token`
- **Description:** First required call. Exchanges client credentials for a bearer token.
- **Auth:** none.
- **Body fields** (Postman uses `application/x-www-form-urlencoded`; the PDF screenshot shows form-data — both are shown to work in the sources):

| Field | Description |
|---|---|
| `client_id` | Provided by Avvanz Inc. |
| `client_secret` | Provided by Avvanz Inc. |

- **Example request:**
```bash
curl --location 'https://staging-middleware.avvanz.com/api/v1/auth/token' \
  --header 'Accept: application/json' \
  --data-urlencode 'client_id=<CLIENT_ID>' \
  --data-urlencode 'client_secret=<CLIENT_SECRET>'
```
- **Example response (200):**
```json
{
  "success": true,
  "message": "Login success",
  "token_type": "Bearer",
  "access_token": "<ACCESS_TOKEN>",
  "status_code": 200,
  "status_text": "OK"
}
```
- **Usage:** send `Authorization: Bearer <access_token>` on every other endpoint. Token format in samples looks like `<number>|<random string>` — treat it as opaque.
- **Errors:** invalid credentials → **401** (loop back and resend correct credentials).
- **Security:** never log, print or store `client_secret` or tokens in shared places (see §12).

---

## 6. Endpoint Reference

### 6.1 Get Packages
- **Endpoint:** `GET /api/v1/package`
- **Description:** Returns the packages configured for the authenticated company. A package is the set of checks applied to a case. **Use the exact returned `package_name` when creating a case.**
- **Optional query (Postman only, undocumented in PDF):** `branch_id` (example `283`, disabled by default) — presumably filters packages by branch (*inferred*).
- **Response `data[]`:** `package_id` (int), `package_name` (string).
- **Example response:**
```json
{
  "success": true,
  "message": "Company packages has been retrieved.",
  "data": [
    { "package_id": 1193, "package_name": "Global Senior Package - Singapore" },
    { "package_id": 1194, "package_name": "Global Senior Package - Philippines" },
    { "package_id": 1195, "package_name": "Global Senior Package - UK" }
  ],
  "status_code": 200,
  "status_text": "OK"
}
```
- **Observed in the PDF screenshot** (same account): more packages such as "Global Senior Package - USA / Australia" and "Global Junior Package - Singapore / Philippines / UK". Package names are per-company; never assume they exist for another company.
- **Note:** Create Case takes `package` as a **name string**, not `package_id`.

### 6.2 Get Branches
- **Endpoint:** `GET /api/v1/company-branches`
- **Description:** Returns company branches. **Use the exact returned `branch_name` when creating a case.**
- **Response `data[]`:** `branch_id` (int), `branch_name` (string).
- **Example:** `{"success": true, "message": "Company branches has been retrieved.", "data": [{"branch_id": 1512, "branch_name": "Branch 1"}], "status_code": 200, "status_text": "OK"}`
- **Note:** Create Case takes `branch` as a **name string**, not `branch_id`.

### 6.3 Create Case
- **Endpoint:** `POST /api/v1/case`
- **Description:** Creates a screening case. Documented as the "first required process … to start the case creation". On success the response contains the case ID and reference number; per the workflow chart the system also **emails the employee** for verification (see §8).
- **Encoding:** The Postman collection sends **raw JSON** (`Content-Type: application/json`). The PDF's screenshot/cURL shows **form-data** (`--form`). Both appear in the sources; prefer JSON (the Postman tests for it expect 201).

**Body fields**

| Field | Type | Mandatory | Description |
|---|---|---|---|
| `first_name` | String(100) | **Yes** | Employee first name |
| `middle_name` | String(100) | Optional | Employee middle name |
| `last_name` | String(100) | Optional | Employee last name (see `no_last_name`) |
| `package` | String(100) | **Yes** | Package **name** (the set of checks for verification) — must match a name from Get Packages |
| `email` | String(100) | **Yes** | Employee email (the verification invitation goes here) |
| `phone_no` | String(50) | Optional | Employee contact number |
| `no_last_name` | Enum(0,1) | Optional | Documented as: `1` = the employee **has** a last name, `0` = the employee has **no** last name. (Counter-intuitive naming — see §11) |
| `branch` | String(100) | Optional | Branch **name** from Get Branches |
| `candidate_data_entry` | Enum(0,1) | Not stated | Documented as: `1` = employee fills in the data; "2" = employer completes the data. The text mentions 2 but the type says (0,1) — see §11 |
| `event_id` | String(100) | Not stated | Unique reference ID from the client system |
| `event_ref_no` | String(150) | Not stated | Unique reference number from the client system |
| `employee_id` | String(100) | Not stated | Employee ID from the client system |
| `cost_center` | String(100) | Not stated | Client cost center |
| `country` | String(200) | **Conditional** | Employee country. **Required if the package's check settings have no pre-selected country or coverage** |
| `requestor_name` | String(150) | Optional | Case requestor name |
| `requestor_email` | String(100) | Optional | Case requestor email |
| `bu` | String(150) | Optional | Business unit name |

"Not stated" means the PDF table leaves the Mandatory cell blank. Postman states: *Required: first_name, package, email. Country is conditionally required.*

**Example request (JSON, from Postman):**
```json
{
  "first_name": "John",
  "middle_name": "",
  "last_name": "Doe",
  "package": "Global Senior Package - Singapore",
  "email": "john.doe@example.com",
  "phone_no": "+6591234567",
  "no_last_name": "0",
  "branch": "Branch 1",
  "candidate_data_entry": "1",
  "event_id": "EVT-1790840134",
  "event_ref_no": "REF-1790840134",
  "employee_id": "EMP-001",
  "cost_center": "Finance",
  "country": "Singapore",
  "requestor_name": "API Requestor",
  "requestor_email": "requestor@example.com",
  "bu": "Corporate"
}
```
**Example response (201):**
```json
{
  "success": true,
  "message": "Case has been created.",
  "data": { "case_id": 733, "case_ref_no": "tpj4913" },
  "status_code": 201,
  "status_text": "Created"
}
```
- **Store both** `case_id` (used in all later URLs — the "Screen Global Case ID") and `case_ref_no`.
- **Errors:** invalid request → **422** (fix parameters and resend).
- **Observed oddity:** in the PDF sample, `middle_name` is sent as the literal string `"null"`; send empty string or omit instead.
- **Idempotency:** **Unknown.** Nothing documents duplicate protection on `event_id`/`event_ref_no`. The agent must not retry Create Case blindly after a timeout; first check Get All Cases.

### 6.4 Get All Cases
- **Endpoint:** `GET /api/v1/case`
- **Description:** Lists created cases. Filters/pagination: see §4.4.
- **Example:** `GET /api/v1/case?date_from=2025-10-01&date_to=2025-10-09&sort=DESC&offset=0&limit=10`
- **Response `data[]` fields:**

| Field | Example | Notes |
|---|---|---|
| `case_id` | 739 | Internal numeric ID used in URLs |
| `case_ref_no` | "PFPAMY2" | Reference number |
| `first_name`, `middle_name`, `last_name` | "John", null, "Doe" | |
| `email`, `phone_no` | | |
| `case_created_date` | "2025-10-01 03:14:34" | `YYYY-MM-DD HH:MM:SS` |
| `order_ref_no` | "O-1759295135-75560" or null | Null until an order is checked out |
| `order_checkout_date` | "2025-10-01 05:05:35" or null | Null until checkout |
| `status` | "Processing", "Insufficient Data" | See §9.1 |

### 6.5 Update Candidate Info
- **Endpoint:** `PUT /api/v1/case/{case_id}/candidate`
- **Description:** Updates candidate details for an existing case. Replace `{case_id}` with the ScreenGlobal case ID returned by Create Case.
- **Encoding:** `application/x-www-form-urlencoded` (PDF cURL and Postman agree). Include `Accept: application/json`.
- **URL parameter:** `case_id` (int, required).
- **Body fields:**

| Field | Type | Mandatory | Description |
|---|---|---|---|
| `first_name` | String(100) | **Yes** | First name |
| `middle_name` | String(100) | No | Middle name |
| `last_name` | String(100) | Optional* | Last name. Screenshot note: "Required if no_last_name is not 0" |
| `email` | String(100) | **Yes** | Email |
| `phone_no` | String(50) | Optional (table) / "Required" (screenshot) | Contact number — sources disagree, see §11 |
| `employee_id` | String(100) | Optional | Client employee ID |
| `no_last_name` | Enum(0,1) | Optional | `1` = has last name, `0` = no last name (as documented) |

- **Example response (200):** `{"success": true, "message": "Case candidate has been updated.", "status_code": 200, "status_text": "OK"}`
- **Unknown:** which case statuses still allow candidate edits after the candidate has begun filling the form.

### 6.6 Get Case Status
- **Endpoint:** `GET /api/v1/case/{case_id}`
- **Description:** Overall status plus candidate details, order/result data and the checks included.
- **Response `data`:**

| Field | Example | Notes |
|---|---|---|
| `case_id` | 733 | |
| `ref_no` | "tpj4913" | Same value as `case_ref_no` elsewhere — **key name differs** |
| `package_name` | "test" | |
| `data_entry_by` | "Candidate" | Who fills the data (matches `candidate_data_entry`) |
| `candidate` | "John Doe" | Full name |
| `email`, `phone_no` | | |
| `position_applied` | "TR053606" | Observed: equals the `employee_id` supplied at creation (*inferred* mapping) |
| `status` | "Cancelled" | See §9.1 |
| `order_id` | null | Null until ordered |
| `result` | null | Final result colour (e.g., `green` in report JSON) once completed |
| `checks[]` | | See §6.7 |

- **Example:**
```json
{
  "success": true,
  "message": "Case status has been retrieved.",
  "data": {
    "case_id": 733, "ref_no": "tpj4913", "package_name": "test",
    "data_entry_by": "Candidate", "candidate": "John Doe",
    "email": "jhon@avvanz.com", "phone_no": "098999999",
    "position_applied": "TR053606", "status": "Cancelled",
    "order_id": null, "result": null,
    "checks": [
      { "check_id": 2241, "check_country_id": 2626,
        "check_name": "Criminal or Criminality Check - Australia (ACIC check)",
        "status": "In Progress", "remarks": "-" }
    ]
  },
  "status_code": 200, "status_text": "OK"
}
```

### 6.7 Get Case Checks Status
- **Endpoint:** `GET /api/v1/case/{case_id}/checks-status`
- **Description:** Status and remarks for every check in the case.
- **Response `data[]`:** `check_id` (int), `check_country_id` (int), `check_name` (string, includes country, e.g., "Education Verification - Philippines"), `status` (string, e.g., "In Progress"), `remarks` (string, "-" when none).
- Use this endpoint to poll progress without downloading the full status object.

### 6.8 Get Case Price
- **Endpoint:** `GET /api/v1/case/{case_id}/price`
- **Description:** Itemised check prices, currency, subtotal, additional costs, discounts, tax, total due.
- **Response `data`:**
```json
{
  "items": [
    { "check_name": "Credit Check - Malaysia (Standard Credit Check)", "price": "20.00" },
    { "check_name": "Criminal or Criminality Check - Malaysia", "price": "30.00" }
  ],
  "currency": "USD",
  "sub_totaL": "50.00",
  "additional_cost_total": "0.00",
  "discount_total": "0.00",
  "tax": "0.00",
  "total_due": "50.00"
}
```
- All money values are **strings with two decimals**. Convert before doing arithmetic.
- **Key typo warning:** the PDF's sample spells the subtotal key `sub_totaL` (capital L). Real responses may use `sub_total`. Code and agents should read either key.

### 6.9 Cancel Case
- **Endpoint:** `PUT /api/v1/case/{case_id}/cancel`
- **Description:** Cancels a created case. **Only cases in status "Insufficient Data" or "Ready For Checkout" can be cancelled.**
- **Body:** none.
- **Example response (200):** `{"success": true, "message": "Case has been cancelled.", "status_code": 200, "status_text": "OK"}`
- **State-changing and likely irreversible** (no un-cancel endpoint is documented). The agent must confirm the exact `case_id` with the user before calling it, and should first check the status with Get Case Status.
- **Unknown:** the error returned when status is not cancellable (assume 4xx; read the message).

### 6.10 Final Report — PDF Stream
- **Endpoint:** `GET /api/v1/case/final-report-pdf-stream/{case_id}`
- **Description:** Downloads the final report as a binary PDF stream.
- **Header:** `Accept: application/pdf`. The response body is a PDF (not JSON) — save it as a file.
- Report appearance (observed): Avvanz-branded "Background Check Report (Confidential due to Personal Data)".

### 6.11 Final Report — Base64
- **Endpoint:** `GET /api/v1/case/final-report-base64/{case_id}`
- **Description:** Final report PDF encoded as Base64 in `data`.
```json
{ "success": true, "message": "Case final report base64 encode.", "data": "<BASE64_STRING>", "status_code": 200, "status_text": "OK" }
```
- Decode with any Base64 decoder and save as `.pdf`. Prefer this over the stream when your HTTP client cannot handle binary bodies.

### 6.12 Final Report — JSON
- **Endpoint:** `GET /api/v1/case/final-report-json/{case_id}`
- **Description:** Structured final report. Message: "Report data has been generated." Full structure in §10.
- Availability: the sample shows status `Completed` with result `green`. **Unknown** whether the endpoint returns a partial report or an error before completion — check the case status first.

### 6.13 Get Paid Invoices
- **Endpoint:** `GET /api/v1/invoice`
- **Description:** Lists **paid** invoices. Filters/pagination: see §4.4 (`date_from`, `date_to`, `sort`, `offset`, `limit`; "Invoice created date").
- **Response `data[]`:**

| Field | Example |
|---|---|
| `invoice_id` | 193 |
| `invoice_no` | "AVNZ-1760421128-1" |
| `company_name` | "TRIP.COM" |
| `branch_name` | "N/A" |
| `currency` | "USD" |
| `sub_total` | "403.00" |
| `discount_amount` | "0.00" |
| `additional_charges` | "0.00" |
| `tax_amount` | "0.00" |
| `total_amount_due` | "403.00" |
| `status` | "Paid" |
| `paid_date` | "2025-10-14" |
| `invoice_pdf_link` | `https://uat-screenglobal.avvanz.com/login?target=https://uat-screenglobal.avvanz.com/invoices/report/193` |

- `invoice_pdf_link` points to the ScreenGlobal web app behind a login redirect; opening it requires an authenticated ScreenGlobal browser session (*inferred* from `login?target=`).
- Money values are strings.

---

## 7. Recommended Integration Recipes

### 7.1 Order a screening for a new hire
1. `POST /api/v1/auth/token` → store `access_token`.
2. `GET /api/v1/package` → pick the exact `package_name` (ask the user if several fit, e.g., by country).
3. `GET /api/v1/company-branches` → pick the exact `branch_name` if the company uses branches.
4. `POST /api/v1/case` with at least `first_name`, `package`, `email` (+ `country` if the package has no preset country; + `event_id`/`event_ref_no`/`employee_id` for traceability) → store `case_id` and `case_ref_no`.
5. On 422, read the message, correct the payload and resend. On 401, re-authenticate.
6. The employee is notified by email (workflow chart) and completes data entry when `candidate_data_entry = 1`.

### 7.2 Track progress
- Poll `GET /api/v1/case/{case_id}` (overall status) or `/checks-status` (per check), or list recent cases with `GET /api/v1/case?date_from=…&sort=DESC`.
- Keep total traffic under **60 requests/minute**; for many cases, batch with the list endpoint and use a polling interval of minutes, not seconds (suggested practice, not documented).

### 7.3 Retrieve results
1. Confirm case status is final (e.g., `Completed`).
2. `final-report-json` for data, `final-report-pdf-stream` or `final-report-base64` for the document.

### 7.4 Correct a mistake
- Wrong name/email/phone/employee ID → `PUT …/candidate`.
- Wrong package or wrong country → no update endpoint exists for these. Cancel (if status allows) and create a new case. (*Inferred:* no documented way to change package.)

### 7.5 Reconcile billing
- Per case: `GET …/case/{case_id}/price`.
- Per period: `GET /api/v1/invoice?date_from=…&date_to=…`; match `invoice_no` to finance records.

### 7.6 Handle rate limiting
- On HTTP 429, wait and retry with backoff (Retry-After header **unknown**; use at least 60 s for the window to reset).

---

## 8. Workflow (from the flow chart "API v1 Flow")

Purpose: *"the process of the Avvanz API Middleware for automatic case creation and notifying the Employee for Verification."*

1. **START**
2. **Send API credentials** → **Validate credentials**
3. Decision **200 or 401**
   - 401 → *Invalid Credential* → resend credentials (loop to step 2)
   - 200 → *Success, get token*
4. **Send parameters with token** → **Process case creation** → **Validate request**
   - Invalid → **422** → correct and resend parameters (loop to step 4)
   - Valid → **201 Success response with Case Ref**, and the **employee is emailed**
5. **END**

---

## 9. Data Dictionary

### 9.1 Case statuses seen in the sources (not an exhaustive enum — **Unknown**)
| Status | Where seen | Meaning / notes |
|---|---|---|
| `Insufficient Data` | Get All Cases sample; Cancel Case rules | Case created but candidate data incomplete. **Cancellable.** No order yet (`order_ref_no` null) |
| `Ready For Checkout` | Cancel Case rules | Data complete, awaiting checkout/order. **Cancellable** |
| `Processing` | Get All Cases sample | Order checked out (`order_ref_no`/`order_checkout_date` set); checks under way |
| `Cancelled` | Get Case Status sample | Cancelled case |
| `Completed` | Final report JSON (`status_text`) | Report complete |

*Inferred lifecycle:* Insufficient Data → Ready For Checkout → Processing → Completed; Cancelled possible only from the first two. How checkout is triggered (API vs ScreenGlobal web UI) is **not documented** — no checkout endpoint exists in this API.

### 9.2 Check statuses seen
`In Progress`, `Clear`, `Stopped` (e.g., remark "Check no longer required as advised by the client."). Full enum **Unknown**. Report-level **result** value observed: `green` (colour-coded result; other values likely but **Unknown**).

### 9.3 Identifier cheat-sheet
| Identifier | Where it comes from | Used for |
|---|---|---|
| `client_id` / `client_secret` | Avvanz Inc. | Get Token |
| `access_token` | Get Token | Bearer auth |
| `case_id` | Create Case (`data.case_id`) | **All case URLs** ("Screen Global Case ID") |
| `case_ref_no` / `ref_no` | Create Case / Get Case Status | Human-facing reference; appears as `mom_ref_no` in the JSON report ("Case Reference No.") |
| `package_id`, `package_name` | Get Packages | Create Case uses `package_name` |
| `branch_id`, `branch_name` | Get Branches | Create Case uses `branch_name` |
| `event_id`, `event_ref_no`, `employee_id`, `cost_center`, `bu` | Client system | Client-side traceability fields stored on the case |
| `check_id`, `check_country_id` | Case status / checks status | Identify a check and its country coverage |
| `order_ref_no` / `order_id` | Get All Cases / Get Case Status | Order created at checkout (null before) |
| `invoice_id`, `invoice_no` | Invoices | Billing |

---

## 10. Final Report JSON — Structure

`GET /api/v1/case/final-report-json/{case_id}` → `data` contains **header blocks** (each `{ "label": "...", "value": "..." }`) and **section blocks**.

### 10.1 Header blocks (label → sample value)
| Key | Label | Sample |
|---|---|---|
| `candidate` | Candidate/Employee Full Name | "James Matias" |
| `company_name` | Company Name | "Individual - Chris" |
| `branch_name` | Branch Name | "-" |
| `report_date` | Date of Report | "01-20-2025" (**MM-DD-YYYY**) |
| `package_name` | Package Code/Level (if any) | "-" |
| `cost_center` | Cost Centre | "-" |
| `mom_ref_no` | Case Reference No. | "AV0117250X6wrd7" |
| `status_text` | Status | "Completed" |
| `result` | Result | "green" |

### 10.2 Field-object shapes
- **Personal section fields:** `{ "label", "candidate_value", "verified_value", "remarks" }`.
- **All other section fields:** `{ "label", "candidate_value", "verified_value", "verified_remarks" }`.
- **Section/check status:** `"status": { "label": "Verified Status", "verified_value": "<Clear|Stopped|In Progress|…>", "remarks": "<free text>" }`.
- `candidate_value` = what the candidate declared; `verified_value` = what the verifier confirmed. `verified_value` may be `null` (not verified/applicable) or `""` (empty); treat both as "no verified value".
- Each section ends with `candidate_files` and `verified_files` arrays. File entries: `{ "doc_type", "filename", "url" }` (doc types seen: `consent`, `valid_id`, `education`, `professional_qualification`). File URLs are login-gated ScreenGlobal links (sample shows a localhost development host; real hosts differ).

### 10.3 Sections and their fields
Each check inside a section is keyed `"<Check name> - <n>"` (e.g., `"Education Verification - 1"`, `"Education Verification - 2"`).

| Section key | Check keys seen | Fields |
|---|---|---|
| `personal` | — | `first_name`, `last_name`, `middle_name`, `aka` (alias/nickname), `birth_date`, `birth_country`, `citizenship`, `phone_no`, `alter_phone_no`, `primary_id_card_no` (Unique ID Number1), `primary_id_card_no_2` (Unique ID Number2), `email`, `gender`, `civil_status`, `position_applied`, `status`, `candidate_files`, `verified_files` |
| `address` | `Address Verification - n`, `Civil Litigation Check - n`, `Criminal or Criminality Check - n` | `address_line_1`, `address_line_2`, `postal_code`, `stay_date_from`, `stay_date_to`, `house_owner_name`, `house_owner_contact_no`, `neighbor_name`, `neighbor_contact_no`, `country`, `status` |
| `employment` | `Employment Details Verification - n` | `company_name`, `annual_salary`, `salary_currency`, `position`, `employment_status`, `date_from`, `date_to`, `country`, `reason_for_leaving`, `postal_code`, `address_line_1`, `address_line_2`, `hr_first_name`, `hr_last_name`, `hr_email`, `hr_personnel_position`, `hr_contact_no`, `eligible_for_rehire`, `agency_name`, `status` |
| `employment_performance` | `Employment Performance Verification - n` | `company_name`, `position`, `address_line_1`, `address_line_2`, `supervisor_first_name`, `supervisor_last_name`, `supervisor_email`, `postal_code`, `supervisor_position`, `country`, `contact_no`, `reference_relationship`, `status` |
| `education` | `Education Verification - n`, `Institution Accreditation Check - n` | `institution_name`, `qualification_attained`, `course`, `date_graduated`, `date_from`, `date_to`, `country`, `address_line_1`, `address_line_2`, `postal_code`, `institution_contact_no`, `institution_email`, `status` |
| `professional_qualification` | `Professional Qualification Verification - n` | `certificate_issuing_authority`, `qualification_attained`, `certificate_number`, `date_granted`, `country`, `status` |

Notes:
- Period dates are `MM-YYYY` (e.g., "08-2016"); `birth_date` is `DD-MM-YYYY` or `MM-DD-YYYY` — **ambiguous** in the sample ("01-01-2001"); do not assume.
- Which sections/checks appear depends on the package ordered. The `address` section holds more than address checks (litigation and criminal checks reuse the address fields).
- `annual_salary` is a string; `salary_currency` is an ISO-like code (e.g., "SGD").
- Remarks can contain who verified and how (e.g., "Verified by <name> - <role> - <phone>, obtained verbally on <date>") — sensitive; see §12.
- In the sample, one `remarks` text is truncated mid-sentence ("…The Commission on Higher Educa") — do not rely on remark length.

---

## 11. Known Ambiguities and Discrepancies (the agent must surface these, not hide them)

1. **`candidate_data_entry`** is typed Enum(0,1) but its description says "1 = Employee will fill out the data. 2 = Employer will complete the data". Valid values may be 1 and 2. Confirm with Avvanz. Only `1` appears in all examples.
2. **`no_last_name` is inverted vs. its name:** documented `1` = has last name, `0` = no last name. The Update Candidate sample sends `no_last_name=1` together with `last_name=Doe`; the Create Case sample sends `0` together with `last_name=Doe` (looks contradictory). Confirm before relying on it.
3. **`phone_no` on Update Candidate:** table says Optional; the Postman screenshot labels it "Required".
4. **Mandatory column blank** for `candidate_data_entry`, `event_id`, `event_ref_no`, `employee_id`, `cost_center` in Create Case. Treat as optional unless Avvanz states otherwise.
5. **Body encoding differs** between sources: Create Case = form-data (PDF) vs JSON (Postman); Get Token = form-data (PDF) vs urlencoded (Postman). Pick one and test on staging.
6. **Get Packages** has an optional `branch_id` query in Postman that the PDF never mentions.
7. **Key typos / inconsistencies:** `sub_totaL` in the price sample; `ref_no` vs `case_ref_no`; `mom_ref_no` for the case reference in the report; `remarks` vs `verified_remarks` inside report fields; `middle_name` sent as `"null"` string in a sample.
8. **Pagination parameter types** are declared as strings (`offset` String(100)); send plain integers as text.
9. **Document metadata:** the file is named "v1_8" but the cover page shows version 1.0.1; the Changelog table in the PDF is **empty**.
10. **Hosts in samples** are development/UAT (`localhost…`, `uat-screenglobal.avvanz.com`), not the production hosts.
11. **Header section naming:** the PDF section titled "HTTP Accept Headers" actually lists `Content-Type: application/json`.

---

## 12. Security and Data-Handling Rules for the Agent

- **Never** output, request in clear text, log or store: `client_secret`, `access_token`, or real candidate personal data (names, emails, phone numbers, ID numbers, birth dates, salaries, addresses, criminal/litigation findings, file URLs) outside the task at hand.
- The final report contains **highly sensitive personal data** (identity numbers, criminal and litigation results, salary, references). Share it only with the authorised requester and only through approved channels.
- Use **staging** for tests and sample data; use placeholders like `<ACCESS_TOKEN>` in all examples.
- Rotate credentials if a token or secret is ever pasted into chat, tickets, source control or shared Postman workspaces.
- **Hard-coded tokens:** a bearer token was found hard-coded in the `Authorization` header of the "Create Case" request inside the supplied Postman collection. It has been deliberately **excluded** from this knowledge base. It should be revoked/rotated and the header replaced with `Bearer {{access_token}}` (the collection-level bearer auth already does this).
- Confirm destructive actions (**Cancel Case**) with the user and verify `case_id` first.
- Respect the 60 requests/minute limit; avoid tight polling loops.

---

## 13. Frequently Asked Questions

**How do I get API credentials?** Contact Avvanz Inc. for `client_id` and `client_secret`. They are not self-service.

**How do I get a package created?** Register the company in ScreenGlobal, then contact Avvanz Inc. Packages cannot be created through the API.

**Which value do I pass for `package` — ID or name?** The exact **name** returned by Get Packages (e.g., "Global Senior Package - Singapore").

**Which value do I pass for `branch`?** The exact **name** returned by Get Branches (e.g., "Branch 1").

**When is `country` required?** When the selected package's check settings do not have a pre-selected country or coverage.

**What does the employee receive after case creation?** An email inviting them to verify/enter data (flowchart: "201 Success Response — Case Ref & Email the Employee"). Exact email content is **Unknown**.

**Which `id` goes into the URL?** `case_id` — the numeric "Screen Global Case ID" from Create Case, not `case_ref_no`.

**Can I cancel any case?** No — only "Insufficient Data" and "Ready For Checkout".

**How do I get the final report?** Three formats: JSON (`final-report-json`), PDF stream (`final-report-pdf-stream`), Base64 (`final-report-base64`).

**How do I get the price of a case?** `GET /api/v1/case/{case_id}/price`.

**How do I list only paid invoices for a month?** `GET /api/v1/invoice?date_from=2025-10-01&date_to=2025-10-31&sort=DESC&offset=0&limit=50`. The endpoint returns paid invoices.

**What if I get 401?** Credentials or token invalid — call Get Token again.

**What if I get 422?** The case payload failed validation — check mandatory fields (`first_name`, `package`, `email`, conditional `country`), exact package/branch names, and enum values.

**What if I get 429?** You exceeded 60 requests/minute — slow down and retry later.

**Does the API push updates (webhooks)?** None documented — poll instead.

**Is there a sandbox?** Yes — staging at `https://staging-middleware.avvanz.com`.

---

## 14. Glossary

| Term | Meaning |
|---|---|
| **Avvanz ScreenGlobal** | Avvanz web platform for background screening; the API middleware fronts it |
| **Case** | One screening request for one candidate/employee |
| **Package** | A named set of checks applied to a case; created by Avvanz for a company |
| **Check** | A single verification (e.g., Education Verification – Philippines) |
| **Branch** | A company sub-entity attached to a case |
| **Candidate / Employee** | The person being screened |
| **Requestor** | Person at the client who requested the case (`requestor_name`, `requestor_email`) |
| **BU** | Business unit |
| **Checkout / Order** | Step at which a case is ordered and priced; sets `order_ref_no` |
| **Final report** | Result document/data for a completed case |
| **Middleware** | The REST layer that hides ScreenGlobal's backend processes |

---

## 15. What Is NOT Documented (answer "unknown" and escalate to Avvanz)

- Token expiry, refresh, revocation
- Webhooks or push notifications
- Error response schema and full error-code catalogue
- Full list of case statuses, check statuses and result values
- How/where checkout (ordering and payment) is performed relative to the API
- Max `limit`, default sort, total count, date-range inclusivity
- Idempotency/duplicate handling for Create Case
- Ability to change package/country after creation
- File download for `candidate_files`/`verified_files` through the API
- Exact meaning of `candidate_data_entry = 2`
- Changelog / version history (the table is empty)
- SLAs, turnaround times, supported countries and check catalogue (see Get Packages / Case Price for what a given account has)

---

## 16. Source Map

| Topic | Source |
|---|---|
| Overview, use cases, onboarding, base URLs, rate limit, workflow chart | PDF |
| Endpoint paths, parameters, sample responses | PDF + Postman |
| Required fields summary, `branch_id` on packages, tested status codes (200/201), request encodings, variables (`base_url`, `access_token`, `case_id`, `case_ref_no`, `package_name`, `branch_name`, `date_from`, `date_to`, `sort`, `offset`, `limit`) | Postman |
| Known ambiguities flagged by the collection author (`candidate_data_entry`, `no_last_name`) | Postman collection description |
| Final-report JSON structure | PDF sample |