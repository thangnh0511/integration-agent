---
title: Paradox Public API 
type: paradox_api
status: verified
owner: Thang Nguyen
last_updated: 2026-10-28
vendor: Paradox
---


# Paradox Developer Hub — Knowledge Base (Source of Truth for AI Agent)

- **Source:** https://readme.paradox.ai/ (Paradox Developer Hub, API version v1.0)
- **Retrieved:** 2026-10-28
- **Scope of this file:** Every page linked from the Developer Hub navigation (API Reference, Guides, Changelog, Discussions) was read and is documented in Sections 1–8. Section 9 lists the few items that could not be read; do not guess those.
- **Rule for the AI Agent:** Only answer from this file. If an endpoint, field, or behavior is not documented here, say it is not covered and point to the source URL. Never invent parameters.

---

## 1. Overview

| Item | Value |
|---|---|
| Production base URL | `https://api.paradox.ai/api/v1/public` |
| Other environments | `https://<your_environment>.paradox.ai/api/v1/public` (prefix varies by environment) |
| Supported methods | GET, POST, PUT, PATCH (limited endpoints), DELETE |

### Response codes

| Code | Meaning |
|---|---|
| 200 | Request success |
| 400 | Bad request data |
| 401 | Authentication failed / not authenticated |
| 403 | Permission denied. Mostly session expired, non-logged-in call, or access to resources not owned |
| 404 | Invalid API URI |
| 429 | Too Many Requests — client exceeded the endpoint rate limit |
| 432 | Token expired |
| 500 | Internal server error |

Source: https://readme.paradox.ai/reference/overview

---

## 2. Authentication

Paradox public APIs require **OAuth 2.0 (preferred)** or **Basic Authentication**. Request your **account ID** and **API secret key** from the Paradox Integrations Team.

### Token endpoints by environment

| Environment | Token URL |
|---|---|
| Olivia DEV2 | `https://dev2api.paradox.ai/api/v1/public/auth/token` |
| Olivia TEST | `https://testapi.paradox.ai/api/v1/public/auth/token` |
| Olivia STG | `https://stgapi.paradox.ai/api/v1/public/auth/token` |
| Olivia PROD | `https://api.paradox.ai/api/v1/public/auth/token` |
| EU STG | `https://api.stg.eu1.paradox.ai/api/v1/public/auth/token` |
| EU PROD | `https://api.eu1.paradox.ai/api/v1/public/auth/token` |

### 2.1 OAuth 2.0 Token — `POST /auth/token`

- Header: `Content-Type: application/x-www-form-urlencoded`
- Body (form data):

| Field | Type | Required | Description |
|---|---|---|---|
| `grant_type` | string | yes | `client_credentials` (default) |
| `client_id` | string | yes | Account ID provided by Paradox |
| `client_secret` | string | yes | Secret Key provided by Paradox |

- Response: `access_token`, `expires_in` (seconds), `token_type`
- Use on every later request: `Authorization: Bearer <access_token>`
- Responses documented: 200, 400

### 2.2 Postman OAuth 2.0 setup

Authorization tab → type OAuth 2.0 → "Add authorization data to" = Request Headers. Header Prefix = `Bearer`; Grant Type = Client Credentials; Access Token URL = token URL above; Client ID = Account ID; Client Secret = Secret Key; Client Authentication = "Send client credentials in body". Click Get New Access Token → Proceed → Use Token.

### 2.3 Basic Authentication

Postman → Basic Auth. Username = Account ID, Password = API Secret.

### 2.4 Verify JWT — `POST /auth/verify_jwt_token`

Verifies a generated JWT token. **Only applicable to company-level API credentials**; confirm auth type with your Paradox integration resource.

| Body field | Type | Description |
|---|---|---|
| `account_id` | string | Account ID |
| `jwt_token` | string | JWT token |

Responses: 200, 400.

Sources: https://readme.paradox.ai/reference/authentication · /oauth-token · /verify-jwt

---

## 3. Language Codes

The `language_preference` field controls the language the system uses to communicate with a candidate or user. Default is `en` when omitted.

Two accepted formats:
1. **Standard codes** (BCP 47, **lowercase exactly as in table**). Regional variants use a hyphen (e.g. `es-mx`).
2. **Locale-style aliases** (e.g. `it_it`, `ja_jp`, `pt_br`), accepted for backward compatibility, **case-insensitive**, auto-mapped to the standard code.

### Standard codes

| Code | Language | Code | Language |
|---|---|---|---|
| en | English | zh-cn | Simplified Chinese |
| en-gb | British English | zh-tw | Traditional Chinese |
| es | Spanish (LATAM) | ja | Japanese |
| es-em | Spanish (Modern Spain) | ko | Korean |
| es-mx | Spanish (Mexico) | vi | Vietnamese |
| fr | French | th | Thai |
| fr-ca | French (Canadian) | tr | Turkish |
| de | German | ru | Russian |
| it | Italian | ar | Arabic |
| nl | Dutch | he | Hebrew |
| pt | Portuguese (Portugal) | hu | Hungarian |
| pt-br | Brazilian Portuguese | pl | Polish |
| ro | Romanian | bg | Bulgarian |
| bs | Bosnian | hr | Croatian |
| cs | Czech | el | Greek |
| sk | Slovak | sr | Serbian (Cyrillic) |
| uk | Ukrainian | da | Danish |
| sv | Swedish | ms | Malay |
| id | Indonesian | km | Khmer |

### Locale aliases → standard code

`it_it→it`, `ja_jp→ja`, `ko_kr→ko`, `pt_br→pt-br`, `zh_cn→zh-cn`, `zh_tw→zh-tw`, `tr_tr→tr`, `en_us→en`, `fr_fr→fr`, `ru_ru→ru`, `de_de→de`, `fr_ca→fr-ca`, `nl_nl→nl`, `es_mx→es-mx`. (Also accepted as-is: `es`, `hu`, `vi`, `bs`, `he`.)

Source: https://readme.paradox.ai/reference/language-codes

---

## 4. Company

All requests require header `Authorization: Bearer <oauth_token>`.

| Endpoint | Method & Path | Returns |
|---|---|---|
| Get Conversations | `GET /company/conversations` | All conversations of the company |
| Get Groups | `GET /company/groups` | All groups of the company |
| Get AI Assistant | `GET /company/ai` | AI assistant's name and image |

### Get Conversations — response keys

| Key | Description |
|---|---|
| `updated_at` | Last timestamp conversation was updated |
| `created_at` | Timestamp conversation was created |
| `oid` | Encrypted id of conversation |
| `label` | Name of the conversation |
| `is_remote` | `1` if not assigned to any location |
| `deactivated` | `1` if conversation is deactivated |

### Get Groups — response keys
`oid` (encrypted group id), `name` (group name).

### Get AI Assistant — response keys
`id` (company id), `ai_name` (assistant name), `ai_logo_uri` (assistant image URI).

Sources: /get-company-conversations · /get-groups · /get-ai-assistants-name-and-image

---

## 5. Candidates

Base path: `/candidates`

| Endpoint | Method & Path |
|---|---|
| Get Candidates | `GET /candidates` |
| Create Candidate | `POST /candidates` |
| Get Single Candidate | `GET /candidates/{ID}` |
| Update Candidate | `PUT /candidates/{ID}` |
| Delete Candidate | `DELETE /candidates/{ID}` |
| Unsubscribe Candidate | `PUT /candidates/unsubscribe` |
| Scheduling Shortlist Review | `POST /scheduling/communication` |
| Send Candidate Message | `POST /candidates/send_message` |
| Create Candidate Documents | `POST /candidates/{OID}/documents` |

### 5.1 Get Candidates — `GET /candidates`

Returns the full JSON of candidates. By default all candidates are sent; limit with `limit`.

**Query parameters**

| Param | Type | Default | Notes |
|---|---|---|---|
| `start_date` / `end_date` | string | | Epoch time or `YYYY-MM-DDTHH:MM:SS` |
| `start_keyword` | string | | |
| `limit` | int32 | 50 | Number of candidates returned ("should less than 50 to test") |
| `offset` | int32 | 0 | |
| `page` | string | | Pagination |
| `status` | int32 | 1 | See status values below |
| `group_name` | string | | |
| `location_id` | int32 | | |
| `source` | int32 | 1 | See source values below |
| `conversation` | boolean | false | Include conversation |
| `note` | boolean | false | Include notes |
| `interviews` | int32 | 0 | Set `1` to include interview information |
| `profile_id` | string | | Candidate's profile id |
| `include_attributes` | string | | Yes/No |
| `candidate_journey_status` | string | | Format `Stage: Status`, e.g. `Capture: Capture Complete` |
| `created_start_date` | string | | `YYYY-MM-DD` |
| `job_loc_code` | string | | Filter by job location code |
| `job_req_id` | string | | External or internal job request ID |
| `ex_id` | string | | Filter by `ex_id_1` (3rd-party identifiers) |
| `email` | string | | Must be ASCII-encoded; special chars such as `+` must be percent-encoded |

**Response keys**

*Candidate*

| Key | Description |
|---|---|
| `start_keyword` | Keyword used by candidate to start the conversation |
| `phone_number` | Phone number |
| `formatted_number` | Phone in E.164 format (e.g. +12144930945) |
| `email` | Email |
| `name` / `edited_name` | Full name / edited name |
| `OID` | Id of the Olivia candidate |
| `external_oid` | Encrypted id of the Olivia candidate |
| `source` | SMS=1, WIDGET=2, RECRUITER_PAGE=3, COMPANY_PAGE=4, EMBEDDED_WIDGET=5, WIT_WIDGET=6 |
| `group_name` | Candidate's group |
| `referrer_email`, `referrer_name`, `external_referrer` | Referrer info |
| `language_preference` | Language code |
| `created_at`, `updated_at` | Timestamps |

*Conversation* (when `conversation=true`)

| Key | Description |
|---|---|
| `media_url` | URL of image/video sent by candidate |
| `message_type` | FREE_TEXT=1, TK_TEXT=2, ASK_ANY_THING_TEXT=3, ASK_CONTINUE_TEXT=4, PROMPT_NAME=5, PROMPT_EMAIL=6, PROMPT_PHONE=7, ASK_DATA=8, ASK_DOCUMENT=9, ASK_LIST_SELECT=10 |
| `answer_text` | Answer from Olivia / company user |
| `question_text` | Text asked by the candidate |
| `from_who` | `0` = AI; otherwise id of the company_user |
| `created_at`, `updated_at` | Timestamps |

*Interview* (when `interviews=1`)

| Key | Description |
|---|---|
| `past_interview_at` | Timestamp of last completed interview |
| `scheduled_at` | Timestamp of scheduled interview |
| `interviewed` | Timestamp of the interview action with the candidate (in Get Candidates); in Update Candidate docs described as "if 1, candidate was already interviewed" |
| `scheduled_by` | Id of company user who scheduled |
| `interview_location_id` | Location ID of the interview |

*Stage*

| Key | Description |
|---|---|
| `viewed` | `1` if viewed by a company user |
| `completed` | `1` if candidate completed the conversation |
| `is_live` | `1` if candidate is in a live conversation |
| `responded` | `1` if responded to by a company user |
| `status` | APPROVED=1, PENDING=2, ARCHIVED=3, DRAFT=4, SPAM=5, UNKNOWN=7, SCHEDULED=8, CONFIRMED=9, REMOVED=10 |
| `interview_stage` | NONE=1, PENDING=2, SCHEDULED=3, SCHEDULE=4, PENDING_INVITE=5, PENDING_OPT_IN_REQUEST=6, PENDING_INVITE_SMS_BS_HOUR=7, NO_AVAILABILITY=8, CANCELED=9, CANCELED_REQUEST=10 |

*Note* (when `note=true`): `text`, `created_at`.

### 5.2 Create Candidate — `POST /candidates`

**Body parameters** (char limits as documented)

| Field | Type | Req | Notes |
|---|---|---|---|
| `name` | string | yes | Full name (limit 50) |
| `first_name`, `last_name` | string | cond. | Required only if `name` missing; combined limit 50 |
| `phone` | string | cond. | Required if email not provided. Country code required (limit 20) |
| `email` | string | cond. | Required if phone not provided (limit 50) |
| `ex_id` | string | | External candidate ID (255) |
| `hirevue_link`, `pymetrics_link` | string | | Virtual interview links (255) |
| `hirevue_instructions` | string | | (255) |
| `ex_step`, `ex_status`, `ex_reason` | string | | External step/status/reason (no limit) |
| `job_req_id`, `job_title` | string | | (255 each) |
| `job_loc_code` | string | | Max length 20 |
| `primary_contact_method` | string | | 1=Email, 2=SMS, 3=Email & SMS, 4=WhatsApp & Email, 5=WhatsApp. Default Email & SMS |
| `hired_date` | date-time | | |
| `adp_link` | string | | (255) |
| `audience_type` | string | | `Blue Collar` \| `White Collar` \| `Blue Collar & White Collar` |
| `hm_cid` | string | | Hirescore candidate ID (255) |
| `external_group_id` | string | | |
| `referrer_email`, `referrer_name` | string | | (50 each) |
| `recruiter_email` | string | | Assigned recruiter |
| `hiring_manager` | string | | `hiring_email` functions the same |
| `external_referrer` | string | | (1000) |
| `language_preference` | string | | Default `en` — see Section 3 |
| `candidatedata` | file | | Report file for bulk create/update |
| `consent_to_marketing` | string | | |
| `resume` | string | | Binary/Base64: .pdf .doc .docx .png .jpg .txt; must be < 10 MB |
| `candidate_journey` | string | | Journey name |
| `candidate_journey_status` | string | | `{stage_name}: {status_name}` |
| `candidate_attribute_data` | json | | e.g. `{"key_name":"value"}`; custom attribute values limit 255 |
| `note` | string | | |
| `job_application_id` | string | | (255) |
| `use_application_id_for_identity` | boolean | false | To identify returning candidates |
| `candidate_location_info` | string | | (100) |
| `external_source_id` | string | | (255) |
| `offer_letter` | string | | Base64: .doc .docx .pdf .png .jpg .jpeg .txt .pages .HTML (payload >10 MB noted) |
| `offer_file_name` | string | | (150) |
| `use_paradox_status_map` | boolean | false | If true, `status_map_name` required |
| `status_map_name` | string | | Journey Status Mapping name in Paradox |
| `status_map_ex_id` | string | | External-system status ID mapped to a Paradox Journey Status |
| `talent_community` | boolean | false | Add to Talent Community |
| `community_of_interest` | int32 | | Which community, when `talent_community=true` |
| `skip_send_opt_in` | boolean | | Only for CRM and when `talent_community=true` |
| `is_internal_job_feed` | boolean | false | Candidate applying to internal job; omitted = external job |

Responses: 200, 400.

### 5.3 Get Single Candidate — `GET /candidates/{ID}`

- Path `ID` (required): either `OID` or `external_oid`.
- Query: `conversation` (bool, default false), `note` (bool, default false).

### 5.4 Update Candidate — `PUT /candidates/{ID}`

- Path `ID` (required): `OID`, `external_oid`, or `ex_id`.
- Body fields (all optional in docs): `phone`, `email`, `name`, `first_name`, `last_name` (use first+last if no `name`), `status` (APPROVED=1, PENDING=2, ARCHIVED=3, DRAFT=4, SPAM=5), `location`, `hirevue_link`, `pymetrics_link`, `ex_step`, `ex_status`, `ex_reason`, `job_req_id`, `job_title`, `primary_contact_method` (1 Email, 2 SMS, 3 Email & SMS, 4 WhatsApp & Email), `hired_date`, `adp_link`, `audience_type`, `hm_cid`, `external_group_id`, `hirevue_instructions`, `referrer_email`, `referrer_name`, `external_referrer`, `language_preference` (default `en`), `resume` (Base64), `candidate_journey`, `candidate_journey_status`, `candidate_attribute_data`, `note`, `candidate_location_info`, `external_source_id`, `offer_letter`, `offer_file_name`, `use_paradox_status_map`, `status_map_name`, `status_map_ex_id` (the last two required when `use_paradox_status_map=true`).
- Response keys: same structure as Get Candidates (Section 5.1).

### 5.5 Delete Candidate — `DELETE /candidates/{ID}`
Path `ID` (required): `OID` or `external_oid`.

### 5.6 Unsubscribe Candidate — `PUT /candidates/unsubscribe`

| Body | Type | Req | Description |
|---|---|---|---|
| `OID` | string | yes | `OID` or `external_oid` |
| `action_id` | int32 | | Unsubscribe = `1` (default), Resubscribe = `0` |

Returns `success = True` on success; otherwise `success = False` plus an error.

### 5.7 Scheduling Shortlist Review — `POST /scheduling/communication`

Sends an email to Hiring Manager(s) with a shortlist of candidates for review. Returns `success = True` or `success = False` with an error.

| Body | Type | Req | Description |
|---|---|---|---|
| `message_id` | string | | Message identifier |
| `to` | string | yes | Hiring Manager Workday user IDs, e.g. `{"wids": "id1, id2"}` |
| `email_title` | string | yes | Email subject |
| `email_text` | string | yes | Email body |
| `action_link` | string | yes | Link to HiredScore to view all candidates |
| `sms_text`, `action_text`, `template` | string | | "Future implementation" |

Responses: 200, 400, 401, 500.

### 5.8 Send Candidate Message — `POST /candidates/send_message`

| Body | Type | Req | Description |
|---|---|---|---|
| `OID` | string | yes | `OID` or `external_oid` |
| `message` | string | yes | HTML or plain text |
| `send_as` | int32 | | `1` = send as Olivia (default); a user UID = send as that recruiter |
| `contact_method` | int32 | | 1 Email, 2 Phone. Default prioritizes Phone over Email |
| `email_subject` | string | | Subject for email |

Returns `success = True` or `False` with error. Responses: 200, 400, 401, 500.

### 5.9 Create Candidate Documents — `POST /candidates/{OID}/documents`
Path `OID` (required). Body: `documents` (required, array of document objects). The page does not document the object's inner fields. Auth: OAuth2 Bearer. Response: 200.

---

## 6. Users

Base path: `/users`

| Endpoint | Method & Path | Status in this file |
|---|---|---|
| Get users | `GET /users/` | Documented |
| Get Single User | `GET /users/{OID}` | Documented |
| Get User Roles | `GET /user-roles` | Documented |
| Create User | `POST /users` | Documented |
| Update User | `PUT /users/{OID}` | Documented |
| Partially Update User | `PATCH /users/{OID}` | Documented |
| Delete User | `DELETE /users/{OID}` | Documented |
| Get Single User by employee_id | `GET /users/employees/{employee_id}` | Documented (6.8) |
| Update User by employee_id | `PUT /users/employees/{employee_id}` | Documented (6.8) |
| Deactivate User | `POST /users/{OID}/deactivate` | Documented (6.8) |
| Reactivate User | `POST /users/{OID}/reactivate` | Documented (6.8) |
| Delete User by employee_id | `DELETE /users/employees/{employee_id}` | Documented (6.8) |

### 6.1 Get users — `GET /users/`
Query: `limit` (records per page), `page`, `include_campus_permission` (bool), `external_role_id` (must be used together with `location_id`), `location_id`.

Response keys: `full_name`, `OID`, `company_id`, `phone_number`, `user_email`, `formatted_number`, `language_preference`, `campus_permission_view`, `campus_permission_approve`, `campus_entitlements`, `employee_id`, `external_role_id`, `country_code`.

### 6.2 Get Single User — `GET /users/{OID}`
Path `OID` = user id or ex_id. Query `include_campus_permissions` (int32, default 0; 1 = load, 0 = don't). Response keys: `full_name`, `OID`, `company_id`, `phone_number`, `user_email`, `formatted_number`, `language_preference`, `campus_permission_view`, `campus_permission_approve`, `campus_entitlements`.

### 6.3 Get User Roles — `GET /user-roles`
Query: `limit` (int32, default 0), `page`. Response keys: `uuid`, `name`, `external_id`, `is_default`.

### 6.4 Create User — `POST /users`
Authorization header is required.

| Field | Type | Req | Notes |
|---|---|---|---|
| `name` | string | yes | Full name |
| `phone_number` | string | yes | |
| `email` | string | yes | |
| `login_email` | string | | Login email, unique across users; derived from `email` if omitted |
| `role` | int32 | | Role id |
| `user_type` | int32 | | Full=0, Basic=1 |
| `prod_access_levels` | int32[] | | Candidate Experience Manager=1, Student Experience Manager=2; e.g. `[1,2]` |
| `timezone` | string | | tz database name |
| `language_preference` | string | | default `en` |
| `campus_permissions` | object[] | | School area id + access type |
| `campus_entitlements` | string[] | | Entitlement values; assigns campus permissions if `campus_permissions` not used |
| `employee_id` | string | | External employee id; not blank, unique within company |
| `job_title` | string | | |
| `location_ids` | string[] | | Location OIDs from Get Locations |
| `country_code` | string | | ISO 3166-2 |
| `area_ids` | string[] | | External area IDs configured in UI |
| `location_codes` | string[] | | Location external codes |
| `external_role_id` | string | | Custom user role id |

### 6.5 Update User — `PUT /users/{OID}`
Path `OID` = user id or ex_id. Body: `name` (**required**), `phone_number`, `email`, `role`, `user_type`, `prod_access_levels`, `timezone`, `language_preference`, `employee_id`, `job_title`, `location_ids`, `country_code`, `area_ids`, `external_role_id` (External ID of Roles and Permissions).

### 6.6 Partially Update User — `PATCH /users/{OID}`
Path `OID` = user's **external oid**. Body (all optional): `name` (255), `employee_id`, `job_title`, `role`, `type` (Full=0, Basic=1), `timezone`, `language_preference`, `fname`, `mname`, `lname` (combined limit 255).

### 6.7 Delete User — `DELETE /users/{OID}`
Path `OID` = user id or ex_id. Returns `success = true`; otherwise error code `1005`, message "Not found."

---

## 6A. User Permission

Base path: `/user_permission`. `Authorization: Bearer <oauth_token>` is required on all three.

| Endpoint | Method & Path |
|---|---|
| Add Location Permission | `PUT /user_permission/add_location_permission` |
| Get Location Permissions | `GET /user_permission/get_location_permissions` |
| Delete Location Permission | `DELETE /user_permission/delete_location_permission` |

Shared reference params: `user_reference` (default 1): `1` = encrypted user id, `2` = employee id. `location_reference` (default 1): `1` = encrypted job location id, `2` = job location code.

**Add Location Permission — body:** `user_id` (required; encrypted user id or employee id), `user_reference`, `location_id` (required; encrypted job location id or code), `location_reference`, `group_ids` (required; array of encrypted group ids). Response: `success` (true/false), `location` object (`oid`, `name`).

**Get Location Permissions — query:** `user_id` (required), `user_reference`. Response: list of job locations with `oid`, `name`.

**Delete Location Permission — body:** `user_id` (required), `user_reference`, `location_id` (required), `location_reference`. Returns `success = true` on success.

---

## 6B. Scheduling (Interview)

Base path: `/interview`.

| Endpoint | Method & Path |
|---|---|
| Get multiparty interviewers | `GET /interview/mp_list_interviewers` |
| Get interview setting | `GET /interview/get_setting` |
| Get job location rooms | `GET /interview/get_job_loc_rooms` |
| Send interview alerts | `PUT /interview/interview_alerts` |
| Get interview history | `GET /interview/interview_history` |

**Interview type enum** (`type` / `interview_type`): IN_PERSON_INTERVIEW=1, PHONE_INTERVIEW=2, IN_PERSON_MEETING=3, PHONE_MEETING=4, SEQUENTIAL_INTERVIEW=5, SEQUENTIAL_MEETING=6, VIRTUAL_INTERVIEW=7, VIRTUAL_MEETING=8, GROUP_INTERVIEW=9, PHONE_CONVERSATION=10, IN_PERSON_CONVERSATION=11, VIRTUAL_CONVERSATION=12, IN_PERSON_SESSION=13, PHONE_SESSION=14, SEQUENTIAL_SESSION=15, VIRTUAL_SESSION=16, GROUP_SESSION=17, INTERVIEW_PREFERENCE=18, BREAK=19 (BREAK only listed on Send interview alerts).

### 6B.1 Get multiparty interviewers
- **Authorization header documented as `jwt <account_id>:<jwt_token>`** (differs from the Bearer format used elsewhere — follow the page).
- Query: `OID` (int64, required, candidate id), `interview_type` (int32, required, default 1), `jobloc_id` (int32), `load_joblocs` (int32, default 1), `load_rr_groups` (int32, default 1).
- Response keys: `interviewers` (list: `name`, `user_id`, `avatar_uri`, `instructions`, `instructions_phone`, `instructions_virtual`, `is_basic`, `is_owner`, `jobloc_ids`), `jobloc_total`, `joblocs` (list: `jobloc_id`, `name`, `addr_1`, `addr_2`, `city`, `admin1_name`, `admin1_code`, `zip_code`), `include_owner`, `rr_groups` (Round Robin groups: `group` {`id`, `name`, `company_id`}, `users` [{`id`, `user_id`, `fname`, `lname`, `name`, `alias`, `area_code`, `phone_number`, `slug`, `ex_source`, `created_at`, `updated_at`}]).
- `ex_source`: DEFAULT=0, TENSTREET=1, SMASHFLY=2, WEBHOOK=3, SENDOUTS=4, APPLICANTSTACK=5, ICIMS=6, PCRECRUITER=7, AVATURE=8, CUSTOM=9.

### 6B.2 Get interview setting
- Query: `OID` (int64, required, candidate long id), `company_id` (int32, required), `name` (string, required), `phone`, `email`.
- Response keys: `duration`, `company_name`, `jobloc_id`, `interview_types`, `instructions`, `instructions_virtual`, `instructions_phone`, `interview_instructions`, `type`, `lead_name`, `lead_id`, `lead_long_id`, `areas`, `interviewers`, `jobloc_room_id`, `scheduled_at`, `wait_schedule` (False = request sent, True = not yet).

### 6B.3 Get job location rooms
- Query: `jobloc_id` (int32, required). Response keys: `id`, `name`, `job_loc_id`, `seats`, `deleted`.

### 6B.4 Send interview alerts — `PUT /interview/interview_alerts`
Lets 3rd-party integrators send Interview Requests to Paradox; also **creates candidates that do not exist**.

> **Prerequisite:** In Paradox go to Settings → Client Setup → Integrations and enable **"Enable Validation for Interview Alerts Scheduling"** so new public API scheduling requests are validated (optional attendees, attendee counts, required roles, related rules). Marking interviewers as optional via objects requires this toggle.

Key body fields:

| Field | Type | Notes |
|---|---|---|
| `OID` | int32 | Paradox candidate application ID (max 20) |
| `ex_id` | string | External candidate ID (255) |
| `job_application_id` | string | External job application ID (255) |
| `use_application_id_for_identity` | boolean | Use application ID as unique identifier; allows creating and scheduling candidates from this endpoint |
| `action` | string, **required** | `interview` (send times to candidate) \| `reschedule` \| `cancel` (max 10); default `interview` |
| `interview_duration` | int32, **required** | Minutes; **min 5, max 415** |
| `interview_type` | int32, **required** | See enum above; default 0 |
| `custom_interview_type` | string | API ID of the custom interview type |
| `interview_segments` | object[] | Only with a sequential interview type |
| `interviewer_emails` | object[] | Comma-separated list or array (255) |
| `interviewer_ex_ids` | object[] | External employee IDs |
| `interview_team` | string | Dynamic round-robin from user emails or employee IDs; **max 25 interviewers; priority High** |
| `alternate_interview_team` | string | Same format; **priority Medium**; max 25 |
| `interviewer_ids` | object[] | e.g. `["u_636","u_74","g_6","exg_1089"]` = users 636, 74, round-robin group 6, external group 1089 |
| `interview_jobloc_code` | string | External location ID |
| `interview_jobloc_id` | int32 | Paradox location ID |
| `interview_jobloc_room_id` | int32 | Paradox room ID; **`-1` = "Any Available Room"** |
| `interview_jobloc_room_ex_id`, `interview_jobloc_room_email` | string | Room external id / email |
| `interview_location_address` | string | Any valid street address (need not exist in Paradox) |
| `interview_instructions` | string | Max 320 |
| `candidate_interview_prep_ex_id`, `recruiter_interview_prep_ex_id` | string | (255) |
| `interview_owner_email`, `interview_owner_ex_id` | string | Defaults to the user tied to API credentials |
| `generate_virtual_url` | boolean | default false |
| `private` | int32 | 0 Public, 1 Private; account default if omitted |
| `first_name`, `last_name` | string | (50) |
| `phone` | int32 | (20) |
| `email` | string | (50) |
| `job_title` | string (255), `job_req_id` (50), `job_loc_code` (20) | |
| `candidate_journey`, `candidate_journey_status` | string | `{stage}: {status}` |
| `sequential_order_automation` | int32 | default -1; (In Order Only)=0, (In Order, Any Order)=1, (In Order, Any Order, Multi-Day)=2, (Multi-Day Only)=3 |
| `force_sequent_multi_days` | boolean | default false |
| `recorded_interview_id` | string | External id of recorded interview |
| `resume` | string | Base64, < 10 MB |
| `note` | string | Creates candidate note |
| `notes_data` | string | Only for Kenexa requests |
| `attendee_count` | string | Number of attendees for all Round Robin Groups and Interview Team |
| `use_paradox_status_map`, `status_map_name`, `status_map_ex_id` | | Same behavior as Create Candidate |
| `primary_contact_method` | string | 1 Email, 2 SMS, 3 Email & SMS, 4 WhatsApp & Email, 5 WhatsApp |
| `debrief` | object | All child properties required for a debrief |

Response when `success = 1`: `interview_data`, `recruiter_text` (message sent to candidate), `processed` (1 = alert sent, 0 = not yet), `company_id`, `candidate` (OID). When `success = 0`: `error` message.

### 6B.5 Get interview history
- Query: `OID` (int64, required), `limit` (default 10), `offset` (default 0).
- Response keys: `OID`, `interview_history` (list), `itv_details` (sub-interviews when sequential), `interview_status`, `scheduled_at`, `interview_kind` (single / sequential / event / recorded), `interview_type`, `original_interview_type` (for custom types), `attendees` (`name`, `email`, `phone_number`, `avatar_uri`), `location` (`id`, `name`, `country`, `admin1_name`, `admin1_code`, `City`, `latitude`, `longitude`).

---

## 6C. Candidate Attributes

Base path: `/candidate/attributes/{OID}` where `OID` = **candidate long ID (int64)**.

| Endpoint | Method | Behavior |
|---|---|---|
| Get Candidate Attributes | `GET` | Returns `candidate_id` (long_id) and `attributes` (JSON object) |
| Patch Candidate Attributes | `PATCH` | Merges the passed attributes with current ones |
| Update Candidate Attributes | `PUT` | **Replaces all existing attributes** with the payload |

Body field documented on both PATCH and PUT: `candidate_journey_status` (string, format `<stage name>: <status name>`). The pages do not list other attribute keys.

---

## 6D. Reporting

Base path: `/reporting/reports`. OAuth2 Bearer.

| Endpoint | Method & Path |
|---|---|
| Get Report List | `GET /reporting/reports` |
| Create Report | `POST /reporting/reports` |
| Get Single Report | `GET /reporting/reports/{ID}` |

- **Get Report List** response keys: `description`, `available_fields` (field-to-report-file mapping is under "Report Field Documentation", which was not read), `can_filter` ("currently not in use but will be enabled in future").
- **Create Report** body: `callbackUrl` (string, **required**, invoked when report finishes), `report` (report name, e.g. `CAPTURE_CANDIDATE_SPECIFIC`), `from_date`, `to_date` (`YYYY-MM-DD`). Response: `id` (report run instance id), `status` (`create` = queued), `callbackUrl`, `url` (no value yet).
- **Get Single Report**: path `ID` = report instance id from Create Report. `status` is `create` while queued and `ready` when the file is available; `url` is the CSV download link, **valid for 48 hours**.
- Flow: list reports → create report with `callbackUrl` → wait for callback or poll Get Single Report until `ready` → download CSV from `url` within 48 h.

---

## 6E. Location Areas

Base path: `/areas`. OAuth2 Bearer.

| Endpoint | Method & Path | Success code |
|---|---|---|
| Get Area List | `GET /areas` | 200 |
| Get Single Area | `GET /areas/{OID}` | 200 |
| Create an area | `POST /areas` | **201** |
| Update an area | `PUT /areas/{OID}` | 200 |
| Delete an area | `DELETE /areas/{OID}` | **204** |

Body for Create/Update: `name` (string, **required**), `parent_id` (string), `external_area_id` (string). Path `OID` = the area's oid. Response schemas for the GET endpoints are not documented on the pages.

---

## 6F. Locations

> Note: **Get Locations** lives under `/company/locations`; the other location endpoints use `/locations` (no `/company/`). OAuth2 Bearer on all.

| Endpoint | Method & Path | Success |
|---|---|---|
| Get Locations | `GET /company/locations` | 200 |
| Get Single Location | `GET /locations/{OID}` | 200 (404 if not found) |
| Create a location | `POST /locations` | 201 |
| Update a location | `PUT /locations/{OID}` | 200 |
| Deactivate a location | `POST /locations/{ID}/deactivate` | 204 |
| Get Single Location by job_loc_code | `GET /locations?job_loc_code=...` | 200 (404 if not found) |
| Update Location by job_loc_code | `PUT /locations/job_loc/{job_loc_code}` | 200 |
| Delete a location | `DELETE /locations/{OID}` | 204 — **DEPRECATED** |

**Get Locations — response keys:** `oid` (encrypted id), `location_id`, `name`, `addr_1`, `addr_2`, `zip_code`, `latitude`, `longitude`, `phone_number` (AI phone number assigned to the location), `status` (**1 Active, 2 Pending, 3 Deleted**), `conversation_id`, `is_remote` (`1` = remote), `created_at`, `updated_at`. Returns all locations by default.

**Get Single Location:** path `OID` = the location's external oid. **By job_loc_code:** query `job_loc_code` (required).

**Create a location — body:** `name` (required), `state_code` (required), `addr_1`, `addr_2`, `city`, `zip_code`, `time_zone`, `email`, `phone_number`, `job_loc_code` (**only Company Administrators can edit `job_loc_code`**), `parent_id` (id of the area the location belongs to), `custom_locations_attributes` (JSON, values max 255 chars, e.g. `{"key_name":"value"}`).

**Update a location — body:** same fields as create, all optional; `job_loc_code` requires admin or company staff. Path `OID` = location's external oid.

**Deactivate a location:** deactivates without hard-deleting; **deactivated locations no longer accept new applicants**. Path `ID` accepts either `job_loc_code` or internal OID. Success 204.

**Update Location by job_loc_code:** path `job_loc_code` (required). The page lists no body fields; assume the same fields as Update a location only after verifying.

**Delete a location:** deprecated — "Deletion has been replaced by Deactivation". Use Deactivate instead.

---

## 6G. Location Rooms

Base path: `/rooms`. OAuth2 Bearer.

| Endpoint | Method & Path | Success |
|---|---|---|
| Get Room List | `GET /rooms?location_id={loc_id}` | 200 |
| Get Single Room | `GET /rooms/{room_id}` | 200 |
| Create a room | `POST /rooms` | 201 |
| Update a room | `PUT /rooms/{room_id}` | 200 |
| Delete a room | `DELETE /rooms/{room_id}` | 204 |

- **Get Room List:** `location_id` is required; without it the API returns a 400 text response "No Location ID Provided". Header `accept` may be `application/json` (default) or `text/plain`.
- **Get Single Room:** `room_id` = Room OID.
- **Create a room — body:** `name` (required), `location_id` (required; Location OID of the location the room is in), `seats` (required, int32; 400 "Missing Seats" if omitted), `cal_email` (optional; at least one calendar must be active in the system for this field to work; email domain is validated against active calendars; 400 "Validate Email" on failure), `room_ex_id` (optional; unique identifier for the room **within a location** — same value may exist in different locations but not twice in the same location).
- **Update a room — body:** `name` (required), `seats` (required), `location_id`, `cal_email`, `room_ex_id` (same rules as create).
- **Delete a room:** path `room_id` (required).

---

## 6H. Guides: IFrame Usage

The only Guides pages are "Getting Started with Paradox" (links back to the API Overview) and **IFrame Usage**. IFrame Usage describes embeddable Paradox modals hosted on `olivia.paradox.ai`. All require `jwt_token` (token generated per Verify JWT / company-level credentials) and `account_id`.

| Modal | URL | Parameters |
|---|---|---|
| Demo page | `https://olivia.paradox.ai/demo/sf-iframes` | `OID`, `jwt_token`, `account_id` |
| Conversations | `https://olivia.paradox.ai/external/convo?OID={OID}&jwt_token={jwt_token}&account_id={account_id}` | `OID` (candidate long ID), `jwt_token`, `account_id` |
| Scheduling | `https://olivia.paradox.ai/external/scheduling?OID={OID}&name={name}&email={email}&phone={phone}&jwt_token={jwt_token}&account_id={account_id}` | `OID`, `jwt_token`, `account_id`, `name`, `email`, `phone`; also `ContactID`, `ContactDBID`, `ClientID`, `is_show_conversation` (marks lead as read when from iframe conversation) |
| Settings | `https://olivia.paradox.ai/external/settings?jwt_token={jwt_token}&account_id={account_id}` | `jwt_token`, `account_id` |

(The source page's example URLs contain a sample token and account id; they are intentionally not reproduced here.)

---

## 6I. Changelog and Discussions

- **Changelog:** contains a single entry, "Welcome to Paradox" (almost 9 years old at retrieval). No technical content.
- **Discussions:** community Q&A, **not authoritative documentation**. Topics seen: message templates, SSO setup, OData `$metadata`/`$count`, adding job sources (Indeed, LinkedIn), Reporting API error, webhooks, rate limits, pagination parameters. The "Rate limitation" thread has **no answer**, so numeric rate limits remain undocumented. Do not treat discussion posts as official guidance.

---

## 7. Enumerations quick reference

| Enum | Values |
|---|---|
| Candidate `source` | SMS=1, WIDGET=2, RECRUITER_PAGE=3, COMPANY_PAGE=4, EMBEDDED_WIDGET=5, WIT_WIDGET=6 |
| Candidate `status` | APPROVED=1, PENDING=2, ARCHIVED=3, DRAFT=4, SPAM=5, UNKNOWN=7, SCHEDULED=8, CONFIRMED=9, REMOVED=10 |
| `interview_stage` | NONE=1, PENDING=2, SCHEDULED=3, SCHEDULE=4, PENDING_INVITE=5, PENDING_OPT_IN_REQUEST=6, PENDING_INVITE_SMS_BS_HOUR=7, NO_AVAILABILITY=8, CANCELED=9, CANCELED_REQUEST=10 |
| `message_type` | FREE_TEXT=1, TK_TEXT=2, ASK_ANY_THING_TEXT=3, ASK_CONTINUE_TEXT=4, PROMPT_NAME=5, PROMPT_EMAIL=6, PROMPT_PHONE=7, ASK_DATA=8, ASK_DOCUMENT=9, ASK_LIST_SELECT=10 |
| `primary_contact_method` | 1 Email, 2 SMS, 3 Email & SMS, 4 WhatsApp & Email, 5 WhatsApp (5 only listed in Create) |
| `user_type` | Full=0, Basic=1 |
| `prod_access_levels` | Candidate Experience Manager=1, Student Experience Manager=2 |
| Unsubscribe `action_id` | Unsubscribe=1, Resubscribe=0 |
| Interview `type` | 1 IN_PERSON_INTERVIEW … 18 INTERVIEW_PREFERENCE, 19 BREAK (full list in 6B) |
| Interview alert `action` | interview, reschedule, cancel |
| `user_reference` | 1 encrypted user id, 2 employee id |
| `location_reference` | 1 encrypted job location id, 2 job location code |
| Report `status` | create (queued), ready |
| Location `status` | 1 Active, 2 Pending, 3 Deleted |

---

### 6.8 Employee-id and activation endpoints

**Get Single User by employee_id** — `GET /users/employees/{employee_id}`. Path `employee_id` (required). Responses: 200, 400. The page documents no response keys.

**Update User by employee_id** — `PUT /users/employees/{employee_id}`. Path `employee_id` (required).

| Body | Type | Req | Notes |
|---|---|---|---|
| `name` | string | yes | Full name |
| `phone_number`, `email` | string | | |
| `role` | int32 | | "in range from 0 to 9" |
| `user_type` | int32 | | Full=0, Basic=1 |
| `prod_access_levels` | int32[] | | Candidate Experience Manager=1 and/or Student Experience Manager=2 |
| `timezone` | string | | |
| `language_preference` | string | | See Section 3 |
| `external_role_id` | string | | **Roles revert to default if this is not included in the payload** |
| `area_id` | string[] | | External area IDs configured in UI |
| `location_codes` | string[] | | Job locations' external ids |

**Deactivate User** — `POST /users/{OID}/deactivate` (path `OID` = user id or ex_id). Response key: `activate_status` (status of the user).

**Reactivate User** — `POST /users/{OID}/reactivate` (path `OID` = user id or ex_id). Response key: `activate_status`.

**Delete User by employee_id** — `DELETE /users/employees/{employee_id}`. Responses: 200, 400.

---

## 8. Agent usage guidance (derived from the docs above)

1. **Authenticate first:** call the token endpoint for the correct environment (PROD vs EU PROD, etc.), then send `Authorization: Bearer <token>` on every call. Token expiry returns 432; re-request a token.
2. **Identifier rules differ per endpoint:** candidates accept `OID` / `external_oid` (Update also accepts `ex_id`); users accept user id or `ex_id` (PATCH uses the external oid). Use the identifier named on the specific endpoint.
3. **Create Candidate requires `name` plus at least one of `phone` or `email`.** Phone must include country code.
4. **Encode emails properly** in the Get Candidates `email` filter (ASCII, percent-encode `+`).
5. **Language:** send lowercase standard codes (e.g. `pt-br`); underscore aliases are tolerated but standard codes are preferred.
6. **Rate limits:** respect 429 responses; the docs do not state numeric limits.
7. **Destructive endpoints** (Delete Candidate, Delete User) — confirm with the requester before calling.
8. **Interview alerts:** enable "Validation for Interview Alerts Scheduling" first; duration must be 5–415 minutes; `-1` room id = any available room.
9. **Auth header differs on Get multiparty interviewers** (`jwt <account_id>:<jwt_token>`).
10. **Update Candidate Attributes (PUT) replaces everything; PATCH merges.** Choose carefully.
11. **Update User by employee_id resets role to default** if `external_role_id` is omitted.
12. **Reports are asynchronous**; download link expires after 48 hours.
13. **Locations:** use Deactivate, not Delete (deprecated). Only company admins can set `job_loc_code`. Deactivated locations stop accepting applicants.
14. **Rooms:** `room_ex_id` must be unique within a location; `location_id` is mandatory when listing rooms.
15. **Rate limits and pagination:** the docs only say 429 = rate limit exceeded; no numeric limits are published. Pagination uses `limit`/`offset`/`page` where listed on a given endpoint.
16. **Docs inconsistencies to be aware of:** `interviewed` is described differently on Get vs Update Candidate; Get Candidates `limit` text says "should less than 50 to test"; Get Single User lists fewer response keys than Get users.

---

## 9. Coverage statement

All pages linked from the Developer Hub navigation (API Reference, Guides, Changelog, Discussions index) were read as of 2026-10-28, except:

- `https://readme.paradox.ai/llms.txt` — mentioned by the site as an index for AI agents, but could not be fetched with the available tool.
- "Report Field Documentation" — referenced by Get Report List; no link was found.
- Individual Discussions threads — only the "Rate limitation" thread was opened; it has no answer.
- Response body schemas shown only as interactive examples ("Try It" panels) were not visible in the fetched text, so JSON response examples are not included.

If a question falls outside this file, the agent must say so and point to https://readme.paradox.ai/ rather than guess.