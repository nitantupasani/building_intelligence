# Smart Building Platform Backend Implementation Plan

This plan translates your target architecture into **execution steps you (the team) need to perform manually**.

## 1) Finalize Architecture Decisions (Week 1)
1. Select cloud and region strategy (AWS/GCP/Azure, primary + DR region).
2. Choose API stack per service (Node/Fastify vs Go/Gin), and lock a service-by-service language matrix.
3. Select managed providers for Kafka, PostgreSQL/Timescale, Redis, and Elasticsearch.
4. Approve tenancy model for launch (`shared DB + per-tenant schema + RLS`) and migration criteria to dedicated DB.
5. Define non-functional targets (SLOs, latency budgets, RTO/RPO, retention timelines) as signed engineering requirements.

## 2) Security & Compliance Foundations (Week 1-2)
1. Create data classification policy (PII vs non-PII) and field-level encryption policy.
2. Define GDPR workflows and legal sign-off:
   - Right to Access export SLA.
   - Right to Erasure pseudonymization approach.
   - Consent record format/versioning.
3. Define SOC2/ISO27001 evidence expectations for audit logging, access reviews, and retention.
4. Establish key management decisions (KMS strategy, rotation cadence, ownership model).
5. Approve Admin/FM MFA policy and service-to-service trust model (mTLS through mesh).

## 3) Platform Bootstrap (Week 2-4)
1. Provision Kubernetes clusters:
   - Primary multi-AZ production cluster.
   - Warm standby cluster in DR region.
2. Set up network/security baseline:
   - Private VPC networking.
   - WAF, gateway ingress, and restricted admin paths.
   - No public IPs for internal services.
3. Create container registry, Helm/ArgoCD GitOps repos, and environment promotion workflow.
4. Provision managed data stack:
   - PostgreSQL 16.
   - TimescaleDB.
   - Redis 7 (cluster mode when needed).
   - Kafka.
   - Object storage for exports/snapshots.
5. Establish secrets manager and bootstrap secret rotation process.

## 4) Identity, Tenanting, and Access Control (Week 3-5)
1. Implement tenant-scoped JWT claim contract (`tenantId`, `userId`, `role`, `buildingIds`, `exp`, `iat`).
2. Implement refresh token rotation and reuse-detection invalidation policy.
3. Build and adopt a shared RBAC SDK used by all services.
4. Implement DB tenancy isolation:
   - Per-tenant schemas.
   - Mandatory RLS as defense in depth.
5. Run cross-tenant penetration tests focused on API, cache namespaces, and DB access paths.

## 5) Domain Services Delivery Sequence (Week 4-10)
Implement in this order to reduce integration risk:

1. **Building/Tenant Service** (source of truth first).
2. **Config/Schema Registry Service** (required before vote integrity controls).
3. **Vote Ingestion + Vote Processor**:
   - `votes.raw` ingest path.
   - Idempotency (`voteUuid`, Redis SETNX).
   - Schema validation + migration routing.
   - DLQ pipeline + operational alerting.
4. **Presence Service** with weighted confidence model + TTL expiry sweeps.
5. **Analytics Service**:
   - Hot path (Redis rolling aggregates + SSE/WebSocket).
   - Cold path (scheduled materializations).
6. **Notification Service** (alerts, feedback loop, FM broadcasts).
7. **Audit Service** (append-only + immutable snapshot exports).

## 6) Eventing Contract Governance (Week 5-8)
1. Publish canonical event schemas for each Kafka topic.
2. Add schema validation in producer CI and consumer startup checks.
3. Define partition keys per topic and confirm expected ordering guarantees.
4. Define replay/recovery runbooks for consumer offset resets and DLQ reprocessing.
5. Create PagerDuty policies for DLQ depth and consumer lag SLO breaches.

## 7) Offline-First Backend Support (Week 7-9)
1. Implement `/api/v1/votes/batch` with per-item status reporting.
2. Enforce 100-item batch cap and strict schema/timestamp validation.
3. Add clock skew policy handling (`late_submission`, future timestamp rejection).
4. Ensure endpoint always returns `200` for partial failures with itemized statuses.
5. Run high-retry soak tests simulating reconnect storms from mobile clients.

## 8) SRE, Observability, and Operations (Week 6-10)
1. Adopt OpenTelemetry standard and enforce trace propagation from gateway.
2. Ship service dashboards for golden signals + tenant-segmented health views.
3. Define and activate alert routing for P1/P2 classes matching your SLO table.
4. Implement zero-downtime deployment controls:
   - expand/contract DB migrations,
   - rolling deploy defaults,
   - canary policy for risky changes.
5. Create incident response runbooks for:
   - Kafka lag spikes,
   - DB failover,
   - token theft/reuse events,
   - regional failover.

## 9) Validation Gates Before Production Launch
1. Perform load test at projected shift-change peak traffic.
2. Verify idempotency under replay storms (no double count across retries).
3. Validate DR failover against RTO/RPO targets.
4. Execute GDPR request drills end-to-end (access + erasure).
5. Conduct security review (OWASP API, tenancy isolation, secrets handling, mTLS).
6. Require sign-off from Engineering, Security, and Product before enabling tenant onboarding.

## 10) What You Must Own (Not Delegable)
These decisions/actions require your direct ownership and cannot be safely automated by an assistant alone:

1. Vendor and cloud contract choices (cost, legal, residency, procurement constraints).
2. Legal/compliance sign-off (GDPR/SOC2/ISO interpretations and policy approvals).
3. Production access governance (who can access what, break-glass process, audit policy).
4. Incident management operating model (on-call staffing, escalation matrix, pager rotations).
5. Risk acceptance decisions (availability vs cost, retention length vs privacy posture).
6. Final production go/no-go approval.

---

## Suggested 30/60/90-Day Milestones
- **Day 30**: Platform/bootstrap complete, auth/tenant model live, Building + Config services operational.
- **Day 60**: Vote/Presence/Analytics hot path complete, observability + SLO alerting active.
- **Day 90**: DR tested, GDPR pipelines verified, audit retention jobs active, production launch ready.
