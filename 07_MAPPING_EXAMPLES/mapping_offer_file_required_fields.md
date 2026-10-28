---
title: Mapping Offer File Required Fields
type: mapping
domain: Offer File
status: verified
owner: Thang Nguyen
last_updated: 2026-10-28
tags:
  - offer_file
  - mapping
  - required_fields
---

# Mapping: Offer File Required Fields

## Purpose

This document lists common required fields for offer file integrations.

## Common Required Fields

| Source Field | Target Field | Required | Notes |
|---|---|---|---|
| candidate_id | Candidate ID | Yes | Must uniquely identify candidate |
| requisition_id | Requisition ID | Yes | Must match the target system requisition |
| offer_status | Offer Status | Yes | Must use approved status values |
| offer_date | Offer Date | Usually | Format must be confirmed |
| start_date | Start Date | Usually | Required if downstream process depends on start date |

## Validation

- Confirm all required fields exist.
- Confirm field names match expected schema.
- Confirm date format.
- Confirm status values.
- Test with sample file.

## Risks

- Missing candidate ID may prevent matching.
- Incorrect requisition ID may route data incorrectly.
- Invalid offer status may block downstream processing.
