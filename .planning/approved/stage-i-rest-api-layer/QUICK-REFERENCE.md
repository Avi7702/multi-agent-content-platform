# Stage I REST API - Quick Reference Card

**One-page summary of all 24 REST endpoints**

---

## Base URL
```
https://mcp-gateway.workers.dev/api/v1
```

## Authentication
```http
Authorization: Bearer sk_tenant_{TENANT_ID}_{SECRET_KEY}
```

## Rate Limits
- **Free**: 100 requests/hour, 1,000/day
- **Professional**: 1,000 requests/hour, 10,000/day

---

## Content Generation (6 endpoints)

### 1. Generate Candidates
```http
POST /content/generate-candidates
```
**Body**: `{skill, count, tenantId}`
**Returns**: 3 content options (A, B, C) with media

### 2. Validate Content
```http
POST /content/validate
```
**Body**: `{candidates, platform, tenantId}`
**Returns**: Validation results with issues/fixes

### 3. Rerank Candidates
```http
POST /content/rerank
```
**Body**: `{candidates, platform, tenantId}`
**Returns**: Sorted by engagement score

### 4. Score Brand Voice
```http
POST /content/score-brand-voice
```
**Body**: `{text, tenantId}`
**Returns**: Score 0-100 with breakdown

### 5. Parse Intent
```http
POST /content/parse-intent
```
**Body**: `{text, tenantId}`
**Returns**: Intent + entities

### 6. Retrieve Skill
```http
POST /content/retrieve-skill
```
**Body**: `{intent, entities, tenantId}`
**Returns**: Skill template + structure

---

## Publishing (5 endpoints)

### 7. Publish to LinkedIn
```http
POST /publish/linkedin
```
**Body**: `{candidateId, text, mediaType, mediaUrl, tenantId}`
**Returns**: `{postId, postUrl, publishedAt}`

### 8. Publish to Twitter
```http
POST /publish/twitter
```
**Body**: Same as LinkedIn
**Returns**: `{postId, postUrl, publishedAt}`

### 9. Publish to Facebook
```http
POST /publish/facebook
```
**Body**: Same as LinkedIn
**Returns**: `{postId, postUrl, publishedAt}`

### 10. Publish to Instagram
```http
POST /publish/instagram
```
**Body**: Same as LinkedIn
**Returns**: `{postId, postUrl, publishedAt}`

### 11. Multi-Platform Publish
```http
POST /publish/multi-platform
```
**Body**: `{candidateId, text, mediaType, mediaUrl, platforms: [], tenantId}`
**Returns**: `{results: [{platform, success, postId}], successCount, failureCount}`

---

## Analytics (4 endpoints)

### 12. Get Metrics
```http
GET /analytics/metrics?tenantId={id}&dateRange={7d|30d|90d}
```
**Returns**: Total posts, engagement, platform breakdown

### 13. Get Post Performance
```http
GET /analytics/post/{postId}?tenantId={id}
```
**Returns**: Likes, comments, shares, engagement rate

### 14. Get Insights
```http
GET /analytics/insights?tenantId={id}&dateRange={7d|30d}
```
**Returns**: AI-generated insights + recommendations

### 15. Track Event
```http
POST /analytics/track
```
**Body**: `{event, properties, tenantId}`
**Returns**: `{eventId, tracked: true}`

---

## Knowledge Base (4 endpoints)

### 16. Query Brand Voice
```http
POST /knowledge/brand-voice
```
**Body**: `{query, tenantId}`
**Returns**: Top matches from brand guidelines

### 17. Query Skills Library
```http
POST /knowledge/skills
```
**Body**: `{query, tenantId}`
**Returns**: Matching skill templates

### 18. Query Product Catalog
```http
POST /knowledge/products
```
**Body**: `{query, tenantId}`
**Returns**: Matching products with specs

### 19. Query Example Posts
```http
POST /knowledge/examples
```
**Body**: `{query, tenantId}`
**Returns**: High-performing past posts

---

## Products (3 endpoints)

### 20. Get Product Info
```http
GET /products/{productId}?tenantId={id}
```
**Returns**: Name, price, stock, specs, images

### 21. Get Product Price
```http
GET /products/{productId}/price?tenantId={id}
```
**Returns**: Price, currency, discounts

### 22. Check Stock
```http
GET /products/{productId}/stock?tenantId={id}
```
**Returns**: Stock status, quantity

---

## Messaging (2 endpoints)

### 23. Send WhatsApp
```http
POST /messaging/whatsapp
```
**Body**: `{to, message, tenantId}`
**Returns**: `{messageId, status: "sent"}`

### 24. Send SMS
```http
POST /messaging/sms
```
**Body**: `{to, message, tenantId}`
**Returns**: `{messageId, status: "sent"}`

---

## Standard Response Format

### Success (200 OK)
```json
{
  "success": true,
  "data": { ... },
  "metadata": {
    "requestId": "req-abc123",
    "timestamp": "2025-11-10T12:00:00Z",
    "processingTimeMs": 250
  }
}
```

### Error (400/401/429/500)
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": { ... }
  },
  "metadata": {
    "requestId": "req-def456",
    "timestamp": "2025-11-10T12:00:00Z"
  }
}
```

---

## HTTP Status Codes

| Code | Meaning | When |
|------|---------|------|
| 200 | OK | Success |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Invalid/missing token |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Error | Server failure |

---

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INVALID_REQUEST` | 400 | Missing/invalid fields |
| `INVALID_JSON` | 400 | Malformed JSON |
| `UNAUTHORIZED` | 401 | Invalid token |
| `TOKEN_EXPIRED` | 401 | Expired API key |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Server failure |
| `SERVICE_UNAVAILABLE` | 500 | Downstream service down |
| `TIMEOUT` | 500 | Request timeout |

---

## Rate Limit Headers

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1699999999
```

When rate limit exceeded:
```http
HTTP/1.1 429 Too Many Requests
Retry-After: 1200
```

---

## Example cURL Commands

### Generate Content
```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/content/generate-candidates \
  -H "Authorization: Bearer sk_tenant_xyz_abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "skill": {
      "template": "product_announcement",
      "parameters": {"product": "A193 mesh spacers"}
    },
    "count": 3,
    "tenantId": "tenant_xyz"
  }'
```

### Publish to LinkedIn
```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/publish/linkedin \
  -H "Authorization: Bearer sk_tenant_xyz_abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "candidateId": "A",
    "text": "Excited to announce...",
    "mediaType": "image",
    "mediaUrl": "https://r2.cloudflare.com/product-a.jpg",
    "tenantId": "tenant_xyz"
  }'
```

### Get Analytics
```bash
curl -X GET "https://mcp-gateway.workers.dev/api/v1/analytics/metrics?tenantId=tenant_xyz&dateRange=7d" \
  -H "Authorization: Bearer sk_tenant_xyz_abc123"
```

---

## Dify HTTP Tool Configuration

```yaml
type: http-request
method: POST
url: https://mcp-gateway.workers.dev/api/v1/content/generate-candidates
headers:
  Authorization: Bearer {{secret.MCP_API_KEY}}
  Content-Type: application/json
body:
  skill:
    template: "{{workflow.variables.skill_template}}"
    parameters:
      product: "{{workflow.variables.product_name}}"
  count: 3
  tenantId: "{{workflow.variables.tenant_id}}"
timeout: 30
output_variable: candidates_response
```

---

## Testing Tips

### Save API Key
```bash
export MCP_API_KEY="sk_tenant_xyz_abc123"
```

### Pretty-Print JSON
```bash
curl ... | jq '.'
```

### Save Response
```bash
curl ... -o response.json
```

### Debug with Verbose
```bash
curl -v ...
```

---

## Implementation Timeline

| Day | Tasks | Deliverables |
|-----|-------|--------------|
| **Day 1** | Router, auth, rate limiter | Foundation complete |
| **Day 2** | REST ↔ JSON-RPC translator | Translation working |
| **Day 3** | Content + publishing handlers | 11 endpoints done |
| **Day 4** | Analytics + knowledge + products + messaging | 24 endpoints done |
| **Day 5** | Testing, docs, deployment | Production ready |

---

## Success Criteria

✅ All 24 endpoints deployed
✅ 80%+ test coverage
✅ <300ms p95 response time
✅ <1% error rate
✅ Rate limiting accurate
✅ Dify integration working

---

## Architecture Summary

```
User → Dify Workflow (HTTP Tool)
     → REST API Layer (Auth, Rate Limit, Translate)
     → JSON-RPC Handler (Existing MCP Gateway)
     → 16 Microservices (Stage D + E + F)
     → External Services (OpenAI, Google Vision, Shopify, etc.)
```

---

## Next Steps

1. **Read**: STAGE-I-REST-API-SPECIFICATION.md (full details)
2. **Test**: CURL-EXAMPLES.md (all 24 endpoints)
3. **Implement**: IMPLEMENTATION-CHECKLIST.md (5-day plan)
4. **Deploy**: `wrangler deploy`
5. **Verify**: Test in Dify workflow

---

**Print this card and keep it at your desk for quick reference!**
