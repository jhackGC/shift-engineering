---
layout: post
title: "Decomposing a Marketplace Monolith Back End into Microservices"
date: 2026-02-10
categories: architecture microservices node c# azure nestjs
---

# Decomposing a Marketplace Monolith Back End into Microservices

## Context

Our marketplace platform started as a monolithic NodeJS/NestJS application deployed in a single Lambda function in AWS.

It grew, and it grew big, serving customers, sellers, and internal operations teams. But it did the job, we needed to focus on building features and scaling the business, not on architectural purity. But now it needs to scale, and it needs to be maintainable.

After five years of growth, the codebase was becoming unmanageable.

The monolith handled everything from customer-facing site, payment processing, inventory management, user management, and recommendation engines, etc.

We wanted to break it down into microservices to improve scalability, reliability, and developer productivity. But we had to do it without disrupting the business or causing downtime. And slo, microservice are more complex to build and maintain, you have the orchestration, the messaging, the monitoring, the logging, the data consistency, the network latency, etc. So we had to be careful and intentional about how we decomposed the monolith.

We didn't want to jump into a microservice architecture without a clear plan and real need for it. We had to identify the right boundaries, the right services, the right data models, and the right communication patterns. We had to balance the benefits of microservices with the costs and trade-offs.

## Problem

The monolithic architecture created several critical challenges:

**Deployment bottlenecks**: Any change required deploying the entire application (customer, admin, vendor apps), making releases slow and risky.

**Scaling limitations**: The lambda functions were reaching their size limit.

**Technology lock-in**: The entire platform was tied to AWS Lamba functions.

**Database contention**: All services shared a single MongoDB cluster.

## Decision

We decided to incrementally decompose the monolith using a BDD approach, starting breaking the services by domain (customer, admin, vendor)

**Phase 1: Extract Read-Heavy Services**

- Recommendation Engine (serving 100M+ requests/day)
- Product Search (isolated data model, clear API contract)

**Phase 2: Extract Business-Critical Transactional Services**

- Payment Processing (PCI compliance requirements favored isolation)
- Inventory Management (needed real-time updates with sub-second latency)

**Phase 3: Extract Remaining Services**

- User Management
- Order Management
- Analytics & Reporting

**Technical Approach:**

**Data Strategy:**

- Event-driven synchronization using Kafka
- Dual-write pattern during migration
- Eventually consistent reads acceptable for non-critical paths

**Infrastructure:**

- Kubernetes for container orchestration
- Istio service mesh for traffic management and observability
- PostgreSQL per service, shared RDS cluster for cost optimization

## Trade-offs

**Benefits:**

- **Independent deployment**: Teams could ship features 3x faster (weekly vs. monthly releases)
- **Technology flexibility**: Adopted Python for ML services, Go for high-throughput APIs
- **Targeted scaling**: Reduced infrastructure costs by 40% through right-sized resources
- **Improved reliability**: Service isolation prevented cascading failures
- **Better developer experience**: Smaller codebases improved onboarding and productivity

**Costs:**

- **Operational complexity**: Monitoring, logging, and debugging across 15+ services required new tools (Datadog, ELK stack)
- **Data consistency**: Eventual consistency model required careful design of business workflows
- **Network latency**: Inter-service communication added 10-50ms per hop (mitigated with caching)
- **Development overhead**: Service boundaries required well-defined APIs and contracts
- **Migration effort**: Full decomposition took 18 months with a dedicated platform team

**Technical Debt:**

- Some services still shared database tables during transition
- Inconsistent API versioning across early services
- Monitoring gaps during migration led to blind spots

## Outcome

After 18 months of incremental migration:

**Performance Improvements:**

- API response times improved by 60% (p95: 800ms → 320ms)
- Recommendation service scaled to 500M requests/day
- Database query performance improved 3x with focused indexing per service

**Business Impact:**

- Deployment frequency: 2-3 weeks → daily deployments
- Incident MTTR reduced by 50% (clearer service boundaries)
- Feature velocity increased 3x (parallel team development)

**Engineering Culture:**

- Teams took ownership of services end-to-end
- On-call rotation became service-specific, improving expertise
- Experimentation culture flourished with lower deployment friction

**Lessons Learned:**

1. **Start with clear service boundaries**: Services with well-defined domains migrated smoothly
2. **Invest in observability early**: Distributed tracing was essential for debugging
3. **Feature flags are critical**: Enabled safe, gradual rollouts and quick rollbacks
4. **Don't underestimate data migration**: Schema changes and data consistency were the hardest parts
5. **Team structure matters**: Aligned teams with service ownership for best results

The decomposition transformed our ability to scale both technically and organizationally. While microservices aren't a silver bullet, the intentional extraction of services aligned with business domains proved successful for our marketplace platform.

---

_This post reflects our architectural journey. Your context and constraints may differ—evaluate trade-offs carefully before adopting similar patterns._
