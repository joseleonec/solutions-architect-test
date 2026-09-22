# Solution Plan – BP Internet Banking (Solution Architect Assessment)

Source requirements: [PROBLEM_STATEMENT.md](PROBLEM_STATEMENT.md). Deliverable: one PDF (C4 Context, Container, Component + supporting diagrams and justifications) and a public GitHub repo with the PDF, URL pasted in the CoderPad comments.

## 1. Timeline (72 h limit)

| Phase | Budget | Objective | Exit criteria |
| --- | --- | --- | --- |
| 1. Design decisions | ~28 h (35%) | Fix every architectural decision with 2+ justifications and the options evaluated | Decision register (section 3) complete |
| 2. Diagramming | ~28 h (40%) | Build C4 L1 → L2 → L3, plus onboarding/auth sequence and infrastructure diagrams | Each diagram passes its checkpoint (section 4) |
| 3. Assembly & QA | ~10 h (15%) | Write PDF, run the criteria checklist, publish | Checklist (section 6) fully ticked, PDF readable |
| Buffer | ~6 h (10%) | PDF export issues, GitHub upload, comment on CoderPad | Submitted at least 4 h before the deadline |

Rule: do not start a diagram level until the decisions it depends on are in the register.

## 2. Assumptions (state them in the PDF)

- Cloud: **Azure or AWS is decided in D1**; the rest of the plan is cloud-neutral until then.
- The company's OAuth 2.0 product is treated as a generic IdP (Keycloak / Okta / Entra-class); no vendor is assumed.
- Country: **Ecuador** (no local Azure/AWS region; primary in South Central US (Texas), DR in its official pair North Central US (Illinois), based on measured 103–104 ms RTT; cross-border transfer rules (LOPDP) apply). Regulations cited: LOPDP, Código Orgánico Monetario y Financiero, Superintendencia de Bancos security and operational-risk rules, AML/UAFE, and PCI DSS if cards are involved. Verify exact current versions on official sources before submitting; JPRF-2025-0155 is not used, and Resolución SB-2021-2126 (operational risk) and the 2026 cybersecurity law are cited as reported by the project owner.
- Core platform exposes **REST** APIs (confirmed by the project owner); the adapters still isolate BP's domain from its model.
- Sizing assumption: a **mid-size Ecuadorian bank**, about 1 million customers, 300,000 monthly active users, 60,000 daily active users, 40,000 transfers/day, peak about 15 requests/second on read APIs and 5 transfers/second (illustrative; state as assumption in the PDF and in the cost estimate).
- Targets: availability SLO **99.95%**, DR **RPO ≤ 5 minutes and RTO ≤ 1 hour**, p95 read API latency under 500 ms inside the region (plus the ~103 ms network floor from Ecuador).
- Scope: **cards are out of scope**, so PCI DSS is not a requirement (noted as a future consideration if cards are added).
- Audit design simplified: **no full Event Sourcing** (D11).

## 3. Decision register (Phase 1)

Each row needs: **2+ theoretical justifications, alternatives evaluated, trade-off accepted**.

| ID | Decision | Alternatives to evaluate | Notes |
| --- | --- | --- | --- |
| D1 | Cloud provider and regions | Azure vs AWS | Latency to users, managed services, cost, multi-AZ + paired/secondary region for DR |
| D2 | SPA framework | React vs Angular | Keep short; SPA is not the graded focus |
| D3 | Mobile framework (**2 options required**) | Flutter vs React Native | Performance, biometric/secure-storage plugins, team skills, single codebase |
| D4 | Authentication flow | Auth Code + PKCE vs implicit vs password grant | PKCE for SPA and mobile (public clients); no implicit, no password grant; system browser on mobile |
| D5 | Onboarding with facial recognition | SaaS (Jumio, Onfido) vs cloud-native (Azure AI Face / AWS Rekognition) vs in-house | Document scan + liveness + anti-injection; manual-review fallback; biometric data minimization and retention |
| D6 | Post-onboarding login | Password + MFA vs passkeys/FIDO2 vs device biometrics | Fingerprint unlocks a device-bound key, not a server-side check; step-up auth for transfers |
| D7 | Integration layer | API Gateway + BFF vs direct service calls | Routing, throttling, WAF, JWT validation, one BFF per client type |
| D8 | Service decomposition | Basic data, Movements, Transfers (mandatory) + Notification, Audit, Frequent-client cache, Aggregation | Justify each added service by performance or responsiveness |
| D9 | Sync vs async communication | REST/HTTPS vs gRPC vs messaging (AMQP/Kafka) | Sync for reads, async for notifications and audit |
| D10 | Transfer reliability | Saga vs 2PC | Idempotency keys, outbox pattern, compensation for interbank failures |
| D11 | Audit design | Full Event Sourcing vs outbox + append-only store | Immutable, tamper-evident, queryable; prefer the lighter option unless justified |
| D12 | Frequent-client persistence | Cache-aside (Redis) vs CQRS read model vs materialized view | Define what "frequent" means and how invalidation works |
| D13 | Data access | DB per service vs shared DB; SQL vs NoSQL per workload | Encryption at rest, least-privilege access |
| D14 | Notifications (**2+ required**) | SMS providers (2), email; push deferred to a later stage | Fallback between channels, retries, dead-letter queue |
| D15 | Resilience and DR | Active-active vs active-passive | Circuit breaker, retry with backoff, bulkhead, health probes, auto-scaling, self-healing, RPO/RTO |
| D16 | Security | Zero trust, network segmentation, secrets management, mTLS | WAF, DDoS protection, HSM/KMS, PAM, tokenization |
| D17 | Observability | Logs, metrics, traces, alerting | Cloud-native monitor + OpenTelemetry, SLO-based alerts, runbooks |
| D18 | Cost management | Reserved vs on-demand, caching, right-sizing | Include a rough monthly estimate and top cost drivers |
| D19 | Compliance mapping | Regulation → control table | Data protection, AML/KYC, PCI DSS (if cards), audit retention, data residency of biometric vendor |
| D20 | Delivery strategy: staged rollout and pretotyping | Big-bang vs staged with gates vs prototype-then-rebuild | Four stages (pretotype, read-only MVP, transactional, enterprise HA), gates with hypotheses, resilience floor, cost curve, technical-debt register |

## 4. Diagramming (Phase 2), with checkpoints

Build in draw.io, keep the `.drawio` sources, export PNG/SVG. Status: **done**, see [diagrams/README.md](diagrams/README.md). Rules applied (Simon Brown's C4 guidance): one level per diagram, no deployment concepts in Container diagrams, each topic or queue as its own container, one container per Component diagram, dynamic diagrams for runtime behaviour.

1. **C4 L1 Context** (non-technical): customers, BP staff/support, internet banking system, Core platform, complementary customer-data system, IdP, identity-verification vendor, notification providers, interbank network. Every arrow labeled with a plain-language purpose.
   *Checkpoint: a non-technical reader can explain the diagram in one minute.*
2. **C4 L2 Container**: SPA, mobile app, API Gateway/BFF, IdP, microservices, databases, cache, message broker, audit store, cloud components shown without deep detail. Each arrow: protocol plus a few words.
   *Checkpoint: every requirement in the statement maps to at least one container.*
3. **C4 L3 Component** (main services: Transfers, Movements, Onboarding, Audit): controllers, application services, domain, repositories, adapters to Core and external systems, the patterns chosen (CQRS/outbox/saga/circuit breaker), protocols and security boundaries (TLS, mTLS, JWT validation, scopes).
   *Checkpoint: each pattern in the register appears on a diagram, and each security boundary is explicit.*
4. **Deployment diagram** (infrastructure and DR): edge, primary and DR regions, zones, replication, private links to the BP data center, failover path. Kept separate from the Container diagram (no deployment concepts at level 2).
5. **Dynamic diagrams**: mobile sign-in, onboarding, interbank transfer with notification and audit (numbered runtime steps).

Onboarding flow to model (per D5/D6): Onboarding Service orchestrates the vendor SDK (document + liveness) → receives signed result → provisions the user in the IdP with a verified-identity claim → normal Auth Code + PKCE login → device key registration.

## 5. Document assembly (Phase 3)

PDF outline: **maximum 15 pages in total**, written in Spanish. The page-by-page budget, the business-process table and the split between PDF and repository are in [pdf-outline.md](pdf-outline.md).

Repo contents: the 15-page PDF, README (one-paragraph summary + how the deliverable is organized), the full decision register, all 14 diagrams as `.drawio` sources and image exports.

## 6. Grading-criteria checklist (fill during Phases 1–2, not only at the end)

| # | Criterion | Covered by | Done |
| --- | --- | --- | --- |
| 1 | Meets requirements, all justified | Decision register | ☐ |
| 2 | Quality/depth of C4 diagrams | Section 4 | ☐ |
| 3 | Responsibility segmentation, decoupling | D8, D9 | ☐ |
| 4 | Architecture patterns | D10–D12, D15 | ☐ |
| 5 | External service integration | D7, D14, Core/interbank adapters | ☐ |
| 6 | Front-end and mobile architecture | D2, D3 | ☐ |
| 7 | Data access architecture | D12, D13 | ☐ |
| 8 | Cloud knowledge | D1, infrastructure diagram | ☐ |
| 9 | Cost management | D18 | ☐ |
| 10 | Authentication architecture | D4, D6 | ☐ |
| 11 | Onboarding integration | D5, onboarding sequence | ☐ |
| 12 | Audit solution | D11 | ☐ |
| 13 | Banking regulations and security standards | D16, D19 | ☐ |
| 14 | HA and fault tolerance | D15, infrastructure diagram | ☐ |
| 15 | Monitoring | D17 | ☐ |

## 7. Submission checklist

- ☐ PDF renders correctly, diagrams legible at print size
- ☐ Public GitHub repo created and PDF pushed
- ☐ Repo URL pasted in the CoderPad comments
- ☐ PDF uploaded as the exercise answer before the deadline

## 8. Audience and writing guide

Source: the Devsu job posting (<https://apply.workable.com/devsu/j/44BDCDDA17>), "Arquitecto de Soluciones – Líder Técnico de Desarrollo". It asks for someone who can translate business objectives into technical solutions **with Product Owners**, manage the technical roadmap and technical debt, review designs and mentor teams, with "exceptional communication skills to simplify technical concepts". The PDF is therefore read by two audiences: **technical leaders** (depth, patterns, trade-offs) and **Product Owners / business stakeholders** (value, risk, cost, roadmap).

Writing rules:
1. **Two layers per topic.** Start each section with a short plain-language summary (what, why, business impact, risk), then the technical detail. A Product Owner should be able to stop after the summary.
2. **Business impact for every decision.** Each entry in the decision register carries a business-impact note: effect on customers, cost, time to market, risk, and any decision the business or Legal must make.
3. **Diagrams for two readers.** The Context diagram is the Product Owner's diagram (no technology names beyond systems and people). Container and Component diagrams carry the technical depth. Add a one-paragraph reading guide under each.
4. **Define jargon once.** Include a short glossary (PKCE, BFF, CQRS, RPO/RTO, saga, outbox) and avoid unexplained acronyms in summaries.
5. **Show the roadmap and technical debt, since the role owns them.** Add a phased delivery roadmap (MVP: movements + transfers; next: notifications, frequent-client optimization; later: new products) and a technical-debt / evolution section that names the shortcuts accepted now (for example DR region without availability zones) and when to revisit them.
6. **Be explicit about risks and open questions**, with an owner (Legal, Security, Product) for each, instead of hiding them.
7. **Quantify where possible:** latency measured, availability targets, estimated monthly cost drivers.
8. **Keep it scannable:** short paragraphs, tables for comparisons, one decision per page where possible.

Language: the exercise and the job posting are in Spanish, so the final PDF should be **written in Spanish** (Ecuadorian financial-sector audience). The working files are in English; translate at the assembly phase (Phase 3), keeping technical terms (API Gateway, BFF, CQRS) in English as is common in the industry.
