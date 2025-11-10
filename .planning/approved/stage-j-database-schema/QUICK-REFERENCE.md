# DATABASE SCHEMA - QUICK REFERENCE

**Stage J: Complete Database Architecture**
**Version:** 1.0.0 | **Date:** November 10, 2025

---

## Quick Stats

- **Total Tables:** 30
- **Database:** Cloudflare D1 (SQLite)
- **Vector Storage:** Cloudflare Vectorize (3 indexes)
- **Migrations:** 5 migration scripts
- **Indexes:** 40+ performance indexes

---

## Core Tables (3)

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `tenants` | Multi-tenant root | tenant_id, region, kms_key_id |
| `users` | User accounts (RBAC) | user_id, tenant_id, role |
| `brand_profiles` | Private brand voice (ENCRYPTED) | tenant_id, embeddings, palette |

---

## Content Generation (10 Tables)

| Table | Purpose | Privacy |
|-------|---------|---------|
| `skills` | Shared skill library | GLOBAL (shared) |
| `skill_versions` | Version control | GLOBAL |
| `skill_embeddings` | Similarity search | GLOBAL |
| `skill_metrics_daily` | Performance metrics | Aggregated |
| `content_templates` | 5 post templates | PER-TENANT |
| `generated_content` | Generation history | PER-TENANT |
| `post_configurations` | Posting preferences | PER-TENANT |
| `revision_events` | Defect/preference logs | PER-TENANT |
| `guard_results` | Quality check results | PER-TENANT |
| `brand_knowledge_entries` | Knowledge base | PER-TENANT |

---

## Publishing (3 Tables)

| Table | Purpose | Security |
|-------|---------|----------|
| `social_accounts` | OAuth tokens | ENCRYPTED |
| `published_posts` | Publishing history | PER-TENANT |
| `rate_limits` | API throttling | PER-TENANT |

---

## Analytics (5 Tables)

| Table | Purpose |
|-------|---------|
| `post_analytics` | Platform metrics (likes, shares, etc.) |
| `performance_benchmarks` | Rolling 90-day benchmarks |
| `ab_tests` | A/B test tracking |
| `conversions` | ROI & attribution |
| `analytics_sync_errors` | Error logging |

---

## Security (3 Tables)

| Table | Purpose |
|-------|---------|
| `api_keys` | API authentication |
| `audit_logs` | Compliance audit trail |
| `sharing_events` | Skill promotion audit |

---

## Vector Indexes (Cloudflare Vectorize)

| Index | Dimensions | Purpose |
|-------|------------|---------|
| `nds-brand-voice` | 1536 (text-embedding-3-small) | Brand voice similarity |
| `nds-skills` | 1024 (voyage-3-lite) | Skill matching |
| `nds-knowledge` | 1536 (text-embedding-3-small) | Knowledge base search |

---

## Multi-Tenant Isolation

**SHARED (Safe to share across tenants):**
- ✅ Skills (structural patterns, layouts)
- ✅ Guards (quality rules)
- ✅ Performance metrics (aggregated, K >= 5)

**PRIVATE (Never shared):**
- ❌ Brand voice (tone, lexicon, messaging)
- ❌ Brand assets (colors, typography, logos)
- ❌ OAuth tokens (encrypted per-tenant)
- ❌ Individual performance data

**Enforcement:**
- Row-Level Security (RLS) policies
- Per-tenant KMS encryption
- K-anonymity (>= 5 tenants)
- Application-layer validation

---

## Key Indexes

**Performance-Critical:**
```sql
-- Users
CREATE INDEX idx_users_tenant ON users(tenant_id);
CREATE INDEX idx_users_email ON users(email);

-- Skills
CREATE INDEX idx_skills_task_platform ON skills(task, platform, modality, status);

-- Posts
CREATE INDEX idx_published_posts_tenant ON published_posts(tenant_id, published_at DESC);
CREATE INDEX idx_published_posts_platform ON published_posts(platform, published_at DESC);

-- Analytics
CREATE INDEX idx_post_analytics_tenant ON post_analytics(tenant_id, synced_at DESC);
```

---

## Common Queries

### Get Tenant Brand Profile
```sql
SELECT * FROM brand_profiles WHERE tenant_id = ?;
```

### Find Top Skills
```sql
SELECT s.skill_id, s.name, AVG(m.approval_rate) as avg_approval
FROM skills s
JOIN skill_metrics_daily m ON s.skill_id = m.skill_id
WHERE m.date >= date('now', '-30 days')
GROUP BY s.skill_id
ORDER BY avg_approval DESC
LIMIT 10;
```

### Get Recent Posts
```sql
SELECT * FROM published_posts
WHERE tenant_id = ?
ORDER BY published_at DESC
LIMIT 50;
```

### Calculate Engagement Rate
```sql
SELECT
  post_id,
  (likes + comments + shares + clicks) * 1.0 / NULLIF(impressions, 0) as engagement_rate
FROM post_analytics
WHERE tenant_id = ?;
```

---

## Migration Order

1. **001_create_core_tables.sql** - tenants, users, brand_profiles
2. **002_create_content_generation_tables.sql** - skills, templates, content
3. **003_create_publishing_tables.sql** - social_accounts, published_posts
4. **004_create_analytics_tables.sql** - post_analytics, conversions
5. **005_create_security_tables.sql** - api_keys, audit_logs

---

## Security Checklist

- [ ] RLS policies enabled on sensitive tables
- [ ] Per-tenant KMS keys configured
- [ ] K-anonymity checks in aggregations (>= 5 tenants)
- [ ] OAuth tokens encrypted at rest
- [ ] Audit logging enabled
- [ ] Regional data residency configured
- [ ] API rate limiting enforced

---

## Performance Optimization

**Best Practices:**
1. Use cursor-based pagination (not OFFSET)
2. Pre-compute daily aggregates
3. Create covering indexes for hot queries
4. Use composite indexes for multi-column filters
5. Analyze queries with EXPLAIN QUERY PLAN

**Example: Cursor Pagination**
```sql
-- GOOD
SELECT * FROM published_posts
WHERE tenant_id = ? AND published_at < ?
ORDER BY published_at DESC
LIMIT 50;

-- BAD
SELECT * FROM published_posts
WHERE tenant_id = ?
ORDER BY published_at DESC
LIMIT 50 OFFSET 10000;  -- Slow!
```

---

## RBAC Roles

| Role | Permissions |
|------|-------------|
| `owner` | Full access, billing, delete account |
| `admin` | Manage users, settings, content |
| `member` | Create/edit content, publish posts |
| `viewer` | Read-only access |

---

## Rate Limits (Per Platform)

| Platform | Limit |
|----------|-------|
| LinkedIn | 100 posts/user/day |
| Twitter | 50 posts/user/day |
| Facebook | 200 API calls/hour |
| Instagram | 25 posts/user/day |

---

## Cost Estimates

**Monthly Operational Costs (1000 posts):**
- API Costs: $332
- Infrastructure (Cloudflare): $55
- **Total:** $387/month ($0.39/post)

**After 6 months (mature system):**
- API Costs: $220 (skill reuse optimization)
- Infrastructure: $55
- **Total:** $275/month ($0.28/post)

---

## Support

**Full Documentation:** See `STAGE-J-DATABASE-SCHEMA.md`
**Questions:** Database Architecture Team
**Last Updated:** November 10, 2025
