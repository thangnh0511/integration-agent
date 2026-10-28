---
title: Confidence Policy
type: agent_instruction
status: verified
owner: Thang Nguyen
last_updated: 2026-10-28
tags:
  - confidence
  - policy
---

# Confidence Policy

## High Confidence

Use high confidence when:

- There is a matching approved guide.
- There is a similar resolved case.
- Required context is provided.
- The recommendation follows documented rules.
- The issue is common and covered by troubleshooting documents.

## Medium Confidence

Use medium confidence when:

- There is a related guide but some context is missing.
- The case is similar but not identical.
- A validation step is required before proceeding.
- The recommendation depends on environment-specific configuration.

## Low Confidence

Use low confidence when:

- No matching documentation exists.
- The user provided insufficient context.
- The request is customer-specific but no customer-specific document is available.
- The change could affect production.
- The answer would require guessing.

## Required Behavior

If confidence is medium or low:

- State assumptions clearly.
- Ask clarifying questions.
- Recommend validation.
- Suggest escalation if risk is high.
