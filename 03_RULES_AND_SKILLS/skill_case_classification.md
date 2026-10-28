---
title: Skill - Case Classification
type: skill
status: verified
owner: Thang Nguyen
last_updated: 2026-10-28
tags:
  - skill
  - classification
---

# Skill: Case Classification

## Purpose

Classify the user's request before recommending a solution.

## Classification Dimensions

### Integration Type

- API integration
- File-based integration
- Offer file integration
- Franchise integration
- Background check integration
- Mapping configuration
- Unknown

### Issue Type

- Implementation request
- Mapping issue
- Timeout issue
- Authentication issue
- Payload issue
- File format issue
- Environment issue
- Production impact
- Unknown

### Risk Level

- Low: informational or non-production guidance
- Medium: configuration change or customer-specific behavior
- High: production impact, security, data loss, duplicate data, candidate status impact

## Required Behavior

Before giving a specific solution:

1. Identify the likely integration type.
2. Identify the issue type.
3. Identify missing information.
4. Identify risk level.
5. Ask clarifying questions if needed.

## Output

Include classification briefly in the Summary section when helpful.
