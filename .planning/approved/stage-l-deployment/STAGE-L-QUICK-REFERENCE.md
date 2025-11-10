# Stage L: Production Deployment - Quick Reference Card

**Version**: 1.0.0
**Date**: November 10, 2025

---

## 🚀 Quick Start: Deploy DIFY-AGENT to Production

### Prerequisites Checklist

- [ ] GitHub account with access to dify-agent repository
- [ ] Vercel account (Pro plan)
- [ ] Railway account (Pro plan) OR Dify Cloud subscription
- [ ] Cloudflare account (Workers Paid plan)
- [ ] Google Cloud account (GKE access)
- [ ] Domain name configured (dify-agent.com)
- [ ] All API keys obtained (OpenAI, Google Vision, Twilio, LinkedIn)

---

## 📋 Deployment Order (Critical Path)

**Follow this exact order to avoid dependency issues:**

1. **Set up Infrastructure** (Day 1-2)
   - Create GKE cluster (8 nodes)
   - Configure Cloudflare Workers routes
   - Set up Railway environments

2. **Deploy Backend Services** (Day 3-4)
   - Deploy MCP Gateway to Cloudflare Workers
   - Deploy Dify backend to Railway
   - Deploy OpenManus agents to GKE

3. **Deploy Frontend** (Day 5)
   - Configure Vercel project
   - Set environment variables
   - Deploy Next.js app

4. **Configure Monitoring** (Day 6-7)
   - Set up Grafana dashboards
   - Configure Sentry error tracking
   - Set up PagerDuty alerting

5. **Testing & Validation** (Day 8-9)
   - Run E2E tests on staging
   - Load test with k6 (100 concurrent users)
   - Security scan with OWASP ZAP

6. **Production Launch** (Day 10)
   - Deploy to production
   - Monitor for 24 hours
   - Document deployment

---

## 🌍 Environment URLs

| Environment | Frontend | Dify API | MCP Gateway | OpenManus |
|-------------|----------|----------|-------------|-----------|
| **Local** | http://localhost:3000 | http://localhost:5001 | http://localhost:8787 | http://localhost:8000 |
| **Development** | https://dev.dify-agent.com | https://dify-api-dev.dify-agent.com | https://mcp-dev.dify-agent.com | https://openmanus-dev.dify-agent.com |
| **Staging** | https://staging.dify-agent.com | https://dify-api-staging.dify-agent.com | https://mcp-staging.dify-agent.com | https://openmanus-staging.dify-agent.com |
| **Production** | https://app.dify-agent.com | https://dify-api.dify-agent.com | https://mcp.dify-agent.com | https://openmanus.dify-agent.com |

---

## ⚙️ Component Deployment Commands

### Frontend (Vercel)

```bash
# Deploy to production
cd frontend
vercel --prod

# Deploy to staging
vercel --prod --alias staging.dify-agent.com

# Rollback (via Vercel dashboard)
# https://vercel.com/your-org/dify-agent/deployments
```

---

### MCP Gateway (Cloudflare Workers)

```bash
# Deploy to production
cd mcp-gateway-final
pnpm install --frozen-lockfile
pnpm wrangler deploy

# Deploy to staging
pnpm wrangler deploy --env staging

# Rollback
pnpm wrangler rollback --message "Rollback to previous version"

# Tail logs
pnpm wrangler tail
```

---

### Dify Backend (Railway)

```bash
# Deploy via Railway CLI
npm install -g @railway/cli
railway login
railway up --service dify-api --environment production
railway up --service dify-worker --environment production

# OR deploy via Railway dashboard
# https://railway.app/project/dify-agent/deployments

# Rollback: Select previous deployment in Railway dashboard
```

---

### OpenManus (Kubernetes on GKE)

```bash
# Connect to cluster
gcloud container clusters get-credentials dify-agent-prod --zone us-central1-a

# Deploy agents
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/secrets.yaml
kubectl apply -f k8s/deployment-redis.yaml
kubectl apply -f k8s/deployment-agents.yaml
kubectl apply -f k8s/hpa.yaml

# Check status
kubectl get pods -n openmanus-production
kubectl get svc -n openmanus-production

# Rollback
kubectl rollout undo deployment/openmanus-agents -n openmanus-production

# View logs
kubectl logs -f deployment/openmanus-agents -n openmanus-production --tail=100
```

---

## 🔑 Environment Variables (Production)

### Frontend (Vercel)

```bash
NEXT_PUBLIC_DIFY_API_URL=https://dify-api.dify-agent.com
NEXT_PUBLIC_MCP_API_URL=https://mcp.dify-agent.com
NEXT_PUBLIC_SENTRY_DSN=https://...@sentry.io/production
NODE_ENV=production
```

### Dify Backend (Railway)

```bash
MODE=api # or worker, sandbox
SECRET_KEY=${DIFY_SECRET_KEY} # Stored in Railway secrets
DB_USERNAME=${POSTGRES_USER}
DB_PASSWORD=${POSTGRES_PASSWORD}
DB_HOST=${POSTGRES_HOST}
REDIS_HOST=${REDIS_HOST}
OPENAI_API_KEY=${OPENAI_API_KEY}
```

### MCP Gateway (Cloudflare Workers)

```bash
# Set via wrangler secrets
wrangler secret put TWILIO_AUTH_TOKEN
wrangler secret put LINKEDIN_ACCESS_TOKEN
wrangler secret put OPENAI_API_KEY
wrangler secret put SUPABASE_SERVICE_KEY
```

### OpenManus (Kubernetes Secrets)

```bash
# Create secrets
kubectl create secret generic openmanus-secrets \
  --from-literal=openai-api-key=$OPENAI_API_KEY \
  --from-literal=google-vision-api-key=$GOOGLE_VISION_API_KEY \
  -n openmanus-production
```

---

## 📊 Monitoring Dashboards

| Service | URL | Purpose |
|---------|-----|---------|
| Grafana | https://grafana.dify-agent.com | System metrics, business metrics |
| Sentry | https://sentry.io/organizations/dify-agent | Error tracking, performance |
| Datadog | https://app.datadoghq.com | APM, distributed tracing |
| PagerDuty | https://dify-agent.pagerduty.com | On-call alerting |
| UptimeRobot | https://uptimerobot.com | Uptime monitoring |

---

## 🚨 Emergency Procedures

### Scenario: High Error Rate (>1%)

```bash
# 1. Check Sentry for error details
# https://sentry.io/organizations/dify-agent/issues/

# 2. Rollback immediately
# Frontend (Vercel): Use dashboard
# MCP Gateway:
cd mcp-gateway-final && pnpm wrangler rollback

# OpenManus:
kubectl rollout undo deployment/openmanus-agents -n openmanus-production

# Dify: Use Railway dashboard to rollback

# 3. Notify team
# Slack: #production-alerts
```

---

### Scenario: Complete System Outage

```bash
# 1. Check health endpoints
curl https://app.dify-agent.com/health
curl https://dify-api.dify-agent.com/health
curl https://mcp.dify-agent.com/health
curl https://openmanus.dify-agent.com/health

# 2. Check component status
kubectl get pods -n openmanus-production
railway status --service dify-api

# 3. Restart services if needed
kubectl rollout restart deployment/openmanus-agents -n openmanus-production
railway restart --service dify-api

# 4. Start war room
# Slack: Create #incident-war-room-YYYY-MM-DD
# Assign Incident Commander
```

---

### Scenario: Database Corruption

```bash
# 1. Stop writes
kubectl scale deployment openmanus-agents --replicas=0 -n openmanus-production
railway scale --service dify-worker --replicas 0

# 2. Restore from backup
railway run --environment production pg_restore -d dify dify-backup-latest.dump

# 3. Verify data integrity
railway run --environment production psql -d dify -c "SELECT COUNT(*) FROM published_posts;"

# 4. Resume operations
kubectl scale deployment openmanus-agents --replicas=8 -n openmanus-production
railway scale --service dify-worker --replicas 5
```

---

## 💰 Cost Summary (Quick Reference)

| Environment | Monthly Cost | Per-Customer (100) | Notes |
|-------------|-------------|-------------------|-------|
| **Local** | $0 | $0 | Docker Compose |
| **Development** | $150 | $1.50 | Shared dev environment |
| **Staging** | $500 | $5.00 | Production replica |
| **Production** | $1,586-$2,286 | $15.86-$22.86 | Auto-scales with usage |

**Break-Even**: 38 customers @ $25/month ARPU
**Payback Period**: 6.4 months at 100 customers
**5-Year ROI**: 3,885% ($1.54M profit on $40K investment)

---

## 🔐 Security Checklist

- [ ] All secrets stored in secrets managers (not in .env files)
- [ ] API keys rotated quarterly
- [ ] MFA enabled for all admin access
- [ ] SSL/TLS certificates valid (auto-renewed by Cloudflare)
- [ ] Database backups automated (daily)
- [ ] Audit logging enabled (all sensitive actions)
- [ ] OWASP security scan passed (weekly)
- [ ] Penetration testing completed (quarterly)

---

## 📞 Support Contacts

**On-Call Rotation**:
- **Primary**: PagerDuty (phone call + SMS)
- **Escalation**: Slack #production-alerts

**External Support**:
- **Vercel**: https://vercel.com/support
- **Railway**: https://railway.app/help
- **Cloudflare**: https://support.cloudflare.com
- **Google Cloud**: https://cloud.google.com/support

---

## ✅ Post-Deployment Checklist

**Immediate (First 1 Hour)**:
- [ ] All health checks passing (4/4 components)
- [ ] Error rate <0.1% (Sentry dashboard)
- [ ] Latency p95 <500ms (Grafana dashboard)
- [ ] No customer complaints (support tickets, social media)

**First 24 Hours**:
- [ ] Monitor Grafana dashboards continuously
- [ ] Review Sentry errors (fix critical issues)
- [ ] Check cost dashboards (no unexpected spikes)
- [ ] Test all key user workflows (generate content, publish post)

**First Week**:
- [ ] Review metrics after 7 days (deployment impact analysis)
- [ ] Conduct post-deployment retrospective
- [ ] Update documentation (lessons learned)
- [ ] Plan next deployment (features, improvements)

---

## 📚 Documentation Links

| Document | Purpose | Location |
|----------|---------|----------|
| **STAGE-L-DEPLOYMENT-ARCHITECTURE.md** | Complete deployment guide (50+ pages) | `.planning/approved/` |
| **STAGE-L-COST-BREAKDOWN.md** | Detailed cost analysis | `.planning/approved/` |
| **STAGE-L-QUICK-REFERENCE.md** | This document | `.planning/approved/` |
| **CI/CD Pipeline** | GitHub Actions workflows | `.github/workflows/` |
| **Kubernetes Manifests** | K8s deployment configs | `k8s/` |
| **Docker Compose** | Local development setup | `docker-compose.yml` |
| **Terraform** | Infrastructure as Code | `terraform/` |

---

## 🎯 Success Metrics (Production)

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| Uptime | >99.9% | ___% | ⬜ |
| Error Rate | <0.1% | ___% | ⬜ |
| API Latency (p95) | <500ms | ___ms | ⬜ |
| Customer Satisfaction | >4.5/5 | ___/5 | ⬜ |
| Active Customers | 100+ | ___ | ⬜ |
| Posts Published/Day | 2,000+ | ___ | ⬜ |

---

**Last Updated**: November 10, 2025
**Document Version**: 1.0.0
**Author**: Claude Planning Agent

---

**🚀 Ready to Deploy? Start with Section "Deployment Order" above!**
