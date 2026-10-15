# api-gateway

NGINX routing integration and gateway-facing API concerns. Keep business logic out of the proxy configuration. Add request IDs, bounded rate limits, and health/readiness endpoints.

## Implementation checklist

- [ ] Define input/output contract.
- [ ] Add configuration validation and `/health` endpoint (for HTTP service).
- [ ] Add structured logging and correlation IDs.
- [ ] Add unit and contract tests.
- [ ] Add Dockerfile and document dependencies.
- [ ] Add timeout, retry, and failure behaviour where relevant.
