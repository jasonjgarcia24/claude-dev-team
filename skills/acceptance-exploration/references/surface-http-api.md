# Surface: HTTP API

Probing tool: Bash (curl or httpie) or a minimal client script.

## How to probe

Issue requests against each documented endpoint with representative payloads. Capture status, body, headers, and latency.

Pattern:

```bash
curl -sS -X POST "$BASE/endpoint" \
  -H 'content-type: application/json' \
  -d '{"field":"value"}' \
  -o /tmp/body.json \
  -w 'status=%{http_code} time=%{time_total}s\n' \
  -D /tmp/headers.txt
```

For auth: set `Authorization` header; test unauthenticated and wrong-user variants for 401/403 verification.

## Evidence to capture

- Full request/response pair per endpoint (headers + body) — save to evidence dir
- Status code + latency per request (from `-w` format string)
- Any retry/backoff traces

## Stage-specific additions

### MVP
- Every documented endpoint returns something — no 404, no connection refused, no 502
- Valid inputs → 2xx; invalid inputs → 4xx with a usable error body (not a stack trace)

### Beta
- Auth boundaries hold: unauthenticated → 401, wrong-user → 403 (not 500)
- Idempotency on documented-idempotent endpoints: retry the same payload, outcome unchanged
- Content-type mismatches and malformed JSON → 400, not 500
- Duplicate-submit / replay: behavior matches contract

### GA
- Status codes match semantic: 2xx success, 4xx client error with actionable body, 5xx server error — no lies (e.g., 200 on logical failure)
- Response schemas match the documented contract (field names, types, nullability, required vs. optional)
- Error responses include actionable messages, not stack traces or internal details
- Latency within stated budget (e.g., p95 targets if documented)
- Rate-limit behavior matches documentation (429 with `Retry-After` when applicable)
- CORS / security headers configured where expected
