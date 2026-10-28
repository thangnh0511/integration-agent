---
title: Rule - Timeout Handling
type: rule
domain: API
status: verified
owner: Thang Nguyen
last_updated: 2026-10-28
risk_level: medium
tags:
  - timeout
  - api
  - rule
---

# Rule: Timeout Handling

## Applies To

API integrations where a third-party API takes longer than the current timeout setting.

## Required Context

Ask for:

- API name
- Endpoint
- Current timeout value
- Average response time
- Maximum observed response time
- Environment
- Error message
- Retry behavior
- Production impact

## Standard Rule

If the API consistently responds successfully but takes longer than the current timeout, consider increasing the timeout to a value above the maximum observed response time, if allowed by the integration policy.

## Example

If the API takes around 55 seconds to respond and the current timeout is lower, a 60-second timeout may be considered, but this must be validated in a non-production environment first.

## Validation Checklist

- Confirm response succeeds within new timeout.
- Confirm no duplicate requests are created.
- Confirm retry behavior does not conflict with timeout.
- Confirm user experience impact.
- Confirm logs show expected behavior.
- Confirm production change approval if applicable.

## Risks

- Longer timeouts may delay error handling.
- Retry behavior may create duplicate calls.
- Production changes may impact user experience.
- Upstream API may still exceed the new timeout.

## Escalation

Escalate if:

- The timeout change affects production.
- The API response is inconsistent.
- Retry behavior is unclear.
- There is risk of duplicate transactions.
