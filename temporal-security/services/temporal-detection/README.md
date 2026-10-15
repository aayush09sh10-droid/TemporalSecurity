# temporal-detection

Python/FastAPI service for rule-based and statistical baselines first, then temporal ML inference. Return evidence and model version; do not execute mitigations here.

## Implementation checklist

- [ ] Define input/output contract.
- [ ] Add configuration validation and `/health` endpoint (for HTTP service).
- [ ] Add structured logging and correlation IDs.
- [ ] Add unit and contract tests.
- [ ] Add Dockerfile and document dependencies.
- [ ] Add timeout, retry, and failure behaviour where relevant.
