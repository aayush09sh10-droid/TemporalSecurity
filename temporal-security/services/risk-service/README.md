# risk-service

Convert detection evidence into a calibrated severity/risk result. Keep score semantics documented; do not present uncalibrated scores as probabilities.

## Implementation checklist

- [ ] Define input/output contract.
- [ ] Add configuration validation and `/health` endpoint (for HTTP service).
- [ ] Add structured logging and correlation IDs.
- [ ] Add unit and contract tests.
- [ ] Add Dockerfile and document dependencies.
- [ ] Add timeout, retry, and failure behaviour where relevant.
