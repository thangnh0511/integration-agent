---
title: SMSC Mapping for Professional and Frontline Journey
type: case_study
domain: SMSC
integration_type: Mapping
status: verified
owner: Thang Nguyen
last_updated: 2026-10-28
risk_level: medium
tags:
  - smsc
  - mapping
  - professional
  - frontline
---

# Case: SMSC Mapping for Professional and Frontline Journey

## Summary

Mapping values were updated on SMSC for Professional and Frontline journeys.

## Context

- System: SMSC
- Issue type: Mapping update
- Affected journeys:
  - Professional
  - Frontline

## Problem

The mapping value needed to be updated for both Professional and Frontline journeys.

## Solution

Update the mapping value on SMSC for the affected journeys and validate both flows.

## Implementation Steps

1. Confirm the current mapping value.
2. Confirm the expected mapping value.
3. Identify affected journeys.
4. Update mapping value on SMSC.
5. Test Professional journey.
6. Test Frontline journey.
7. Confirm downstream behavior.
8. Document the change.

## Validation Checklist

- Professional journey uses the expected mapping.
- Frontline journey uses the expected mapping.
- No unrelated journey is impacted.
- Downstream data is correct.
- Change is documented.

## Risks

- Updating only one journey may create inconsistent behavior.
- Incorrect mapping may affect downstream status or routing.

## Related Rules

- Rule: Mapping Changes
- Workflow: Mapping Change Request
