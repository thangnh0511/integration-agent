---
title: Domain Index
type: domain_index
status: verified
owner: Thang Nguyen
last_updated: 2026-10-28
tags:
  - domain_index
  - integration
  - vendor_integrations
  - paradox_public_api
  - ats
  - background_check
  - sso
  - data_feed
  - mapping
---

# Domain Index

## Purpose

This file is the primary navigation index for the Franchise Integration Knowledge Base.

Use this file to identify the correct domain, subdomain, and folder before looking for specific use cases, customer implementations, Jira references, Confluence links, or assets.

## Maintenance CSV

A CSV version is available for easier maintenance:

```text
02_KNOWLEDGE_BASE/00_DOMAIN_INDEX/domain_index_maintenance.csv
```

## Security Rule

Do not store real credentials, client secrets, API keys, access tokens, refresh tokens, passwords, private keys, certificates, authorization headers, or customer secrets in this index.

If a source contains credentials, use redacted placeholders only:

- `[REDACTED_CLIENT_ID]`
- `[REDACTED_CLIENT_SECRET]`
- `[REDACTED_ACCESS_TOKEN]`
- `[REDACTED_API_KEY]`
- `[REDACTED_AUTHORIZATION_HEADER]`
- `[REDACTED_CERTIFICATE]`

## Domain Table

| Domain | Subdomain | Category | Folder | Source | Notes |
|---|---|---|---|---|---|
| Vendor Integrations | Paradox | General / Other / API / SSO / Data Feed | `02_KNOWLEDGE_BASE/01_VENDOR_INTEGRATIONS/PARADOX` | Integration tracking sheet | Includes mapping/payload updates, SSO issues, API migration, job feed, scrape setup, credential rotation, public API docs. |
| Vendor Integrations | Workday | ATS | `02_KNOWLEDGE_BASE/01_VENDOR_INTEGRATIONS/WORKDAY` | Integration tracking sheet | Includes ATS/HCM integration, new hire employee request, position report, integration design. |
| Vendor Integrations | Paylocity | ATS | `02_KNOWLEDGE_BASE/01_VENDOR_INTEGRATIONS/PAYLOCITY` | Integration tracking sheet | Includes ATS integrations, client permission blockers, API investigation, validation confirmation. |
| Vendor Integrations | UKG Pro | ATS | `02_KNOWLEDGE_BASE/01_VENDOR_INTEGRATIONS/UKG` | Integration tracking sheet | Includes UKG bi-directional integration and ATS flow. |
| Vendor Integrations | SuccessFactors | ATS / API Authentication | `02_KNOWLEDGE_BASE/01_VENDOR_INTEGRATIONS/SUCCESSFACTORS` | Integration tracking sheet | Includes OAuth 2.0 authentication and API authentication impact cases. |
| Background Check | First Advantage / FADV | Background Check / API | `02_KNOWLEDGE_BASE/06_BACKGROUND_CHECK/FADV` | Integration tracking sheet | Includes background check implementations, SIT cases, timeout/API troubleshooting. |
| Background Check | Accurate | Background Check | `02_KNOWLEDGE_BASE/06_BACKGROUND_CHECK/ACCURATE` | Integration tracking sheet | Includes vendor credential dependency, verification, and production move. |
| Background Check | Sterling | Background Check | `02_KNOWLEDGE_BASE/06_BACKGROUND_CHECK/STERLING` | Integration tracking sheet | Includes package ID setup, regional package logic, staging validation, production readiness. |
| Background Check | Avvanz | Background Check | `02_KNOWLEDGE_BASE/06_BACKGROUND_CHECK/AVVANZ` | User requested domain | Prepared as a maintainable domain even if no current sheet rows were visible in exported worksheet. |
| Assessments | Corvirtus | Assessment | `02_KNOWLEDGE_BASE/07_ASSESSMENTS/CORVIRTUS` | Integration tracking sheet | Includes assessment update cases and production readiness. |
| Paradox Public API | Authentication | OAuth / Credentials / Token Flow | `02_KNOWLEDGE_BASE/02_PARADOX_PUBLIC_API/authentication` | Integration tracking sheet + readme.paradox.ai placeholder | Store only sanitized OAuth/client credential guidance. Do not store secrets. |
| Paradox Public API | Endpoints | Candidate Status / Job Feed / Webhooks | `02_KNOWLEDGE_BASE/02_PARADOX_PUBLIC_API/endpoints` | Integration tracking sheet + readme.paradox.ai placeholder | Includes candidate status API, job feed retrieve postings, webhook research, proxy domain update. |
| SSO and IDP | SSO Provider / Metadata / Certificate | SSO | `02_KNOWLEDGE_BASE/03_SSO_AND_IDP` | Integration tracking sheet | Includes SSO login error, provider change, IDP metadata, certificate update. |
| Data Feed and Job Feed | SFTP / Scrape / Proxy Domain / Feed File | Data Feed | `02_KNOWLEDGE_BASE/04_DATA_FEED_AND_JOB_FEED` | Integration tracking sheet | Includes SFTP job search, scrape production setup, job feed endpoint update. |
| Mapping and Payload | Mapped Values / Payload Attributes / Journey Mapping | Mapping | `02_KNOWLEDGE_BASE/08_MAPPING_AND_PAYLOAD` | Integration tracking sheet | Includes new mapped values, payload attributes, Professional/Frontline mapping, syntax fix. |

## Suggested Agent Behavior

When answering user questions, the agent should:

1. Identify the relevant domain.
2. Identify the relevant subdomain or vendor.
3. Check the corresponding use case files.
4. Check customer implementation examples if available.
5. Check Jira references and Confluence link indexes if relevant.
6. Check assets only when they are marked safe for agent usage.
7. Avoid deprecated or archived content unless the user explicitly asks for historical context.
8. Redact any credentials or secrets found in user input or source content.

## Notes From Tracking Sheet

- The tracking sheet includes ATS, Background Check, Assessment, SSO, API, Data Feed, Mapping, credential rotation, certificate update, and production readiness cases.
- Some requests are marked `Ready To PROD`, `SIT`, `Waiting CS / Client`, `Done / Resolved`, `In-progress`, or `Obsolete`.
- The agent should use status to decide whether to recommend testing, follow-up, production readiness checks, documentation, or escalation.
- Cases involving OAuth, client credentials, client secrets, access tokens, certificates, or API authentication must follow the security policy.
