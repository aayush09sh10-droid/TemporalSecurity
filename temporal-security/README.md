# Temporal Security Platform

A research-oriented microservices platform for detecting dangerous behavioural transitions that may precede application resource exhaustion. The system correlates telemetry over time rather than relying only on individual malicious-request signatures.

> **Project status:** Architecture scaffold / implementation starting point. The directories are organized for development; the services are not implemented yet.

## 1. Goals

- Collect application and infrastructure telemetry.
- Correlate signals across configurable time windows.
- Detect transitions such as `NORMAL -> ANOMALOUS -> DEGRADING -> CRITICAL`.
- Produce explainable risk assessments and alerts.
- Support controlled, auditable mitigation.
- Evaluate whether temporal, multi-signal detection provides earlier warning than request-level and threshold-based baselines.

This is a research prototype, not a replacement for a commercial WAF, DDoS protection platform, or production incident-response system.

## 2. Architecture at a glance

```text
Clients / Load Simulator
          |
          v
      NGINX Gateway -------> Demo Application
          |                       |
          +------ telemetry ------+
                      |
                      v
             Telemetry Service
                      |
                    Kafka
                      |
                      v
             Stream Processor
                      |
                      v
             Feature Engineering
                      |
                      v
          Temporal Detection (Python)
                      |
                      v
                Risk Scoring
                      |
                      v
              Policy / Mitigation
                 |          |
                 v          v
          Incident/Audit   Gateway Adapter
                 |
                 v
           React Dashboard
```

### Communication rules

- **HTTP/REST:** synchronous queries, configuration and control commands.
- **Kafka:** telemetry, features, detection results, alerts and other asynchronous events.
- **Redis:** short-lived state, rolling-window aggregates, deduplication keys and caches.
- **PostgreSQL:** durable incidents, policies, audit records and application metadata.
- **Object storage (optional later):** historical event archives and model artifacts.

Services must not read or write another service's private database tables directly. Share versioned contracts, not internal implementation details.

## 3. Technology stack

| Area | Technology |
|---|---|
| Gateway | NGINX |
| Core APIs and services | Node.js + TypeScript |
| ML / temporal inference API | Python + FastAPI |
| Event streaming | Apache Kafka |
| Fast temporary state | Redis |
| Durable relational data | PostgreSQL |
| Dashboard | React + TypeScript |
| Local development | Docker Compose |
| Metrics and dashboards | Prometheus + Grafana |
| Tracing | OpenTelemetry |
| Centralized logs (later) | Loki |
| Testing | Unit, contract, integration, end-to-end and resilience tests |

Start with Docker Compose. Introduce Kubernetes only when deployment and scaling requirements justify it.

## 4. Repository structure

```text
temporal-security/
├── apps/
│   └── dashboard/                 # React UI
├── services/
│   ├── api-gateway/               # Gateway-facing API and routing integration
│   ├── identity-service/          # Users, roles, sessions; add when needed
│   ├── telemetry-service/         # Collect, validate and publish telemetry
│   ├── stream-processor/           # Kafka consumers and time-window aggregation
│   ├── feature-service/            # Build versioned temporal feature vectors
│   ├── temporal-detection/         # Python rules, state machine and ML inference
│   ├── risk-service/               # Risk scoring and explanations
│   ├── policy-service/             # Policy decisions and approved mitigation
│   ├── alert-service/              # Alert deduplication and notifications
│   └── incident-service/           # Durable incidents and audit history
├── packages/
│   ├── event-contracts/             # Versioned event schemas
│   ├── api-contracts/               # Shared API schemas/types
│   ├── config/                      # Shared configuration conventions
│   ├── logger/                      # Structured logging helpers
│   ├── observability/               # Metrics/tracing conventions
│   └── testing-utils/               # Shared test fixtures
├── ml/
│   ├── datasets/                    # Dataset manifests and documentation; avoid committing secrets/data
│   ├── preprocessing/
│   ├── feature-engineering/
│   ├── baselines/
│   ├── experiments/
│   ├── training/
│   ├── evaluation/
│   ├── model-registry/
│   └── notebooks/
├── infrastructure/
│   ├── nginx/
│   ├── kafka/
│   ├── redis/
│   ├── postgres/
│   ├── prometheus/
│   ├── grafana/
│   ├── loki/
│   └── otel-collector/
├── simulator/
│   ├── scenarios/                   # Reproducible normal/degradation scenarios
│   ├── traffic-generator/
│   ├── resource-exhaustion-lab/      # Authorized isolated test environment only
│   └── evaluation/
├── tests/
│   ├── unit/
│   ├── contract/
│   ├── integration/
│   ├── end-to-end/
│   ├── resilience/
│   └── performance/
├── docs/
│   ├── architecture/
│   ├── api/
│   ├── event-schemas/
│   ├── threat-model/
│   └── research-methodology/
├── .env.example
├── .gitignore
├── package.json
├── pnpm-workspace.yaml
├── docker-compose.yml
└── README.md
```

## 5. Service responsibilities

| Service | Owns | Main interfaces |
|---|---|---|
| `api-gateway` | Routing, request IDs, edge limits | HTTP |
| `identity-service` | Users, roles, sessions | REST; optional service identity |
| `telemetry-service` | Event validation and ingestion | HTTP/OTLP input; Kafka producer |
| `stream-processor` | Event-time windows and aggregation | Kafka consumer/producer; Redis |
| `feature-service` | Versioned feature vectors | Kafka consumer/producer |
| `temporal-detection` | Transition detection and model inference | Kafka or HTTP inference API |
| `risk-service` | Risk score, severity, evidence summary | Kafka consumer/producer or REST |
| `policy-service` | Policy evaluation and action authorization | REST/control commands; Kafka events |
| `alert-service` | Alert lifecycle and notification delivery | Kafka consumer; notification providers |
| `incident-service` | Incident history and audit records | REST; Kafka consumer |

Keep the first implementation small. `identity-service`, `alert-service`, model registry, threat intelligence, dependency graph and advanced sampling can be added when there is a concrete requirement. Do not split every feature into a separate deployable service by default.

## 6. Phased implementation plan

### Phase 1 — Foundation
- [ ] Confirm repository and service boundaries.
- [ ] Add Node/TypeScript workspace and Python service conventions.
- [ ] Configure Docker Compose networking and environment variables.
- [ ] Start NGINX, Kafka, Redis and PostgreSQL.
- [ ] Define health endpoints, structured logs and correlation IDs.
- [ ] Write the first version of event contracts.

**Exit criteria:** infrastructure starts reliably; containers can reach each other by service name; secrets are not committed.

### Phase 2 — Gateway and telemetry
- [ ] Create a small demo application behind NGINX.
- [ ] Add request IDs and safe access logging.
- [ ] Implement telemetry ingestion, validation and Kafka publishing.
- [ ] Add invalid-event and unavailable-broker tests.

**Exit criteria:** a request to the demo app produces a validated telemetry event in Kafka.

### Phase 3 — Stream processing and features
- [ ] Consume telemetry events.
- [ ] Implement configurable event-time windows and late-event handling.
- [ ] Calculate request rate, latency percentiles, error rate and resource trends.
- [ ] Store only required short-lived state in Redis.
- [ ] Add replayable test fixtures.

**Exit criteria:** known input events produce expected feature values across multiple windows.

### Phase 4 — Temporal detection
- [ ] Implement threshold and statistical baselines first.
- [ ] Define state transitions and transition evidence.
- [ ] Add the Python/FastAPI detection service.
- [ ] Publish versioned detection results.
- [ ] Later compare suitable time-series or sequence models against baselines.

**Exit criteria:** controlled test scenarios produce explainable, reproducible detection results.

### Phase 5 — Risk and response
- [ ] Add risk scoring and severity calibration.
- [ ] Add policy evaluation and alert-only mode.
- [ ] Add an explicit, auditable mitigation adapter.
- [ ] Require safe limits, idempotency, authorization and rollback for actions.

**Exit criteria:** a detection can create an incident/alert; mitigation is only applied when an explicit policy allows it.

### Phase 6 — Incidents and dashboard
- [ ] Persist incident lifecycle and audit history in PostgreSQL.
- [ ] Build React views for telemetry, transitions, alerts and mitigation status.
- [ ] Add live updates only after the underlying APIs and event flow are stable.

**Exit criteria:** a developer can trace an incident from telemetry through detection to response.

### Phase 7 — Research validation and optimization
- [ ] Create reproducible normal-load and controlled resource-exhaustion scenarios in an isolated, authorized lab.
- [ ] Compare request-level, threshold and temporal multi-signal baselines.
- [ ] Measure precision, recall, PR-AUC where appropriate, false alarms/hour, warning lead time and inference overhead.
- [ ] Test consumer lag, duplicate events, service restarts and dependency failures.
- [ ] Split or add services only when measured requirements justify the added operational complexity.

**Exit criteria:** publishable/reproducible evaluation results and documented limitations.

## 7. Event contract examples

### Telemetry event (illustrative)

```json
{
  "eventId": "evt_example",
  "schemaVersion": "1.0",
  "timestamp": "2026-10-15T05:30:00Z",
  "serviceId": "demo-api",
  "endpointId": "checkout-create",
  "metrics": {
    "requestsPerSecond": 120,
    "p95LatencyMs": 250,
    "errorRate": 0.02,
    "cpuUtilization": 0.65,
    "memoryUtilization": 0.70,
    "dbLatencyMs": 90,
    "queueDepth": 12
  }
}
```

### Detection result (illustrative)

```json
{
  "eventId": "det_example",
  "schemaVersion": "1.0",
  "serviceId": "demo-api",
  "currentState": "DEGRADING",
  "predictedTransition": "RESOURCE_EXHAUSTION_RISK",
  "predictionHorizonSeconds": 60,
  "riskScore": 72,
  "evidence": [
    "LATENCY_TREND_INCREASING",
    "QUEUE_DEPTH_GROWING"
  ],
  "modelVersion": "baseline-v1"
}
```

These are examples, not trained-model outputs. Define units, timestamp semantics, missing-value rules, tenant boundaries, idempotency and schema compatibility before production use. Never treat an uncalibrated score as a probability.

## 8. Engineering rules

1. One service owns each durable business entity.
2. Avoid synchronous chains across many services on the critical request path.
3. Make Kafka consumers idempotent; assume messages may be delivered more than once.
4. Version event schemas and keep backward compatibility where practical.
5. Add timeouts, bounded retries, dead-letter handling and consumer-lag monitoring.
6. Do not log passwords, tokens, secrets or unnecessary personal data.
7. Do not let the ML service directly modify gateway configuration. Route actions through policy and mitigation controls.
8. Start in observe-only mode; test mitigation in an isolated environment and require explicit policy authorization.
9. Add services based on independent scaling, deployment, ownership or security needs—not service count.
10. Keep tests, threat model, architecture decisions and research methodology in the repository.

## 9. Local development conventions

- Keep credentials in local environment variables or a local `.env` file that is excluded from Git.
- `.env.example` contains placeholders only.
- Use a consistent health endpoint such as `/health` for HTTP services.
- Include a readiness check for dependencies where useful.
- Use structured logs with `timestamp`, `service`, `level`, `traceId`/`correlationId` and event IDs.
- Keep model artifacts and large datasets out of Git unless there is a deliberate versioning plan.
- Document each service's port, dependencies, environment variables, input/output contract and test command in its own README.

## 10. First milestone

Do **not** implement every service immediately. The first end-to-end milestone is:

1. NGINX routes a request to the demo application.
2. Telemetry service validates and publishes an event to Kafka.
3. A consumer reads the event and calculates a basic time-window feature.
4. The output is visible in logs or a small diagnostic endpoint.

Only after that path works should the temporal detection service be integrated.

## 11. First local setup

Requirements: Git, Docker Desktop with the Linux engine enabled, Node.js LTS, pnpm, and Python 3.11+.

1. Copy `.env.example` to `.env` and change the local development password.
2. Start infrastructure only:

   ```bash
   docker compose up -d
   docker compose ps
   ```

3. Check dependencies:

   ```bash
   docker compose logs -f kafka postgres redis
   ```

4. Begin implementing Phase 2. Add application services to Compose only after their Dockerfiles and configuration exist.

The included Compose file is an **infrastructure scaffold**, not a claim that the application microservices are already implemented. It is intended for local development, not production. Review image versions and security settings before using it beyond a local environment.
