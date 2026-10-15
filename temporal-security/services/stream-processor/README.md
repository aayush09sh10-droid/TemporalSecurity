# stream-processor

Consume telemetry, handle duplicates and late events, aggregate event-time windows, and publish aggregate outputs. Make processing idempotent.

## Implementation checklist

- [ ] Define input/output contract.
- [ ] Add configuration validation and `/health` endpoint (for HTTP service).
- [ ] Add structured logging and correlation IDs.
- [ ] Add unit and contract tests.
- [ ] Add Dockerfile and document dependencies.
- [ ] Add timeout, retry, and failure behaviour where relevant.
