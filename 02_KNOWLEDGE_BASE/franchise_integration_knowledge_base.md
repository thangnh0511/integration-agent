---
title: Franchise Integration Knowledge Base
type: knowledge_guide
status: verified
owner: Thang Nguyen
last_updated: 2026-10-28
risk_level: medium
tags:
  - franchise
  - integration
  - tracking_logs
  - ats
  - background_check
  - assessment
  - sso
  - api
  - data_feed
  - job_feed
  - credential_rotation
  - production_readiness
---

# Franchise Integration Knowledge Base

## Overview

This knowledge base summarizes recurring integration patterns from the integration tracking log. It is intended to help the Sana Agent classify requests, recommend next steps, identify risks, and generate implementation or validation checklists.

The tracking log includes integration and issue requests across partners such as Paradox, Workday, Paylocity, UKG Pro, First Advantage / FADV, Accurate, Sterling, Corvirtus, and other customer-specific systems.

## Core Request Categories

### ATS Integrations

ATS-related cases may involve Workday, Paylocity, UKG Pro, SuccessFactors, or customer-specific ATS flows.

Common work includes:

- Clarifying API requirements
- Running integration requests
- Preparing new hire or employee request flows
- Waiting for client permissions
- Waiting for vendor API investigation
- Validating implementation before client confirmation
- Preparing go-live or production move

Required context:

- Customer
- ATS partner
- Environment
- API or file flow
- Current issue
- Required permission
- Expected behavior
- Production target date

### Background Check Integrations

Background check cases may involve First Advantage / FADV, Accurate, Sterling, or similar partners.

Common work includes:

- Waiting for vendor credentials or required setup information
- Configuring package IDs or package mapping
- Running integration successfully in test
- Waiting for client or vendor verification
- Moving to production after validation
- Handling regional package logic such as US, AU, NZ, or other region-specific packages

Required context:

- Vendor
- Customer
- Package ID or package mapping
- Region
- Environment
- Credential availability
- Test result
- Vendor confirmation
- Production readiness

### Assessment Integrations

Assessment-related cases may involve vendors such as Corvirtus.

Common work includes:

- Updating assessment integration setup
- Testing candidate assessment flow
- Confirming vendor behavior
- Preparing production move

Required context:

- Vendor
- Customer
- Assessment flow
- Environment
- Current configuration
- Expected behavior
- Validation evidence

### SSO and IDP Integrations

SSO cases may include login errors, SSO provider changes, metadata exchange, IDP metadata, certificate updates, or identity provider migrations.

Common work includes:

- Investigating SSO login errors
- Sending metadata to the client
- Waiting for client setup
- Receiving IDP metadata file
- Updating IDP certificate
- Validating in staging
- Moving to production after client confirmation

Required context:

- Customer
- SSO provider
- Environment
- Metadata status
- Certificate status
- Error message
- Affected users
- Client confirmation
- Production target

### API Migration and Endpoint Updates

API migration cases may involve changing endpoints, proxy domains, or API authentication methods.

Common work includes:

- Replacing old API endpoints with new proxy domains
- Reconfiguring job feed endpoints
- Validating response from new endpoint
- Testing staging before production
- Updating authentication method such as OAuth 2.0
- Confirming downstream job feed or candidate status behavior

Required context:

- Old endpoint
- New endpoint
- Environment
- Authentication method
- Current response
- Expected response
- Feed or API reference
- Validation result
- Production impact

### Data Feed and Job Feed Integrations

Data feed or job feed cases may involve file feeds, scrape setup, job posting retrieval, custom reports, or customer-hosted APIs.

Common work includes:

- Waiting for client to update job feed files
- Configuring scrape or feed setup
- Updating API proxy domains
- Retrieving job postings
- Validating feed details
- Confirming data freshness and required fields

Required context:

- Feed type
- Endpoint or file location
- Environment
- Required fields
- Authentication method
- Feed validation result
- Owner or client contact
- Production readiness

### Mapping Updates

Mapping cases may involve payload attributes, mapped values, candidate status, journey-specific behavior, or downstream status/routing.

Common work includes:

- Adding new mapped values
- Adding payload attributes
- Updating mapping for Professional journey
- Updating mapping for Frontline journey
- Validating affected journey
- Confirming downstream behavior

Required context:

- Source system
- Target system
- Source field
- Target field
- Current mapping
- Expected mapping
- Journey
- Environment
- Production impact

### Credential Rotation and Certificate Updates

Credential and certificate cases may involve API client secret rotation, certificate update, OAuth migration, or scheduled credential reset.

Important security rule:

- Never store real client credentials, client secrets, API keys, access tokens, refresh tokens, authorization headers, passwords, private keys, or certificates in this knowledge base.
- Use redacted placeholders only.
- If credentials were shared in the source notes, treat them as exposed and recommend rotating according to company policy.

Common work includes:

- Credential rotation request
- Generating new client secret through approved process
- Updating client credentials in secure configuration
- Updating certificates
- Validating authentication after update
- Preparing production change

Required context:

- Customer
- API client or integration name
- Environment
- Rotation reason
- Current expiration or rotation date
- Secure owner
- Validation plan
- Production impact

## Status-Based Workflow

### Done / Resolved

Use this status when the issue has been implemented, verified, or closed.

Recommended documentation:

- Root cause
- Solution
- Validation result
- Owner
- Related Jira
- Lessons learned

### Waiting CS / Client

Use this status when the next action depends on Customer Success or the client.

Recommended next steps:

- Summarize what is pending
- Draft follow-up questions
- Identify missing permissions or confirmations
- Track owner/contact point

### Ready To PROD

Use this status when the integration has passed test or staging validation and is waiting for production deployment.

Production readiness checklist:

- Client confirmed expected behavior
- Vendor confirmed expected behavior if applicable
- SIT/UAT evidence is available
- Rollback plan exists
- Monitoring plan exists
- Production deployment owner is confirmed
- Production date is confirmed

### SIT

Use this status when testing is in progress.

SIT checklist:

- Test cases are defined
- Test data is available
- Expected result is documented
- Actual result is recorded
- Issues are tracked
- Blockers are escalated

### In-progress

Use this status when implementation, investigation, or requirement gathering is still active.

Recommended next steps:

- Identify current blocker
- Confirm missing information
- Confirm owner
- Confirm expected timeline
- Continue implementation or troubleshooting

### Obsolete

Use this status when the request is no longer active or has been replaced.

Recommended behavior:

- Do not use obsolete case as implementation guidance
- Use only for historical context
- Prefer current verified guides or cases

## Priority Handling

### Medium

Normal priority. Follow standard implementation, validation, and escalation processes.

### ASAP / Urgent

High urgency. The agent should:

- Identify production impact
- Clarify deadline
- Identify blocker
- Recommend escalation if owner or dependency is unclear
- Avoid skipping validation
- Create concise action plan

### Free Fire / Critical

Treat as critical or very high urgency.

The agent should:

- Confirm production impact immediately
- Identify impacted workflow
- Recommend escalation to integration lead or responsible owner
- Preserve logs and current state
- Avoid unvalidated production changes
- Create immediate action checklist

## Common Partner Dependency Patterns

### Vendor Credentials or Required Information Pending

If vendor credentials or required information are pending:

- Do not proceed with real credential handling in chat
- Ask for status of secure credential delivery
- Confirm credential owner
- Confirm required setup details
- Recommend secure credential storage and approved process

### Client Permission Pending

If client permission is pending:

- Identify required permission
- Draft client follow-up
- Confirm who owns client communication
- Do not mark ready for production until permission is granted

### Vendor Investigation Pending

If vendor investigation is pending:

- Summarize error and evidence
- Collect logs or response examples without secrets
- Identify vendor contact
- Track expected response date
- Prepare workaround only if documented and approved

## Production Readiness Checklist

Before moving any integration to production, confirm:

- Non-production validation completed
- Client or internal owner confirmed behavior
- Vendor confirmed if vendor dependency exists
- Required permissions are granted
- No secrets are exposed
- Rollback plan exists
- Monitoring plan exists
- Production owner is confirmed
- Production date is confirmed
- Related Jira or tracking reference is updated

## Security and Redaction Guidance

The source tracking log may contain notes related to authentication, OAuth, access tokens, client credentials, and API endpoints.

The agent must not repeat actual credential values.

Use placeholders:

| Data Type | Placeholder |
|---|---|
| Client ID | [REDACTED_CLIENT_ID] |
| Client Secret | [REDACTED_CLIENT_SECRET] |
| Access Token | [REDACTED_ACCESS_TOKEN] |
| Refresh Token | [REDACTED_REFRESH_TOKEN] |
| Authorization Header | [REDACTED_AUTHORIZATION_HEADER] |
| API Key | [REDACTED_API_KEY] |
| Password | [REDACTED_PASSWORD] |
| Certificate | [REDACTED_CERTIFICATE] |
| Private Key | [REDACTED_PRIVATE_KEY] |

If a user provides credentials, warn them and recommend credential rotation according to company policy.

## Suggested Agent Response Patterns

### For a New Integration Request

Return:

1. Request classification
2. Missing information
3. Recommended implementation path
4. Required partner/client dependencies
5. Validation checklist
6. Risks
7. Escalation criteria

### For a Waiting Client Case

Return:

1. Current blocker
2. What is needed from client or CS
3. Suggested follow-up message
4. Impact of delay
5. Next step after confirmation

### For a Ready To PROD Case

Return:

1. Production readiness summary
2. Validation evidence needed
3. Approval checklist
4. Rollback and monitoring plan
5. Go-live risk
6. Escalation if anything is missing

### For a Credential Rotation Case

Return:

1. Security warning
2. Do not request or repeat credentials
3. Required secure rotation process
4. Validation checklist
5. Production impact checklist
6. Escalation to security/platform owner if needed

### For an API Timeout Case

Return:

1. Timeout diagnosis
2. Required context
3. Safe recommended timeout change if documented
4. Retry and duplicate risk
5. Test validation
6. Production approval requirement

## Known Recurring Case Examples

### FADV API Timeout

Pattern:

- API response can take longer than expected.
- If response succeeds but exceeds current timeout, consider increasing timeout above observed maximum response time.
- Validate in test first.
- Confirm retry behavior does not create duplicate calls.

### SMSC Mapping for Professional and Frontline Journeys

Pattern:

- Mapping must be validated for both Professional and Frontline journeys.
- Validate downstream behavior separately for each journey.
- Document old mapping and new mapping.
- Confirm no unrelated journey is impacted.

### SSO Provider or Certificate Update

Pattern:

- Send metadata to client.
- Wait for IDP metadata or certificate confirmation.
- Update in staging.
- Ask client to test.
- Move to production only after confirmation.

### Job Feed Endpoint or Proxy Domain Update

Pattern:

- Confirm old endpoint and new endpoint.
- Reconfigure endpoint.
- Validate feed response.
- Confirm downstream scrape or job posting behavior.
- Notify requester after verification.

### Background Check Package Setup

Pattern:

- Confirm vendor package IDs.
- Configure mapping logic.
- Validate test candidates across regions if applicable.
- Wait for vendor/client verification.
- Move to production only after approval.

## Documentation Requirement

After resolving a case, create or update a case study with:

- Case title
- Partner/system
- Function type
- Status
- Priority
- Problem
- Root cause
- Solution
- Validation result
- Contact point if safe to share
- Related Jira if available
- Last updated date
