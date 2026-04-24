---
name: security-and-hardening
description: Harden web application code against vulnerabilities during development. Use while writing any feature that accepts untrusted data, handles authentication or sessions, stores or transmits sensitive information, integrates with third-party APIs, accepts file uploads, or exposes webhooks and callbacks. Covers OWASP Top 10 prevention patterns, input validation at system boundaries, parameterized queries, output encoding, secrets management, rate limiting, session hardening, and the three-tier "always / ask first / never" boundary system. Do not use for post-implementation security audits, threat modeling of finished systems, or vulnerability reports — use the `barb` / `security-auditor` agent for that. This skill is for building secure code; Barb is for auditing built code.
---

# Security and Hardening

Treat every external input as hostile, every secret as sacred, and every authorization check as mandatory. Security is a constraint on every line of code that touches user data, authentication, or external systems — not a phase.

## Split with Barb (security-auditor)

This skill owns **hardening during development** — the patterns, guardrails, and checklists applied while writing code. The `barb` agent (`~/.claude/agents/security-auditor.md`) owns **auditing after implementation** — reviewing built code, classifying severity, producing an audit report. Reach for this skill when building; reach for Barb when reviewing finished code for exploitable issues.

## The Three-Tier Boundary System

### Always Do (No Exceptions)

- Validate all external input at the system boundary (API routes, form handlers).
- Parameterize all database queries — never concatenate user input into SQL.
- Encode output to prevent XSS (use framework auto-escaping; do not bypass it).
- Use HTTPS for all external communication.
- Hash passwords with bcrypt, scrypt, or argon2 (never store plaintext).
- Set security headers (CSP, HSTS, X-Frame-Options, X-Content-Type-Options).
- Use httpOnly, secure, sameSite cookies for sessions.
- Run `npm audit` (or equivalent) before every release.

### Ask First (Requires Human Approval)

- Adding new authentication flows or changing auth logic.
- Storing new categories of sensitive data (PII, payment info).
- Adding new external service integrations.
- Changing CORS configuration.
- Adding file upload handlers.
- Modifying rate limiting or throttling.
- Granting elevated permissions or roles.

### Never Do

- Never commit secrets to version control (API keys, passwords, tokens).
- Never log sensitive data (passwords, tokens, full credit card numbers).
- Never trust client-side validation as a security boundary.
- Never disable security headers for convenience.
- Never use `eval()` or `innerHTML` with user-provided data.
- Never store sessions in client-accessible storage (no auth tokens in localStorage).
- Never expose stack traces or internal error details to users.

## OWASP Top 10 Prevention

For per-category BAD/GOOD code patterns (injection, broken auth, XSS, access control, misconfiguration, sensitive data exposure) plus schema validation and file upload safety, see `references/owasp-patterns.md`.

## Operational Security

For rate limiting, secrets management (`.env` hygiene, pre-commit secret scan), and a decision tree for triaging `npm audit` findings, see `references/operational-security.md`.

## Pre-Ship Security Checklist

For the full auth / authorization / input / data / infrastructure checklist to run before shipping any security-relevant change, see `references/security-checklist.md`.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "This is an internal tool, security doesn't matter" | Internal tools get compromised. Attackers target the weakest link. |
| "We'll add security later" | Security retrofitting is 10x harder than building it in. Add it now. |
| "No one would try to exploit this" | Automated scanners will find it. Security by obscurity is not security. |
| "The framework handles security" | Frameworks provide tools, not guarantees. You still need to use them correctly. |
| "It's just a prototype" | Prototypes become production. Security habits from day one. |

## Red Flags

- User input passed directly to database queries, shell commands, or HTML rendering
- Secrets in source code or commit history
- API endpoints without authentication or authorization checks
- Missing CORS configuration or wildcard (`*`) origins
- No rate limiting on authentication endpoints
- Stack traces or internal errors exposed to users
- Dependencies with known critical vulnerabilities

## Verification

After implementing security-relevant code:

- [ ] `npm audit` shows no critical or high vulnerabilities
- [ ] No secrets in source code or git history
- [ ] All user input validated at system boundaries
- [ ] Authentication and authorization checked on every protected endpoint
- [ ] Security headers present in response (check with browser DevTools)
- [ ] Error responses don't expose internal details
- [ ] Rate limiting active on auth endpoints

For a post-implementation audit with severity classification and an exploit-focused report, hand off to the `barb` agent (`~/.claude/agents/security-auditor.md`).
