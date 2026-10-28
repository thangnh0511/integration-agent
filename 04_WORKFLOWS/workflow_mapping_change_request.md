---
title: Workflow - Mapping Change Request
type: workflow
status: verified
owner: Thang Nguyen
last_updated: 2026-10-28
risk_level: medium
tags:
  - mapping
  - workflow
---

# Workflow: Mapping Change Request

## Purpose

Handle requests to update or validate mapping values.

## Step 1: Collect Required Information

Ask for:

- Customer
- Source system
- Target system
- Source field
- Target field
- Current mapping
- Expected mapping
- Journey or workflow
- Environment
- Production impact

## Step 2: Review Mapping Guide

Check approved mapping examples and customer-specific notes.

## Step 3: Assess Impact

Check:

- Which journey is affected?
- Is this Professional, Frontline, or both?
- Does the mapping affect candidate status?
- Does this affect downstream reporting?

## Step 4: Implement in Non-Production

Apply mapping change in test or staging first.

## Step 5: Validate

Test sample records for each affected journey.

## Step 6: Production Change

Proceed only after approval and validation.

## Step 7: Document

Record:

- Old mapping
- New mapping
- Reason
- Validation result
- Date
- Owner
