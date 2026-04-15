# Architecture Scan Checklist

Use this checklist when scanning GitHub repos with GitHub MCP to inform the tech design doc.

## 1. Repo Structure

- [ ] Top-level directory layout (what lives where?)
- [ ] Key entry points: `main.*`, `index.*`, `app.*`, `server.*`
- [ ] Config files: `.env.example`, `config/`, `settings.*`
- [ ] Infrastructure: `Dockerfile`, `docker-compose.yml`, `k8s/`, `terraform/`, `.github/workflows/`

## 2. Tech Stack

- [ ] Language(s) and runtime versions (`package.json`, `go.mod`, `requirements.txt`, `Gemfile`, `pom.xml`)
- [ ] Frameworks (web framework, ORM, test framework)
- [ ] Key dependencies (message queues, caches, search engines)
- [ ] Database type and migration tooling

## 3. Data Layer

- [ ] Database schema files (`migrations/`, `schema.sql`, `models/`)
- [ ] ORM models or data classes
- [ ] Caching strategy (Redis keys, TTLs)
- [ ] Data retention or archival patterns

## 4. API Surface

- [ ] API definition files (`openapi.yaml`, `swagger.json`, `*.proto`, `schema.graphql`)
- [ ] Router files (list of endpoints)
- [ ] Auth middleware (how is authentication handled?)
- [ ] Rate limiting or quota patterns

## 5. Service Boundaries

- [ ] Service-to-service communication (REST, gRPC, events, queues)
- [ ] Event/message topics or queues (Kafka topics, SQS queues, PubSub)
- [ ] External integrations (third-party APIs, webhooks)

## 6. Observability

- [ ] Logging patterns (structured logs, log levels)
- [ ] Metrics instrumentation (Prometheus, StatsD, Datadog)
- [ ] Tracing setup (OpenTelemetry, Jaeger)
- [ ] Alerting config files

## 7. Deployment

- [ ] CI/CD pipeline files
- [ ] Environment promotion strategy (dev → staging → prod)
- [ ] Feature flag system (if any)
- [ ] Secrets management approach

## 8. Test Patterns

- [ ] Test directory structure
- [ ] Unit vs integration vs e2e split
- [ ] Test fixtures or factories
- [ ] Coverage tooling

---

## Output Format for Each Repo

After scanning, summarize findings as:

```
### Repo: <org/repo>

**Stack:** <language>, <framework>, <DB>
**Entry point:** <file>
**Key patterns:** <2-3 bullet observations>
**Relevant to this PRD:** <how it informs the tech design>
```
