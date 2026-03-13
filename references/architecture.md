# Architecture Reference

## The North Star

Architecture is a product decision. Every choice answers: does this make us faster and more resilient over time? Start with failure modes, not features. Define consistency requirements explicitly. Map data access patterns before choosing storage. Write a constraints document — data residency, offline requirements, legacy integrations, compliance.

The most sophisticated architecture isn't the most complex. It's the one that holds its shape under pressure.

---

## Domain Modeling

Before a single line of code, obsess over the domain model.

**Ubiquitous Language** — Every term has exactly one meaning. When business people and engineers hear different things, bugs hide in the translation.

**Bounded Contexts** — A large system cannot have one unified model. "Customer" in billing ≠ "customer" in support. Across context boundaries: a translation layer.

**Aggregates** — A cluster of domain objects that must be consistent together, with a root entity controlling access. Transactions never span aggregate boundaries. If you need cross-aggregate transactions, boundaries are wrong or you need eventual consistency. Keep aggregates small.

**Event Sourcing** — Store what happened, not current state. Every change is an immutable event. Current state derived by replay. Gives: complete audit log, time-travel debugging, new read models from history, reliable async communication. Tradeoff: maintain separate read models (projections).

**CQRS** — Separate write path (Commands → validation → domain logic → Events) from read path (pre-joined, pre-aggregated, shaped for consumers). Write optimizes for correctness. Read optimizes for speed. Eventually consistent — design for this explicitly.

---

## Service Architecture

**Start Modular Monolith** — One deployable unit with rigorous internal module boundaries enforced by architecture tests. Module A can't query Module B's tables or instantiate its classes directly.

**Extract services only when forced:** Independent scaling, autonomous deployment needs, different reliability requirements, incompatible runtime.

**Synchronous (gRPC):** When caller needs the result, operation is a query, or latency critical. Strongly-typed Protobuf contracts. Use REST for external APIs where developer experience matters more.

**Asynchronous (Kafka/equivalent):** When caller doesn't need immediate response, multiple consumers exist, or you want temporal decoupling.

**Saga Pattern:** For cross-service atomic operations. Choreography (each service reacts — simpler) or Orchestration (central coordinator with state machine — preferred for non-trivial flows). Compensating transactions undo previous steps on failure.

**Idempotency:** Every mutating operation accepts an idempotency key. Check before executing. Redis with TTL for key storage.

**Outbox Pattern:** Atomic DB write + event publish. Same transaction writes data AND event to outbox table. Relay publishes to message broker. Combined with consumer idempotency: at-least-once with no message loss.

---

## Data Architecture — Polyglot Persistence

Match storage to access patterns, not familiarity.

**PostgreSQL** — Transactional data, consistency paramount. JSONB for semi-structured. Row-level security for multi-tenancy.

**Redis** — Session state, caching, rate limiting, leaderboards, pub/sub. Data structure server. Never as primary store.

**Elasticsearch/OpenSearch** — Full-text search, faceted search, geospatial, complex aggregations.

**ClickHouse** — Column-oriented analytics. Sub-second queries over billions of rows. Product analytics and BI belong here, not Postgres.

**Vector Databases** — Pinecone, Weaviate, pgvector. Embeddings, semantic search, RAG, recommendations.

**Data Warehouse** — Snowflake/BigQuery/Redshift for analytics. Data flows via CDC (Debezium reads WAL → Kafka). dbt for transformations. BI queries warehouse, never production.

**Build the data pipeline from the start.** Retrofitting CDC into a mature system is enormously expensive.

---

## Infrastructure Patterns

**Boring is beautiful.** Use managed services. Every hour managing infrastructure is an hour not spent on product.

**Containers:** CPU/memory requests and limits on every container. Multi-stage builds. No secrets in layers.

**Networking:** Public subnets for load balancers only. Private for services. Isolated for databases. Security groups as micro-firewalls.

**CDN & Edge:** Static assets on CDN. Edge runtimes for auth, personalization, A/B testing — 50ms vs 200ms+.

**Infrastructure as Code:** Terraform or equivalent with remote state and PR review. Nothing manually created.

---

## AI-Native Architecture (If Applicable)

**AI Gateway:** Reverse proxy for model calls. Model routing, rate limiting, cost attribution, semantic caching, fallback routing. Never let services call model APIs directly.

**RAG:** Documents → chunk → embed → vector DB. Query → embed → retrieve → inject as context. Advanced: hybrid search (vector + BM25), re-ranking, query decomposition.

**Eval Pipelines:** Curated dataset, automated judges, regression detection. Treat prompt changes like code — version control, CI eval, staged rollout.

**Async AI:** Expensive inference in job queues. User gets immediate response + notification when ready.

**Cost Attribution:** Every AI call tagged with feature, user/tenant, model, token counts.

---

## Reliability Patterns

**Circuit Breakers:** Closed → open → half-open. Prevents one slow dependency from cascading failures.

**SLOs & Error Budgets:** SLIs (quantitative health) + SLOs (targets). Error budget = gap between SLO and 100%. Budget healthy → ship fast. Budget burning → reliability over features.

**Observability — Four Pillars:**
- Metrics: RED method (Rate, Errors, Duration). p99 > average.
- Logs: Structured JSON. Trace ID, request ID.
- Traces: OpenTelemetry. Visual waterfall.
- Continuous Profiling: CPU/memory from production.

---

## Meta-Principles

**Reversibility over optimization.** Design systems where the cost of changing your mind is low.

**Prefer boring technology.** Reach for novel solutions only when boring ones genuinely can't solve the problem.

**Know what not to build.** Every abstraction has carrying cost. Every component can fail. The best architectures use the fewest components that solve the actual problem.
