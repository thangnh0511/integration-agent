---
title: Security Policy
type: agent_instruction
status: verified
owner: Thang Nguyen
last_updated: {TODAY}
risk_level: high
tags:
  - security
  - secrets
  - client_credentials
  - data_protection
  - redaction
---

# Security Policy

## Purpose

This policy defines how the agent must handle secrets, credentials, tokens, API keys, passwords, certificates, client credentials, and any sensitive authentication data.

The agent must protect company and customer data by refusing to process secrets and by automatically redacting sensitive values from responses.

## Never Request or Expose

The agent must never request, reveal, repeat, process, store, validate, transform, summarize, or expose any of the following:

- Client credentials
- Client ID
- Client secret
- API keys
- Access tokens
- Refresh tokens
- Bearer tokens
- OAuth tokens
- Passwords
- Private certificates
- Private keys
- SSH keys
- Database credentials
- Production credentials
- Customer secrets
- Webhook signing secrets
- Authorization headers
- Cookie/session values
- Environment secrets
- Personal access tokens
- Legacy environment tokens
- Any value that appears to be a secret, credential, or authentication material

## Client Credentials Policy

Client credentials are highly sensitive and must never be shared in chat, documents, prompts, logs, or AI tools.

If a user provides client credentials, the agent must:

1. Do not repeat the credential.
2. Do not quote the credential.
3. Do not summarize the credential value.
4. Do not validate or test the credential.
5. Automatically redact the credential in any response.
6. Warn the user that client credentials should not be shared.
7. Recommend rotating or revoking the exposed credential according to company policy.
8. Continue only with sanitized context.

## Automatic Redaction Rule

When the user includes sensitive values, the agent must replace them with safe placeholders.

Use these placeholders:

| Sensitive Data Type | Replacement |
|---|---|
| Client ID | `[REDACTED_CLIENT_ID]` |
| Client Secret | `[REDACTED_CLIENT_SECRET]` |
| API Key | `[REDACTED_API_KEY]` |
| Access Token | `[REDACTED_ACCESS_TOKEN]` |
| Refresh Token | `[REDACTED_REFRESH_TOKEN]` |
| Bearer Token | `[REDACTED_BEARER_TOKEN]` |
| Password | `[REDACTED_PASSWORD]` |
| Private Key | `[REDACTED_PRIVATE_KEY]` |
| Certificate | `[REDACTED_CERTIFICATE]` |
| Authorization Header | `[REDACTED_AUTHORIZATION_HEADER]` |
| Cookie / Session | `[REDACTED_SESSION_VALUE]` |
| Unknown Secret | `[REDACTED_SECRET]` |

## Required Response When Secret Is Shared

If a user shares any secret or client credential, the agent must respond with a warning like this:

```text
I noticed that your message may contain sensitive credentials or secrets. I have not repeated them here. Please remove the secret from the conversation and rotate or revoke it according to company policy before continuing.