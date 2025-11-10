# Stage I REST API - cURL Examples

**Complete reference of all 24 REST API endpoints with example cURL commands**

---

## Authentication

All requests require a Bearer token in the Authorization header:

```bash
Authorization: Bearer sk_tenant_xyz_abc123def456
```

**Replace** `sk_tenant_xyz_abc123def456` with your actual API key.

---

## 1. Content Generation Endpoints (6)

### 1.1 Generate Content Candidates

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/content/generate-candidates \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "skill": {
      "template": "product_announcement",
      "parameters": {
        "product": "A193 mesh spacers",
        "platform": "linkedin"
      }
    },
    "count": 3,
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "candidates": [
      {
        "id": "A",
        "text": "Excited to announce our new A193 mesh spacers...",
        "mediaType": "image",
        "mediaUrl": "https://r2.cloudflare.com/product-a.jpg",
        "dimensions": "1200x627",
        "engagementScore": 8.5,
        "brandVoiceScore": 94
      },
      {
        "id": "B",
        "text": "See how A193 mesh spacers transform construction...",
        "mediaType": "video",
        "mediaUrl": "https://r2.cloudflare.com/product-b.mp4",
        "thumbnail": "https://r2.cloudflare.com/product-b-thumb.jpg",
        "duration": "0:15",
        "engagementScore": 7.2,
        "brandVoiceScore": 88
      },
      {
        "id": "C",
        "text": "Technical specifications and installation guide...",
        "mediaType": "carousel",
        "mediaUrls": [
          "https://r2.cloudflare.com/spec-1.jpg",
          "https://r2.cloudflare.com/spec-2.jpg",
          "https://r2.cloudflare.com/install-1.jpg"
        ],
        "engagementScore": 6.8,
        "brandVoiceScore": 92
      }
    ]
  },
  "metadata": {
    "requestId": "req-abc123",
    "timestamp": "2025-11-10T12:00:00Z",
    "processingTimeMs": 2400
  }
}
```

---

### 1.2 Validate Content (Guards Engine)

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/content/validate \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "candidates": [
      {
        "id": "A",
        "text": "Excited to announce our new A193 mesh spacers...",
        "mediaUrl": "https://r2.cloudflare.com/product-a.jpg"
      }
    ],
    "platform": "linkedin",
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "validatedCandidates": [
      {
        "id": "A",
        "isValid": true,
        "issues": [],
        "autoFixes": []
      }
    ],
    "overallStatus": "PASSED"
  },
  "metadata": {
    "requestId": "req-def456",
    "timestamp": "2025-11-10T12:00:05Z",
    "processingTimeMs": 150
  }
}
```

---

### 1.3 Rerank Candidates

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/content/rerank \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "candidates": [
      {"id": "A", "text": "...", "engagementScore": 8.5},
      {"id": "B", "text": "...", "engagementScore": 7.2},
      {"id": "C", "text": "...", "engagementScore": 6.8}
    ],
    "platform": "linkedin",
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "rankedCandidates": [
      {"id": "A", "engagementScore": 8.5, "rank": 1},
      {"id": "B", "engagementScore": 7.2, "rank": 2},
      {"id": "C", "engagementScore": 6.8, "rank": 3}
    ]
  },
  "metadata": {
    "requestId": "req-ghi789",
    "timestamp": "2025-11-10T12:00:10Z",
    "processingTimeMs": 100
  }
}
```

---

### 1.4 Score Brand Voice

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/content/score-brand-voice \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Excited to announce our new A193 mesh spacers for large-scale construction projects!",
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "score": 94,
    "breakdown": {
      "tone": 95,
      "lexicon": 92,
      "style": 96
    },
    "topMatches": [
      {
        "text": "Example brand voice text from guidelines...",
        "similarity": 0.92
      }
    ]
  },
  "metadata": {
    "requestId": "req-jkl012",
    "timestamp": "2025-11-10T12:00:15Z",
    "processingTimeMs": 200
  }
}
```

---

### 1.5 Parse Intent

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/content/parse-intent \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Create a LinkedIn post about our new mesh spacers",
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "intent": "create_post",
    "entities": {
      "platform": "linkedin",
      "product": "mesh spacers",
      "postType": "product_announcement"
    },
    "confidence": 0.95
  },
  "metadata": {
    "requestId": "req-mno345",
    "timestamp": "2025-11-10T12:00:20Z",
    "processingTimeMs": 150
  }
}
```

---

### 1.6 Retrieve Skill

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/content/retrieve-skill \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "intent": "create_post",
    "entities": {
      "platform": "linkedin",
      "postType": "product_announcement"
    },
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "skill": {
      "template": "product_announcement",
      "structure": {
        "hook": "Question or bold statement",
        "body": "Product benefits and features",
        "cta": "Visit website or contact sales"
      },
      "examples": [
        "Example post 1...",
        "Example post 2..."
      ]
    }
  },
  "metadata": {
    "requestId": "req-pqr678",
    "timestamp": "2025-11-10T12:00:25Z",
    "processingTimeMs": 100
  }
}
```

---

## 2. Publishing Endpoints (5)

### 2.1 Publish to LinkedIn

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/publish/linkedin \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "candidateId": "A",
    "text": "Excited to announce our new A193 mesh spacers...",
    "mediaType": "image",
    "mediaUrl": "https://r2.cloudflare.com/product-a.jpg",
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "postId": "linkedin-123456",
    "postUrl": "https://linkedin.com/posts/company-xyz-123456",
    "platform": "linkedin",
    "publishedAt": "2025-11-10T12:00:30Z"
  },
  "metadata": {
    "requestId": "req-stu901",
    "timestamp": "2025-11-10T12:00:30Z",
    "processingTimeMs": 1200
  }
}
```

---

### 2.2 Publish to Twitter/X

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/publish/twitter \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "candidateId": "B",
    "text": "See how A193 mesh spacers transform construction projects!",
    "mediaType": "video",
    "mediaUrl": "https://r2.cloudflare.com/product-b.mp4",
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "postId": "twitter-789012",
    "postUrl": "https://twitter.com/company_xyz/status/789012",
    "platform": "twitter",
    "publishedAt": "2025-11-10T12:00:35Z"
  },
  "metadata": {
    "requestId": "req-vwx234",
    "timestamp": "2025-11-10T12:00:35Z",
    "processingTimeMs": 900
  }
}
```

---

### 2.3 Publish to Facebook

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/publish/facebook \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "candidateId": "C",
    "text": "Technical specifications and installation guide for A193 mesh spacers",
    "mediaType": "carousel",
    "mediaUrls": [
      "https://r2.cloudflare.com/spec-1.jpg",
      "https://r2.cloudflare.com/spec-2.jpg",
      "https://r2.cloudflare.com/install-1.jpg"
    ],
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "postId": "facebook-345678",
    "postUrl": "https://facebook.com/company-xyz/posts/345678",
    "platform": "facebook",
    "publishedAt": "2025-11-10T12:00:40Z"
  },
  "metadata": {
    "requestId": "req-yza567",
    "timestamp": "2025-11-10T12:00:40Z",
    "processingTimeMs": 1500
  }
}
```

---

### 2.4 Publish to Instagram

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/publish/instagram \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "candidateId": "A",
    "text": "New A193 mesh spacers now available! #construction #engineering",
    "mediaType": "image",
    "mediaUrl": "https://r2.cloudflare.com/product-a-1080x1080.jpg",
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "postId": "instagram-901234",
    "postUrl": "https://instagram.com/p/abc123xyz",
    "platform": "instagram",
    "publishedAt": "2025-11-10T12:00:45Z"
  },
  "metadata": {
    "requestId": "req-bcd890",
    "timestamp": "2025-11-10T12:00:45Z",
    "processingTimeMs": 1100
  }
}
```

---

### 2.5 Multi-Platform Publish

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/publish/multi-platform \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "candidateId": "A",
    "text": "Excited to announce our new A193 mesh spacers...",
    "mediaType": "image",
    "mediaUrl": "https://r2.cloudflare.com/product-a.jpg",
    "platforms": ["linkedin", "twitter", "facebook"],
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "results": [
      {
        "platform": "linkedin",
        "success": true,
        "postId": "linkedin-123456",
        "postUrl": "https://linkedin.com/posts/company-xyz-123456"
      },
      {
        "platform": "twitter",
        "success": true,
        "postId": "twitter-789012",
        "postUrl": "https://twitter.com/company_xyz/status/789012"
      },
      {
        "platform": "facebook",
        "success": true,
        "postId": "facebook-345678",
        "postUrl": "https://facebook.com/company-xyz/posts/345678"
      }
    ],
    "successCount": 3,
    "failureCount": 0
  },
  "metadata": {
    "requestId": "req-efg123",
    "timestamp": "2025-11-10T12:00:50Z",
    "processingTimeMs": 3200
  }
}
```

---

## 3. Analytics Endpoints (4)

### 3.1 Get Engagement Metrics

```bash
curl -X GET "https://mcp-gateway.workers.dev/api/v1/analytics/metrics?tenantId=tenant_xyz&dateRange=7d" \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456"
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "metrics": {
      "totalPosts": 45,
      "totalEngagement": 12500,
      "avgEngagementRate": 0.042,
      "topPerformingPlatform": "linkedin",
      "platformBreakdown": {
        "linkedin": {
          "posts": 25,
          "engagement": 8000,
          "engagementRate": 0.05
        },
        "twitter": {
          "posts": 15,
          "engagement": 3500,
          "engagementRate": 0.035
        },
        "facebook": {
          "posts": 5,
          "engagement": 1000,
          "engagementRate": 0.028
        }
      }
    },
    "dateRange": "7d"
  },
  "metadata": {
    "requestId": "req-hij456",
    "timestamp": "2025-11-10T12:00:55Z",
    "processingTimeMs": 300
  }
}
```

---

### 3.2 Get Post Performance

```bash
curl -X GET "https://mcp-gateway.workers.dev/api/v1/analytics/post/linkedin-123456?tenantId=tenant_xyz" \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456"
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "postId": "linkedin-123456",
    "platform": "linkedin",
    "publishedAt": "2025-11-03T12:00:00Z",
    "engagement": {
      "likes": 245,
      "comments": 18,
      "shares": 32,
      "clicks": 450,
      "impressions": 5800,
      "engagementRate": 0.051
    },
    "brandVoiceScore": 94,
    "predictedScore": 8.5,
    "actualScore": 9.1
  },
  "metadata": {
    "requestId": "req-klm789",
    "timestamp": "2025-11-10T12:01:00Z",
    "processingTimeMs": 150
  }
}
```

---

### 3.3 Get Insights

```bash
curl -X GET "https://mcp-gateway.workers.dev/api/v1/analytics/insights?tenantId=tenant_xyz&dateRange=30d" \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456"
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "insights": [
      {
        "type": "best_time",
        "message": "Posts published on Tuesday at 10 AM get 35% more engagement",
        "confidence": 0.89
      },
      {
        "type": "content_type",
        "message": "Video posts outperform images by 2.3x on LinkedIn",
        "confidence": 0.92
      },
      {
        "type": "hashtags",
        "message": "Posts with 3-5 hashtags perform 20% better",
        "confidence": 0.85
      }
    ],
    "recommendations": [
      "Increase video content ratio to 40%",
      "Schedule more posts on weekdays",
      "Use 3-5 relevant hashtags per post"
    ]
  },
  "metadata": {
    "requestId": "req-nop012",
    "timestamp": "2025-11-10T12:01:05Z",
    "processingTimeMs": 400
  }
}
```

---

### 3.4 Track Event

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/analytics/track \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "event": "post_published",
    "properties": {
      "postId": "linkedin-123456",
      "platform": "linkedin",
      "candidateId": "A"
    },
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "eventId": "evt-abc123",
    "tracked": true
  },
  "metadata": {
    "requestId": "req-qrs345",
    "timestamp": "2025-11-10T12:01:10Z",
    "processingTimeMs": 50
  }
}
```

---

## 4. Knowledge Base Endpoints (4)

### 4.1 Query Brand Voice

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/knowledge/brand-voice \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What tone should we use for LinkedIn?",
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "matches": [
      {
        "text": "LinkedIn tone: Professional yet approachable. Use industry terminology but avoid jargon...",
        "score": 0.92,
        "source": "brand-guidelines.pdf"
      },
      {
        "text": "For B2B audiences on LinkedIn, maintain authority while being conversational...",
        "score": 0.88,
        "source": "social-media-playbook.pdf"
      }
    ]
  },
  "metadata": {
    "requestId": "req-tuv678",
    "timestamp": "2025-11-10T12:01:15Z",
    "processingTimeMs": 250
  }
}
```

---

### 4.2 Query Skills Library

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/knowledge/skills \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "product announcement template",
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "matches": [
      {
        "template": "product_announcement",
        "structure": {
          "hook": "Question or bold statement",
          "body": "Product benefits and features",
          "cta": "Visit website or contact sales"
        },
        "score": 0.95,
        "examples": ["Example 1...", "Example 2..."]
      }
    ]
  },
  "metadata": {
    "requestId": "req-wxy901",
    "timestamp": "2025-11-10T12:01:20Z",
    "processingTimeMs": 200
  }
}
```

---

### 4.3 Query Product Catalog

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/knowledge/products \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "A193 mesh spacers",
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "matches": [
      {
        "productId": "prod-123",
        "name": "A193 Mesh Spacers",
        "specs": {
          "material": "High-strength steel",
          "dimensions": "100mm x 100mm",
          "weight": "2.5kg"
        },
        "score": 0.98,
        "price": "£45.00",
        "stock": "in_stock"
      }
    ]
  },
  "metadata": {
    "requestId": "req-zab234",
    "timestamp": "2025-11-10T12:01:25Z",
    "processingTimeMs": 180
  }
}
```

---

### 4.4 Query Example Posts

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/knowledge/examples \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "high-performing product announcement posts",
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "matches": [
      {
        "postId": "post-456",
        "text": "Excited to announce our new product...",
        "platform": "linkedin",
        "engagement": {
          "likes": 350,
          "comments": 25,
          "shares": 45
        },
        "score": 0.91,
        "brandVoiceScore": 96
      }
    ]
  },
  "metadata": {
    "requestId": "req-cde567",
    "timestamp": "2025-11-10T12:01:30Z",
    "processingTimeMs": 220
  }
}
```

---

## 5. Products Endpoints (3)

### 5.1 Get Product Info

```bash
curl -X GET "https://mcp-gateway.workers.dev/api/v1/products/prod-123?tenantId=tenant_xyz" \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456"
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "productId": "prod-123",
    "name": "A193 Mesh Spacers",
    "price": "£45.00",
    "stock": "in_stock",
    "specs": {
      "material": "High-strength steel",
      "dimensions": "100mm x 100mm",
      "weight": "2.5kg"
    },
    "images": [
      "https://shopify.com/image-1.jpg",
      "https://shopify.com/image-2.jpg"
    ]
  },
  "metadata": {
    "requestId": "req-fgh890",
    "timestamp": "2025-11-10T12:01:35Z",
    "processingTimeMs": 100
  }
}
```

---

### 5.2 Get Product Price

```bash
curl -X GET "https://mcp-gateway.workers.dev/api/v1/products/prod-123/price?tenantId=tenant_xyz" \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456"
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "productId": "prod-123",
    "price": "£45.00",
    "currency": "GBP",
    "priceInPence": 4500,
    "discounts": []
  },
  "metadata": {
    "requestId": "req-ijk123",
    "timestamp": "2025-11-10T12:01:40Z",
    "processingTimeMs": 80
  }
}
```

---

### 5.3 Check Stock

```bash
curl -X GET "https://mcp-gateway.workers.dev/api/v1/products/prod-123/stock?tenantId=tenant_xyz" \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456"
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "productId": "prod-123",
    "stock": "in_stock",
    "quantity": 250,
    "availableForPurchase": true
  },
  "metadata": {
    "requestId": "req-lmn456",
    "timestamp": "2025-11-10T12:01:45Z",
    "processingTimeMs": 70
  }
}
```

---

## 6. Messaging Endpoints (2)

### 6.1 Send WhatsApp

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/messaging/whatsapp \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "+447700900123",
    "message": "New post published on LinkedIn! Check it out: https://linkedin.com/posts/...",
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "messageId": "whatsapp-msg-123",
    "status": "sent"
  },
  "metadata": {
    "requestId": "req-opq789",
    "timestamp": "2025-11-10T12:01:50Z",
    "processingTimeMs": 500
  }
}
```

---

### 6.2 Send SMS

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/messaging/sms \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "+447700900123",
    "message": "New post published! LinkedIn: https://linkedin.com/posts/...",
    "tenantId": "tenant_xyz"
  }'
```

**Expected Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "messageId": "sms-msg-456",
    "status": "sent"
  },
  "metadata": {
    "requestId": "req-rst012",
    "timestamp": "2025-11-10T12:01:55Z",
    "processingTimeMs": 450
  }
}
```

---

## Error Examples

### 401 Unauthorized (Invalid Token)

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/content/generate-candidates \
  -H "Authorization: Bearer invalid_token" \
  -H "Content-Type: application/json" \
  -d '{}'
```

**Response (401 Unauthorized):**
```json
{
  "success": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid or expired Bearer token",
    "details": {
      "hint": "Check Authorization header format: Bearer sk_tenant_*_*"
    }
  },
  "metadata": {
    "requestId": "req-uvw345",
    "timestamp": "2025-11-10T12:02:00Z"
  }
}
```

---

### 400 Bad Request (Missing Field)

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/content/generate-candidates \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "count": 3
  }'
```

**Response (400 Bad Request):**
```json
{
  "success": false,
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Missing required field: skill.template",
    "details": {
      "field": "skill.template",
      "expected": "string",
      "received": "undefined"
    }
  },
  "metadata": {
    "requestId": "req-xyz678",
    "timestamp": "2025-11-10T12:02:05Z"
  }
}
```

---

### 429 Rate Limit Exceeded

```bash
# After making 101 requests in one hour...

curl -X POST https://mcp-gateway.workers.dev/api/v1/content/generate-candidates \
  -H "Authorization: Bearer sk_tenant_xyz_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

**Response (429 Too Many Requests):**
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded: 100 requests per hour",
    "details": {
      "limit": 100,
      "windowSeconds": 3600,
      "retryAfterSeconds": 1200
    }
  },
  "metadata": {
    "requestId": "req-abc901",
    "timestamp": "2025-11-10T12:02:10Z"
  }
}
```

**Response Headers:**
```
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1699999999
Retry-After: 1200
```

---

## Testing Tips

### 1. Save Your API Key

```bash
export MCP_API_KEY="sk_tenant_xyz_abc123def456"
```

Then use in cURL:
```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/content/generate-candidates \
  -H "Authorization: Bearer $MCP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

---

### 2. Pretty-Print JSON Responses

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/content/generate-candidates \
  -H "Authorization: Bearer $MCP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{...}' | jq '.'
```

---

### 3. Save Response to File

```bash
curl -X POST https://mcp-gateway.workers.dev/api/v1/content/generate-candidates \
  -H "Authorization: Bearer $MCP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{...}' \
  -o response.json
```

---

### 4. Check Response Headers

```bash
curl -i -X POST https://mcp-gateway.workers.dev/api/v1/content/generate-candidates \
  -H "Authorization: Bearer $MCP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

---

### 5. Verbose Output (Debug)

```bash
curl -v -X POST https://mcp-gateway.workers.dev/api/v1/content/generate-candidates \
  -H "Authorization: Bearer $MCP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

---

## Quick Reference

| Category | Endpoints | Methods |
|----------|-----------|---------|
| **Content Generation** | 6 | POST |
| **Publishing** | 5 | POST |
| **Analytics** | 4 | GET, POST |
| **Knowledge Base** | 4 | POST |
| **Products** | 3 | GET |
| **Messaging** | 2 | POST |
| **Total** | **24** | |

---

**All endpoints require Bearer token authentication.**
**Rate limit: 100 requests/hour (Free tier), 1,000 requests/hour (Professional).**
**Base URL: `https://mcp-gateway.workers.dev/api/v1`**
