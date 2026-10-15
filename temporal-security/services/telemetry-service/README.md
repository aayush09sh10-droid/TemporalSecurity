# telemetry-service

Validate incoming telemetry against versioned contracts, attach correlation IDs, and publish accepted events to Kafka. Track rejected and dropped events.

## Implementation checklist

- [ ] Define input/output contract.
- [ ] Add configuration validation and `/health` endpoint (for HTTP service).
- [ ] Add structured logging and correlation IDs.
- [ ] Add unit and contract tests.
- [ ] Add Dockerfile and document dependencies.
- [ ] Add timeout, retry, and failure behaviour where relevant.
