---
layout: post
title: "Decomposing a Marketplace Monolith"
date: 2026-02-10
categories: architecture microservices
---

# Decomposing a Marketplace Monolith

## Context

Our marketplace platform started as a monolithic Rails application serving buyers, sellers, and internal operations teams. After five years of growth, the codebase reached 500K+ lines of code with 150+ database tables. The monolith handled everything from user authentication to payment processing, inventory management, and recommendation engines.

The engineering team grew to 50+ developers working across multiple product verticals. Deploy cycles stretched to 2-3 weeks due to extensive regression testing requirements. A single bug in the checkout flow could block deployments for the entire platform.

Database query performance degraded as traffic scaled to millions of daily active users. Hot tables like `orders` and `inventory` became bottlenecks, causing cascading failures during peak hours.

## Problem

The monolithic architecture created several critical challenges:

**Deployment bottlenecks**: Any change required deploying the entire application, making releases slow and risky. Teams couldn't iterate independently.

**Scaling limitations**: We couldn't scale individual components based on their specific load patterns. The recommendation engine needed GPU instances, while the API servers needed CPU-optimized instances.

**Technology lock-in**: The entire platform was tied to Ruby on Rails. We couldn't adopt better-suited technologies for specific domains (e.g., Go for high-throughput services, Python for ML pipelines).

**Database contention**: All services shared a single PostgreSQL cluster. Lock contention on hot tables caused timeouts and degraded user experience.

**Developer productivity**: Build times exceeded 10 minutes. Running the full test suite took 45 minutes. New engineers needed weeks to understand the codebase.

## Decision

We decided to incrementally decompose the monolith using the Strangler Fig pattern, starting with the highest-value, lowest-risk services:

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

```ruby
# Monolith API: Gradual migration using feature flags
class RecommendationsController < ApplicationController
  def index
    if FeatureFlags.enabled?(:recommendations_service_v2, current_user)
      # Call new microservice
      response = RecommendationsClient.get_recommendations(
        user_id: current_user.id,
        context: params[:context]
      )
      render json: response
    else
      # Fallback to legacy monolith code
      recommendations = RecommendationEngine.generate(current_user)
      render json: recommendations
    end
  end
end
```

```python
# New Recommendations Microservice (Python + FastAPI)
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session
import numpy as np

app = FastAPI()

@app.get("/api/v1/recommendations")
async def get_recommendations(
    user_id: int,
    context: str,
    db: Session = Depends(get_db)
):
    # ML model inference
    user_embedding = await get_user_embedding(user_id)
    item_embeddings = await get_item_embeddings(context)
    
    scores = np.dot(user_embedding, item_embeddings.T)
    top_items = np.argsort(scores)[-10:]
    
    return {
        "user_id": user_id,
        "recommendations": top_items.tolist(),
        "algorithm": "collaborative_filtering_v2"
    }
```

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

*This post reflects our architectural journey. Your context and constraints may differ—evaluate trade-offs carefully before adopting similar patterns.*
