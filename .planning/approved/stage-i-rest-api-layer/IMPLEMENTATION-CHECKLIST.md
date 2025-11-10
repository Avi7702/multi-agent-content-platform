# Stage I REST API Layer - Implementation Checklist

**5-day implementation roadmap with detailed tasks**

---

## Overview

**Goal**: Add REST API layer to MCP Gateway for Dify HTTP Tools integration
**Timeline**: 5 days (1 week)
**Developer**: 1 full-stack engineer
**Complexity**: Medium (building on existing JSON-RPC infrastructure)

---

## Day 1: Foundation & Routing

### Morning (4 hours)

- [ ] **Create directory structure**
  ```bash
  cd mcp-gateway-final/src
  mkdir rest
  mkdir rest/handlers
  ```

- [ ] **Implement REST router** (`src/rest/router.ts`)
  - [ ] Parse incoming HTTP requests
  - [ ] Route `/api/v1/*` paths
  - [ ] Extract Bearer token from headers
  - [ ] Handle GET vs POST methods
  - [ ] Return 404 for unknown routes

- [ ] **Update main entry point** (`src/index.ts`)
  - [ ] Add conditional routing (REST vs JSON-RPC)
  - [ ] Preserve existing JSON-RPC handler
  - [ ] Add error handling for routing failures

### Afternoon (4 hours)

- [ ] **Implement authentication** (`src/rest/auth.ts`)
  - [ ] `validateBearerToken()` function
  - [ ] Extract tenant ID from token
  - [ ] Validate token format: `sk_tenant_*_*`
  - [ ] Check token exists in `USER_CACHE` KV namespace
  - [ ] Return `{valid: boolean, tenantId?: string}`

- [ ] **Implement rate limiter** (`src/rest/rate-limiter.ts`)
  - [ ] `checkRateLimit(tenantId, env)` function
  - [ ] Use `RATE_LIMIT` KV namespace
  - [ ] Implement hourly bucket (100 requests/hour)
  - [ ] Return `{allowed, remaining, resetAt}`
  - [ ] Handle KV expiration (3600s TTL)

### Evening (Testing - 1 hour)

- [ ] **Write unit tests**
  - [ ] Test Bearer token validation (valid, invalid, malformed)
  - [ ] Test rate limiting (under limit, at limit, over limit)
  - [ ] Test routing (known paths, unknown paths, wrong method)

- [ ] **Manual testing**
  ```bash
  wrangler dev
  curl -X POST http://localhost:8787/api/v1/test \
    -H "Authorization: Bearer sk_tenant_test_abc123"
  ```

---

## Day 2: Request/Response Translation

### Morning (4 hours)

- [ ] **Implement translator** (`src/rest/translator.ts`)
  - [ ] `translateRestToJsonRpc(path, method, body, tenantId, requestId)`
    - [ ] Extract JSON-RPC method from REST path
    - [ ] Move request body to `params`
    - [ ] Add `tenantId` to `params`
    - [ ] Generate unique request ID
    - [ ] Return JSON-RPC request object

  - [ ] **Path-to-method mapping**:
    - `/content/generate-candidates` → `generate_candidates`
    - `/publish/linkedin` → `publish_linkedin`
    - `/analytics/metrics` → `get_analytics_metrics`
    - `/products/{id}/price` → `get_product_price`
    - Add all 24 mappings

### Afternoon (4 hours)

- [ ] **Implement response translator**
  - [ ] `translateJsonRpcToRest(jsonRpcResponse, requestId, processingTimeMs)`
    - [ ] Check if `error` or `result`
    - [ ] Map JSON-RPC errors to HTTP status codes
    - [ ] Format success response: `{success, data, metadata}`
    - [ ] Format error response: `{success, error, metadata}`
    - [ ] Add timestamp and request ID

- [ ] **Error code mapping**
  - [ ] `-32600` → `400 INVALID_JSON`
  - [ ] `-32601` → `404 METHOD_NOT_FOUND`
  - [ ] `-32602` → `400 INVALID_REQUEST`
  - [ ] `-32603` → `500 INTERNAL_ERROR`

### Evening (Testing - 1 hour)

- [ ] **Write unit tests**
  - [ ] Test REST → JSON-RPC translation (all 24 paths)
  - [ ] Test JSON-RPC → REST translation (success cases)
  - [ ] Test JSON-RPC → REST translation (error cases)
  - [ ] Test error code mapping

---

## Day 3: Content & Publishing Handlers

### Morning (4 hours)

- [ ] **Implement content handlers** (`src/rest/handlers/content.ts`)
  - [ ] `handleGenerateCandidates(request, env)`
  - [ ] `handleValidateContent(request, env)`
  - [ ] `handleRerankCandidates(request, env)`
  - [ ] `handleScoreBrandVoice(request, env)`
  - [ ] `handleParseIntent(request, env)`
  - [ ] `handleRetrieveSkill(request, env)`

- [ ] **Request validation**
  - [ ] Validate required fields (skill, count, tenantId)
  - [ ] Validate field types (string, number, object)
  - [ ] Return 400 with clear error message if invalid

### Afternoon (4 hours)

- [ ] **Implement publishing handlers** (`src/rest/handlers/publish.ts`)
  - [ ] `handlePublishLinkedIn(request, env)`
  - [ ] `handlePublishTwitter(request, env)`
  - [ ] `handlePublishFacebook(request, env)`
  - [ ] `handlePublishInstagram(request, env)`
  - [ ] `handleMultiPlatformPublish(request, env)`

- [ ] **Integrate handlers into router**
  - [ ] Add switch/case for `/content/*` paths
  - [ ] Add switch/case for `/publish/*` paths
  - [ ] Call appropriate handler function
  - [ ] Pass request and env to handlers

### Evening (Testing - 1 hour)

- [ ] **Integration tests**
  - [ ] Test content generation endpoint (end-to-end)
  - [ ] Test publishing endpoint (mock JSON-RPC response)
  - [ ] Test error handling (missing fields, invalid data)

---

## Day 4: Analytics, Knowledge, Products, Messaging Handlers

### Morning (4 hours)

- [ ] **Implement analytics handlers** (`src/rest/handlers/analytics.ts`)
  - [ ] `handleGetMetrics(request, env)` (GET with query params)
  - [ ] `handleGetPostPerformance(request, env)` (GET with path param)
  - [ ] `handleGetInsights(request, env)` (GET with query params)
  - [ ] `handleTrackEvent(request, env)` (POST)

- [ ] **Implement knowledge handlers** (`src/rest/handlers/knowledge.ts`)
  - [ ] `handleQueryBrandVoice(request, env)`
  - [ ] `handleQuerySkills(request, env)`
  - [ ] `handleQueryProducts(request, env)`
  - [ ] `handleQueryExamples(request, env)`

### Afternoon (3 hours)

- [ ] **Implement products handlers** (`src/rest/handlers/products.ts`)
  - [ ] `handleGetProductInfo(request, env)` (GET with path param)
  - [ ] `handleGetProductPrice(request, env)` (GET with path param)
  - [ ] `handleCheckStock(request, env)` (GET with path param)

- [ ] **Implement messaging handlers** (`src/rest/handlers/messaging.ts`)
  - [ ] `handleSendWhatsApp(request, env)`
  - [ ] `handleSendSMS(request, env)`

### Afternoon (continued - 1 hour)

- [ ] **Integrate all handlers into router**
  - [ ] Add routes for `/analytics/*`
  - [ ] Add routes for `/knowledge/*`
  - [ ] Add routes for `/products/*`
  - [ ] Add routes for `/messaging/*`

### Evening (Testing - 2 hours)

- [ ] **Integration tests for all handlers**
  - [ ] Test analytics endpoints (GET with query params)
  - [ ] Test knowledge endpoints (POST with query)
  - [ ] Test products endpoints (GET with path param)
  - [ ] Test messaging endpoints (POST)

- [ ] **End-to-end tests**
  - [ ] Full workflow: generate → validate → publish
  - [ ] Error handling: rate limit, invalid token, missing fields

---

## Day 5: Testing, Documentation, Deployment

### Morning (3 hours)

- [ ] **Comprehensive testing**
  - [ ] Run all unit tests: `npm test`
  - [ ] Run all integration tests
  - [ ] Test rate limiting (exceed 100 requests)
  - [ ] Test authentication (invalid tokens)
  - [ ] Test all 24 endpoints with cURL

- [ ] **Performance testing**
  - [ ] Measure response time (target: <300ms for simple endpoints)
  - [ ] Test with concurrent requests (10 simultaneous)
  - [ ] Verify KV operations don't slow down requests

### Afternoon (3 hours)

- [ ] **Generate OpenAPI spec** (optional)
  - [ ] Use OpenAPI 3.0 format
  - [ ] Document all 24 endpoints
  - [ ] Add request/response schemas
  - [ ] Add authentication section
  - [ ] Validate with `swagger-cli validate openapi.yaml`

- [ ] **Create Dify HTTP Tool examples**
  - [ ] Write example configurations for 5 common endpoints
  - [ ] Test in Dify (manually configure HTTP Tool node)
  - [ ] Document variable interpolation (`{{workflow.variables.*}}`)
  - [ ] Document secrets management (`{{secret.MCP_API_KEY}}`)

### Evening (2 hours)

- [ ] **Deployment**
  - [ ] Run final tests locally: `wrangler dev`
  - [ ] Deploy to production: `wrangler deploy`
  - [ ] Verify deployment: `curl https://mcp-gateway.workers.dev/api/v1/content/generate-candidates`
  - [ ] Check logs: `wrangler tail`

- [ ] **Create API keys for tenants**
  ```bash
  # Generate API key
  wrangler kv:key put --namespace-id=3c3eb8a6a699479e976cb424456d086f \
    "api_keys:sk_tenant_test_abc123def456" \
    '{"tenantId":"tenant_test","createdAt":"2025-11-10T12:00:00Z"}' \
    --ttl 31536000
  ```

- [ ] **Final verification**
  - [ ] Test all 24 endpoints from production URL
  - [ ] Verify rate limiting works in production
  - [ ] Verify authentication works in production
  - [ ] Check CloudWatch/Datadog for errors

---

## Definition of Done

### Code Quality
- [ ] All TypeScript strict mode enabled
- [ ] No `any` types (use proper types)
- [ ] ESLint passes with 0 warnings
- [ ] Prettier formatting applied

### Testing
- [ ] Unit tests: 80%+ coverage
- [ ] Integration tests: 24 endpoints tested
- [ ] End-to-end test: Full workflow passes
- [ ] Rate limiting verified
- [ ] Authentication verified

### Documentation
- [ ] README updated with REST API section
- [ ] OpenAPI spec generated (optional but recommended)
- [ ] cURL examples documented for all 24 endpoints
- [ ] Dify HTTP Tool examples provided

### Deployment
- [ ] Deployed to production
- [ ] API keys created for test tenant
- [ ] Logs verified (no errors)
- [ ] Performance metrics checked (<300ms avg)

### Dify Integration
- [ ] HTTP Tool configured in Dify workflow
- [ ] Test workflow executes successfully
- [ ] Response parsed correctly in Dify
- [ ] Error handling works (e.g., rate limit, invalid token)

---

## Post-Implementation Tasks

### Week 2 (Monitoring)
- [ ] Monitor error rates (target: <1% 5xx errors)
- [ ] Monitor response times (target: <300ms p95)
- [ ] Monitor rate limit hits (adjust limits if needed)
- [ ] Collect feedback from Dify workflow users

### Month 1 (Optimization)
- [ ] Optimize slow endpoints (>500ms)
- [ ] Add caching for frequently accessed data
- [ ] Implement request retries for transient failures
- [ ] Add more granular rate limiting (per endpoint)

### Future Enhancements
- [ ] Add API versioning (`/api/v2/...`)
- [ ] Add webhook support (for async operations)
- [ ] Add GraphQL layer (alternative to REST)
- [ ] Add OpenAPI-based client SDK generation

---

## Dependencies

### External Services
- [ ] Cloudflare Workers (deployed)
- [ ] Workers KV (USER_CACHE, RATE_LIMIT namespaces)
- [ ] Existing JSON-RPC handler (Stage C/D/E/F)

### Required Credentials
- [ ] Cloudflare API token (for deployment)
- [ ] Dify API key (for testing HTTP Tools)
- [ ] Test tenant API key (for cURL testing)

### Development Tools
- [ ] Node.js 18+
- [ ] wrangler CLI (`npm install -g wrangler`)
- [ ] curl (for testing)
- [ ] jq (for JSON parsing, optional)
- [ ] Postman or Insomnia (for manual testing, optional)

---

## Troubleshooting

### Issue: 401 Unauthorized
**Solution**: Check Bearer token format (`sk_tenant_*_*`) and verify token exists in `USER_CACHE`.

### Issue: 429 Rate Limit Exceeded
**Solution**: Wait for rate limit window to reset (1 hour) or increase limit in code.

### Issue: 500 Internal Error
**Solution**: Check CloudWatch logs for JSON-RPC handler errors. Verify downstream services are reachable.

### Issue: Slow Response Times (>1s)
**Solution**: Profile code with `wrangler tail`. Check if KV operations are slow. Consider caching.

### Issue: Dify Workflow Fails
**Solution**: Check Dify HTTP Tool configuration (URL, headers, body). Verify API key is set in Dify secrets. Check response format matches expected structure.

---

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Deployment Success** | 100% | All 24 endpoints deployed |
| **Test Coverage** | 80%+ | Unit + integration tests |
| **Response Time (p95)** | <300ms | CloudWatch metrics |
| **Error Rate** | <1% | CloudWatch metrics |
| **Rate Limit Accuracy** | 100% | Test with 101 requests |
| **Dify Integration** | 100% | Test workflow passes |

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| **Breaking existing JSON-RPC clients** | Keep JSON-RPC handler unchanged; route only `/api/v1/*` to REST layer |
| **Performance degradation** | Profile code; optimize KV operations; add caching |
| **Rate limiting too restrictive** | Start with 100/hour; monitor usage; adjust based on feedback |
| **Authentication complexity** | Simple Bearer token validation; no OAuth complexity in v1 |
| **Dify configuration errors** | Provide clear examples; test in Dify before documenting |

---

**Stage I Implementation Ready to Start!**

**Next Steps:**
1. Assign full-stack engineer
2. Create branch: `feature/stage-i-rest-api-layer`
3. Follow Day 1 checklist
4. Daily standups to track progress
5. Deploy by end of Day 5
