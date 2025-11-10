# Stage G: Dify Implementation with Custom Media-Rich Frontend - APPROVED PLAN

**Version**: 1.0.0
**Status**: ✅ APPROVED
**Date**: November 10, 2025
**Stage**: G of 12 (Dify Implementation & User Interface)
**Dependencies**: Stage C (Knowledge Base), Stage D (Content Generation), Stage E (Multi-Platform Publishing), Stage F (Analytics)

---

## Executive Summary

### Purpose
Implement the user-facing interface layer using a **custom React frontend** with full video and media support, integrated with **Dify backend** for workflow orchestration. This hybrid approach provides production-grade media display (video, images, GIFs, carousels) while leveraging Dify's powerful workflow engine.

### Critical Requirement: Full Media Support
Users MUST see high-quality previews of ALL media types before choosing which content to publish:
- **Videos**: 1080p+ playback with controls (LinkedIn Reels, Instagram Reels, Twitter videos)
- **Images**: Full resolution (1200x627px LinkedIn, 1080x1080px Instagram, etc.)
- **GIFs**: Smooth playback without compression
- **Carousels**: Multi-image galleries (up to 9 images for LinkedIn)
- **Mixed content**: Video + images in same preview

**Why Custom Frontend?**
- Dify limits image display to 250×250px (insufficient for social media preview)
- Dify has no native video playback
- Dify has no carousel/gallery support
- Custom frontend gives full control over media rendering

### Key Deliverables
1. **Custom React Frontend** - Production-grade chat UI with media gallery
2. **Dify Backend Workflows** - 5 orchestration workflows
3. **Media Components** - Video player, image gallery, carousel, GIF player
4. **Dify API Integration** - Seamless frontend-backend communication
5. **Mobile-First Design** - Touch-optimized, responsive, adaptive streaming
6. **4 Knowledge Bases** - Brand voice, skills, products, examples
7. **Conversational Advisory Pattern** - Text-based A/B/C options with media preview

### Timeline
**23 days** (4.6 weeks)

### Budget
- **Development**: $86,250 (230 engineering hours @ $375/hour)
- **Monthly Operational**: $64/month ($0.64/customer/month for 100 customers)
  - Dify Cloud Professional: $59/month
  - Cloudflare Pages: $0-5/month
  - Media already budgeted in Stage E (R2, Google Vision)

### Success Criteria
- ✅ User can see full-resolution images (1200x627px) in chat
- ✅ User can play videos (1080p+) with controls in chat
- ✅ User can browse image carousels (swipe on mobile, arrow keys on desktop)
- ✅ GIF playback smooth without compression
- ✅ Mobile video auto-plays muted on scroll, fullscreen on tap
- ✅ Conversational advisory pattern (A/B/C text options) works seamlessly
- ✅ All 5 Dify workflows orchestrate backend services correctly
- ✅ 4 knowledge bases integrated and queryable
- ✅ End-to-end latency <3 seconds (user message → Dify → MCP Gateway → response)
- ✅ Works on iOS Safari, Android Chrome, desktop browsers

---

## Table of Contents

1. [System Architecture](#1-system-architecture)
2. [Custom Frontend Stack](#2-custom-frontend-stack)
3. [Media Components Specification](#3-media-components-specification)
4. [Platform-Specific Media Requirements](#4-platform-specific-media-requirements)
5. [Dify Backend Integration](#5-dify-backend-integration)
6. [Conversational Advisory Pattern](#6-conversational-advisory-pattern)
7. [Mobile Optimization](#7-mobile-optimization)
8. [API Specifications](#8-api-specifications)
9. [Database Schema](#9-database-schema)
10. [Cost Analysis](#10-cost-analysis)
11. [Implementation Timeline](#11-implementation-timeline)
12. [Testing Strategy](#12-testing-strategy)
13. [Deployment Plan](#13-deployment-plan)

---

## 1. System Architecture

### 1.1 High-Level Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                   USER INTERFACE LAYER                          │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  Next.js 15 Frontend (Cloudflare Pages)                 │ │
│  │  Enterprise-grade: Supabase/ElevenLabs/Netflix quality  │ │
│  │                                                           │ │
│  │  Components:                                             │ │
│  │  - Chat Interface (multi-turn conversation)             │ │
│  │  - Video Player (react-player, 1080p+, HLS)             │ │
│  │  - Image Gallery (react-image-gallery, full-res)        │ │
│  │  - Carousel (swipe/arrow navigation, touch-optimized)   │ │
│  │  - GIF Player (smooth playback, lazy loading)           │ │
│  │  - Option Selector (A/B/C with media preview)           │ │
│  │  - Mobile Touch Controls (PWA-ready)                    │ │
│  │                                                           │ │
│  │  Tech Stack:                                             │ │
│  │  - Next.js 15 (App Router) + TypeScript (strict)        │ │
│  │  - Tailwind CSS 4 + Radix UI + shadcn/ui               │ │
│  │  - TanStack Query (server state) + Zustand (client)     │ │
│  │  - Vitest + Playwright (testing, 80%+ coverage)         │ │
│  │  - Sentry (errors) + Cloudflare Web Analytics           │ │
│  │  - Lighthouse 90+ score, <200KB bundle, <1.8s LCP       │ │
│  │                                                           │ │
│  └───────────────────────┬──────────────────────────────────┘ │
│                          │ HTTPS API Calls                    │
└──────────────────────────┼────────────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────────────┐
│                 ORCHESTRATION LAYER                             │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │         Dify Backend (Dify Cloud Professional)           │ │
│  │                                                           │ │
│  │  Workflows (5):                                          │ │
│  │  1. Main Chatflow (customer conversation)               │ │
│  │  2. Skill Matcher (retrieves from KB, calls OpenManus)  │ │
│  │  3. Content Generator (calls 11 Stage D microservices)  │ │
│  │  4. Publishing Workflow (calls 4 Stage E adapters)      │ │
│  │  5. Learning Workflow (feedback, analytics)             │ │
│  │                                                           │ │
│  │  Knowledge Bases (4):                                    │ │
│  │  - Brand Voice KB (tone, lexicon, examples)             │ │
│  │  - Skill Library KB (post templates, layouts)           │ │
│  │  - Product Catalog KB (inventory, specs)                │ │
│  │  - Example Posts KB (high-performing past posts)        │ │
│  │                                                           │ │
│  │  Tools Integration:                                      │ │
│  │  - HTTP Tool (calls MCP Gateway endpoints)              │ │
│  │  - Knowledge Retrieval (queries 4 KBs)                  │ │
│  │  - Variable Management (conversation state)             │ │
│  │  - Conditional Logic (IF/ELSE nodes)                    │ │
│  │                                                           │ │
│  └───────────────────────┬──────────────────────────────────┘ │
│                          │ HTTP Requests                      │
└──────────────────────────┼────────────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────────────┐
│                    SERVICES LAYER                               │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │      MCP Gateway (Cloudflare Workers)                    │ │
│  │                                                           │ │
│  │  Stage D Services (11):                                  │ │
│  │  - Intent Parser, Skill Retrieval, Multi-Candidate      │ │
│  │    Generator, Guards Engine, Reranker, Defect           │ │
│  │    Classifier, Learning Engine, Brand Voice,             │ │
│  │    Publishing, Analytics, Context                        │ │
│  │                                                           │ │
│  │  Stage E Services (4):                                   │ │
│  │  - LinkedIn Adapter, Twitter/X Adapter, Facebook         │ │
│  │    Adapter, Instagram Adapter                            │ │
│  │                                                           │ │
│  │  Stage F Services (1):                                   │ │
│  │  - Analytics Collection & Performance Metrics            │ │
│  │                                                           │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Data Flow

**Example: User Requests LinkedIn Post**

```
1. User types: "Create a LinkedIn post about our new [product-name]"
   ↓
2. Frontend sends message to Dify API
   ↓
3. Dify Workflow 1 (Main Chatflow):
   - Parses intent
   - Queries Product Catalog KB ("[product-name]")
   - Calls Workflow 2 (Skill Matcher)
   ↓
4. Dify Workflow 2 (Skill Matcher):
   - Queries Skill Library KB
   - Calls OpenManus (Stage H) for skill recommendation
   - Returns skill template
   ↓
5. Dify Workflow 3 (Content Generator):
   - Calls MCP Gateway → Stage D Intent Parser
   - Calls MCP Gateway → Stage D Multi-Candidate Generator (generates 3 options)
   - Each option includes:
     * Text content
     * Image URL (from Cloudflare R2)
     * Video URL (if applicable)
     * Predicted engagement score
   - Calls MCP Gateway → Stage D Guards Engine (validates all 3)
   - Returns 3 options to Dify
   ↓
6. Dify returns response to Frontend:
   {
     "type": "options",
     "options": [
       {
         "id": "A",
         "text": "Professional product shot...",
         "mediaType": "image",
         "mediaUrl": "https://r2.cloudflare.com/post-a-image.jpg",
         "dimensions": "1200x627",
         "engagementScore": 8.5
       },
       {
         "id": "B",
         "text": "Lifestyle context...",
         "mediaType": "video",
         "mediaUrl": "https://r2.cloudflare.com/post-b-video.mp4",
         "thumbnail": "https://r2.cloudflare.com/post-b-thumb.jpg",
         "duration": "0:15",
         "engagementScore": 7.2
       },
       {
         "id": "C",
         "text": "Technical diagram...",
         "mediaType": "carousel",
         "mediaUrls": [
           "https://r2.cloudflare.com/post-c-1.jpg",
           "https://r2.cloudflare.com/post-c-2.jpg",
           "https://r2.cloudflare.com/post-c-3.jpg"
         ],
         "engagementScore": 6.8
       }
     ],
     "prompt": "Which option would you like to publish? (Type A, B, or C)"
   }
   ↓
7. Frontend renders:
   - Option A: Full 1200x627 image displayed
   - Option B: Video player with thumbnail, play button
   - Option C: Carousel (3 images, swipeable)
   - User sees all options with high-quality media
   ↓
8. User types: "B"
   ↓
9. Frontend sends choice to Dify API
   ↓
10. Dify Workflow 4 (Publishing Workflow):
    - Calls MCP Gateway → Stage E LinkedIn Adapter
    - Publishes post B to LinkedIn
    - Returns success confirmation
    ↓
11. Frontend displays: "✅ Published to LinkedIn! [Preview link]"
```

### 1.3 Why This Architecture?

**Separation of Concerns:**
- **Frontend**: User experience, media rendering (React's strength)
- **Backend**: Workflow orchestration, business logic (Dify's strength)
- **Services**: Data processing, integrations (Cloudflare Workers' strength)

**Best of Each Platform:**
- Dify excels at: Workflows, conditional logic, knowledge bases, LLM integration
- React excels at: Rich UI, media rendering, mobile responsiveness, animations
- Cloudflare Workers excels at: Low latency, global distribution, cost efficiency

**No Compromises:**
- Full-resolution media (not 250px thumbnails)
- Video playback with controls
- Mobile-first UX
- Fast, scalable, production-ready

---

## 2. Custom Frontend Stack

### 2.1 Enterprise-Grade Tech Stack
**Inspired by**: Supabase, ElevenLabs, Netflix production systems

**Framework:** Next.js 15+ with App Router + TypeScript 5.3+
- **Why Next.js:** Production-grade SSR/SSG, file-based routing, API routes, image optimization, automatic code splitting
- **Why TypeScript:** Type safety (strict mode), better DX, catches bugs at compile time
- **Alternative to:** React + Vite (Next.js provides better production features)
- **Used by:** Supabase (open-source), ElevenLabs ($200M ARR scale)

**Package Manager:** pnpm 8+
- **Why pnpm:** 3x faster than npm, disk space efficient, strict dependency resolution
- **Used by:** Supabase (verified in monorepo setup)

**Styling:** Tailwind CSS 4+ with Radix UI + shadcn/ui
- **Tailwind CSS:** Utility-first, mobile-first, responsive design, small bundle
- **Radix UI:** Unstyled accessible primitives (WCAG-compliant, keyboard navigation)
- **shadcn/ui:** Pre-built components (copy-paste, not package dependency)
- **Why this combo:** Supabase's exact stack - accessibility built-in, customizable, no vendor lock-in
- **Custom theme:** Brand colors, spacing, typography (via tailwind.config.ts)

**State Management:**
- **TanStack Query 5+** (formerly React Query): Server state, caching, background refetching
  - **Why:** Netflix-proven pattern, automatic cache invalidation, optimistic updates
  - **Use for:** Dify API calls, conversation history, media fetching
- **Zustand 4.4+**: Client state (UI state only)
  - **Why:** Lightweight (1KB), simple API, TypeScript-first
  - **Use for:** Modal open/close, selected option, user preferences

**HTTP Client:** Built into TanStack Query + fetch API
- **Why:** No Axios needed when using TanStack Query (built-in retries, interceptors)
- **Fallback:** Axios 1.6+ for non-query requests (file uploads)

### 2.2 Media Libraries

#### Video Player: react-player 2.13+
```typescript
import ReactPlayer from 'react-player';

<ReactPlayer
  url="https://r2.cloudflare.com/video.mp4"
  controls={true}
  width="100%"
  height="auto"
  playing={false} // User clicks to play
  muted={false}
  playsinline={true} // Important for mobile
  config={{
    file: {
      attributes: {
        controlsList: 'nodownload', // Prevent download
        disablePictureInPicture: false
      }
    }
  }}
/>
```

**Features:**
- Supports: MP4, MOV, WebM, HLS, YouTube, Vimeo
- Mobile controls (play, pause, seek, fullscreen)
- Adaptive bitrate (HLS/DASH)
- Event handlers (onProgress, onEnded, onError)
- Thumbnail support
- Picture-in-picture

**Why react-player:**
- 30K+ GitHub stars, battle-tested
- Works on iOS Safari (notoriously difficult)
- Handles video format fallbacks
- Lightweight (50KB gzipped)

#### Image Gallery: react-image-gallery 1.3+
```typescript
import ImageGallery from 'react-image-gallery';

<ImageGallery
  items={[
    {
      original: 'https://r2.cloudflare.com/img1.jpg',
      thumbnail: 'https://r2.cloudflare.com/img1-thumb.jpg',
      description: '[product-name] installed on construction site'
    },
    {
      original: 'https://r2.cloudflare.com/img2.jpg',
      thumbnail: 'https://r2.cloudflare.com/img2-thumb.jpg',
      description: 'Close-up of product specifications'
    }
  ]}
  showThumbnails={true}
  showFullscreenButton={true}
  showPlayButton={false}
  slideOnThumbnailOver={false}
  useBrowserFullscreen={true}
  onSlide={(currentIndex) => {
    console.log('User viewing image', currentIndex);
  }}
/>
```

**Features:**
- Touch swipe on mobile
- Keyboard arrow navigation on desktop
- Thumbnail strip
- Fullscreen mode
- Lazy loading
- Responsive

**Why react-image-gallery:**
- Production-proven (11K+ stars)
- Mobile-optimized
- Accessible (ARIA labels, keyboard nav)
- Customizable styling

#### GIF Player: Custom Component
```typescript
// Built on top of <img> tag with loading optimizations
const GifPlayer: React.FC<{url: string}> = ({url}) => {
  const [isLoaded, setIsLoaded] = useState(false);

  return (
    <div className="gif-container">
      {!isLoaded && <Skeleton />}
      <img
        src={url}
        alt="Animated content"
        loading="lazy"
        onLoad={() => setIsLoaded(true)}
        className="max-w-full h-auto"
      />
    </div>
  );
};
```

**Why custom:**
- GIFs are simple <img> tags
- No need for heavy library
- Browser handles playback natively
- Lazy loading optimization

### 2.3 UI Component Library

**shadcn/ui** (Radix UI + Tailwind)
- **Why shadcn/ui:** Copy-paste components, fully customizable, accessible, TypeScript-native
- **Components used:**
  - Button (primary, secondary, ghost variants)
  - Card (for option containers)
  - Dialog (for confirmations)
  - Skeleton (loading states)
  - Toast (notifications)
  - Avatar (user profile)
  - ScrollArea (chat scrolling)

### 2.4 Animation Library

**Framer Motion 11.0+**
```typescript
import { motion } from 'framer-motion';

<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  exit={{ opacity: 0, y: -20 }}
  transition={{ duration: 0.3 }}
>
  {/* Option card content */}
</motion.div>
```

**Animations:**
- Message fade-in (smooth, not jarring)
- Option card slide-up (delightful micro-interaction)
- Media loading skeleton pulse
- Button press feedback (scale down 0.95x on tap)

**Why Framer Motion:**
- Best animation library for React
- 60fps performance
- Spring physics (natural feel)
- Gesture support (drag, swipe)
- 18KB gzipped

### 2.5 Utility Libraries

**date-fns 3.0+** (date formatting)
- Display timestamps: "2 minutes ago", "Yesterday at 3:45 PM"

**clsx + tailwind-merge** (conditional classes)
```typescript
import { cn } from '@/lib/utils';

<div className={cn(
  "p-4 rounded-lg",
  isSelected && "bg-blue-100 border-blue-500",
  !isSelected && "bg-gray-50 border-gray-200"
)}>
```

**react-hook-form 7.48+** + **zod 3.22+** (forms + validation)
- Input validation for chat text
- Emoji picker integration
- File upload (if user wants to provide custom media)
- Schema validation with Zod (type-safe, runtime validation)

### 2.6 Testing Stack
**Inspired by**: Supabase (Vitest + Playwright), Netflix (SafeTest)

**Unit & Integration Testing:** Vitest 1.0+ + React Testing Library 14+
```typescript
// Example: Testing option selection component
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import OptionCard from './OptionCard';

describe('OptionCard', () => {
  it('calls onSelect when clicked', () => {
    const onSelect = vi.fn();
    render(<OptionCard id="A" onSelect={onSelect} />);

    fireEvent.click(screen.getByRole('button'));
    expect(onSelect).toHaveBeenCalledWith('A');
  });
});
```

**Why Vitest:**
- **10x faster** than Jest (Vite-native, ESM-first)
- **Compatible** with Jest API (easy migration)
- **Used by:** Supabase, Vue.js, Nuxt, Vite ecosystem
- **Features:** Watch mode, UI mode, coverage reports

**End-to-End Testing:** Playwright 1.40+
```typescript
// Example: Testing full conversation flow
import { test, expect } from '@playwright/test';

test('user can generate and select LinkedIn post', async ({ page }) => {
  await page.goto('http://localhost:3000');

  // Type message
  await page.fill('[data-testid="chat-input"]', 'Create LinkedIn post about [product-name]');
  await page.click('[data-testid="send-button"]');

  // Wait for options to appear
  await expect(page.locator('[data-testid="option-A"]')).toBeVisible();

  // Verify image displayed
  const image = page.locator('[data-testid="option-A-image"]');
  await expect(image).toHaveAttribute('src', /.*\.jpg$/);

  // Select option B
  await page.click('[data-testid="select-B"]');

  // Verify confirmation
  await expect(page.locator('text=Published to LinkedIn')).toBeVisible();
});
```

**Why Playwright:**
- **Cross-browser:** Chrome, Firefox, Safari, Edge
- **Mobile testing:** iOS Safari, Android Chrome
- **Used by:** Microsoft, GitHub, Supabase
- **Features:** Auto-wait, screenshot, video recording, network mocking

**Component Testing:** Storybook 7+ (optional, for design system)
- Visual regression testing
- Component documentation
- Isolated component development

**Test Coverage Targets:**
- **Unit tests:** 80%+ coverage (critical business logic)
- **Integration tests:** Key user flows (5-10 scenarios)
- **E2E tests:** Happy path + error cases (10-15 tests)

### 2.7 Developer Experience & Tooling

**Code Quality:** ESLint 8+ + Prettier 3+
```json
// .eslintrc.json
{
  "extends": [
    "next/core-web-vitals",
    "plugin:@typescript-eslint/recommended",
    "prettier"
  ],
  "rules": {
    "no-console": "warn",
    "@typescript-eslint/no-unused-vars": "error"
  }
}
```

**Pre-commit Hooks:** Husky 8+ + lint-staged
- Auto-format on commit
- Run linting before commit
- Run tests before push

**Type Generation:** TypeScript strict mode + Zod schemas
```typescript
// Auto-generate types from Dify API responses
import { z } from 'zod';

const DifyResponseSchema = z.object({
  conversation_id: z.string(),
  answer: z.string(),
  metadata: z.object({
    usage: z.number(),
    retrieval_results: z.array(z.any())
  })
});

type DifyResponse = z.infer<typeof DifyResponseSchema>;
```

**Error Tracking:** Sentry 7+ (client-side errors)
- Real-time error alerts
- Source map support (see original TypeScript code, not minified)
- Performance monitoring (Web Vitals)
- User feedback widget

**Analytics:** Cloudflare Web Analytics + Google Analytics 4
- Page views, button clicks, conversion funnel
- Web Vitals (LCP, FID, CLS)
- User demographics, device types

**Local Development:**
```bash
pnpm dev          # Start Next.js dev server (http://localhost:3000)
pnpm test         # Run Vitest unit tests
pnpm test:e2e     # Run Playwright E2E tests
pnpm lint         # Run ESLint
pnpm format       # Run Prettier
pnpm build        # Production build
pnpm preview      # Preview production build locally
```

### 2.8 Performance Targets
**Inspired by**: Netflix production standards

**Core Web Vitals (Lighthouse Score 90+):**
- **Largest Contentful Paint (LCP):** < 1.8s (image loads fast)
- **First Input Delay (FID):** < 100ms (chat input responsive)
- **Cumulative Layout Shift (CLS):** < 0.1 (no layout jumps)
- **Time to Interactive (TTI):** < 3.9s (user can interact quickly)

**Bundle Size Budgets:**
- **Initial JS bundle:** < 200 KB gzipped (Next.js code splitting helps)
- **CSS bundle:** < 20 KB gzipped (Tailwind purges unused classes)
- **Total page weight:** < 1 MB (excluding media content)

**Image Optimization:**
- Use Next.js `<Image>` component (automatic WebP, lazy loading, srcset)
- Cloudflare R2 serves images with cache headers
- Responsive images: serve 1200px on desktop, 600px on mobile

**Video Optimization:**
- HLS adaptive streaming (1080p → 720p → 480p based on bandwidth)
- Preload: metadata only (don't download full video upfront)
- Muted autoplay on mobile (saves data)

**Code Splitting Strategy:**
- Route-based (each page = separate bundle)
- Component-based (lazy load VideoPlayer only when needed)
```typescript
const VideoPlayer = lazy(() => import('./VideoPlayer'));
```

**Caching Strategy:**
- TanStack Query: 5 min stale time for conversation history
- Cloudflare CDN: Cache static assets for 1 year
- Service Worker (optional): Offline mode for chat history

**Performance Monitoring:**
```typescript
// Track custom metrics
import { trackEvent } from '@/lib/analytics'; // Cloudflare Web Analytics

const trackMediaLoad = (mediaType: 'image' | 'video', loadTime: number) => {
  trackEvent({
    event: 'media_load',
    properties: { mediaType, loadTime }
  });
};
```

**Performance Checklist:**
- ✅ Server-side rendering (SSR) for initial load
- ✅ Static generation (SSG) for marketing pages
- ✅ Image optimization (WebP, lazy load, srcset)
- ✅ Code splitting (routes + components)
- ✅ Tree shaking (remove unused code)
- ✅ Compression (Gzip/Brotli on Cloudflare)
- ✅ CDN caching (Cloudflare edge network)
- ✅ Prefetching (Next.js Link component)

---

## 3. Media Components Specification

### 3.1 Video Player Component

**Component:** `<VideoPlayer />`

**Props:**
```typescript
interface VideoPlayerProps {
  url: string;              // Cloudflare R2 URL
  thumbnail?: string;       // Poster image
  autoplay?: boolean;       // Default: false
  muted?: boolean;          // Default: false (true if autoplay)
  loop?: boolean;           // Default: false
  controls?: boolean;       // Default: true
  width?: string;           // Default: "100%"
  height?: string;          // Default: "auto"
  onPlay?: () => void;
  onPause?: () => void;
  onEnded?: () => void;
  onError?: (error: any) => void;
}
```

**Features:**
1. **Adaptive Quality:**
   - Detects network speed
   - Switches between 1080p/720p/480p automatically
   - Uses HLS if source supports it

2. **Mobile Optimizations:**
   - `playsInline` attribute (prevents fullscreen takeover on iOS)
   - Touch controls (tap to play/pause, double-tap to skip 10s)
   - Fullscreen button (opens native fullscreen)
   - Orientation lock in fullscreen (landscape for horizontal videos)

3. **Accessibility:**
   - Keyboard controls (Space = play/pause, Arrow keys = seek)
   - Screen reader announcements ("Video playing", "Video paused")
   - Captions support (WebVTT)

4. **Performance:**
   - Lazy loading (video not fetched until user scrolls near it)
   - Preload="metadata" (loads duration/thumbnail, not full video)
   - Intersection Observer (starts buffering when 50% visible)

**Example Usage:**
```typescript
<VideoPlayer
  url="https://r2.cloudflare.com/mesh-spacers-reel.mp4"
  thumbnail="https://r2.cloudflare.com/mesh-spacers-thumb.jpg"
  width="100%"
  height="600px"
  controls={true}
  autoplay={false}
  onPlay={() => console.log('User started video')}
/>
```

### 3.2 Image Gallery Component

**Component:** `<ImageGallery />`

**Props:**
```typescript
interface ImageGalleryProps {
  images: {
    url: string;          // Full-resolution URL
    thumbnail?: string;   // Thumbnail URL (lazy loading)
    alt: string;          // Accessibility description
    caption?: string;     // Display caption below image
  }[];
  currentIndex?: number;  // Controlled mode
  onImageChange?: (index: number) => void;
  showThumbnails?: boolean; // Default: true
  showNavigation?: boolean; // Arrow buttons, default: true
  showFullscreen?: boolean; // Fullscreen button, default: true
  infinite?: boolean;       // Loop at end, default: true
}
```

**Features:**
1. **Navigation:**
   - Swipe left/right on mobile (gesture detection)
   - Arrow keys on desktop (←/→)
   - Thumbnail strip click (jump to specific image)
   - Dot indicators (shows 1 of N)

2. **Fullscreen Mode:**
   - Overlays full viewport
   - Black background
   - Close button (X)
   - Zoom in/out (pinch on mobile, wheel on desktop)
   - Pan when zoomed (drag gesture)

3. **Lazy Loading:**
   - Only load visible image + next/prev (3 images max in memory)
   - Progressive JPEG loading (blurry → sharp)
   - Skeleton placeholder while loading

4. **Responsive:**
   - Desktop: 1200x627px (LinkedIn native size)
   - Tablet: 100% width, auto height (maintains aspect ratio)
   - Mobile: 100vw, auto height

**Example Usage:**
```typescript
<ImageGallery
  images={[
    {
      url: 'https://r2.cloudflare.com/product-1.jpg',
      thumbnail: 'https://r2.cloudflare.com/product-1-thumb.jpg',
      alt: '[product-name] on construction site',
      caption: 'A193 [product-name] installed - project example'
    },
    {
      url: 'https://r2.cloudflare.com/product-2.jpg',
      alt: 'Close-up of [product-name] specifications'
    }
  ]}
  showThumbnails={true}
  infinite={true}
  onImageChange={(index) => console.log('User viewing image', index)}
/>
```

### 3.3 Carousel Component

**Component:** `<Carousel />` (extends ImageGallery for LinkedIn multi-image posts)

**Props:**
```typescript
interface CarouselProps extends ImageGalleryProps {
  maxImages?: number;     // Default: 9 (LinkedIn limit)
  layout?: 'grid' | 'slider'; // Grid (2x2) or horizontal slider
  aspectRatio?: string;   // "16:9" | "1:1" | "4:5"
}
```

**Features:**
1. **Grid Layout (for 4+ images):**
   ```
   ┌─────┬─────┐
   │  1  │  2  │
   ├─────┼─────┤
   │  3  │  4  │
   └─────┴─────┘
   ```
   - Click any tile → opens fullscreen gallery at that index
   - Hover shows "1 of 9" overlay

2. **Slider Layout (for 2-3 images):**
   - Horizontal scroll
   - Snap to each image (CSS scroll-snap)
   - Progress dots below

3. **Mobile Gestures:**
   - Swipe to navigate
   - Long-press to preview (iOS haptic feedback)

**Example Usage:**
```typescript
<Carousel
  images={carouselImages}
  layout="grid"
  aspectRatio="1:1"
  maxImages={9}
/>
```

### 3.4 Option Card Component

**Component:** `<OptionCard />` (combines media + text + metadata)

**Props:**
```typescript
interface OptionCardProps {
  id: string;                   // "A", "B", "C"
  mediaType: 'image' | 'video' | 'carousel' | 'gif';
  mediaUrl?: string;            // Single media URL
  mediaUrls?: string[];         // Multiple URLs for carousel
  thumbnail?: string;           // Video thumbnail
  text: string;                 // Post text content
  platform: 'linkedin' | 'twitter' | 'facebook' | 'instagram';
  engagementScore: number;      // 0-10
  brandVoiceScore: number;      // 0-100
  isSelected?: boolean;
  onSelect: (id: string) => void;
}
```

**Layout:**
```
┌────────────────────────────────────────────┐
│  Option A: Professional Product Shot       │
├────────────────────────────────────────────┤
│                                            │
│   [HIGH-RES IMAGE OR VIDEO PLAYER]         │
│                                            │
├────────────────────────────────────────────┤
│  Text Preview:                             │
│  "Announcing our new A193 [product-name]..." │
│                                            │
│  📊 Engagement Score: 8.5/10               │
│  🎨 Brand Voice Score: 94/100              │
│  📱 Platform: LinkedIn                     │
│                                            │
│  [ Choose Option A ]  button               │
└────────────────────────────────────────────┘
```

**States:**
- **Default:** Gray border, white background
- **Hover:** Blue border, shadow elevation
- **Selected:** Blue background, checkmark icon, disabled other options

**Example Usage:**
```typescript
<OptionCard
  id="A"
  mediaType="video"
  mediaUrl="https://r2.cloudflare.com/option-a.mp4"
  thumbnail="https://r2.cloudflare.com/option-a-thumb.jpg"
  text="Announcing our new A193 [product-name] for large-scale construction projects..."
  platform="linkedin"
  engagementScore={8.5}
  brandVoiceScore={94}
  isSelected={selectedOption === 'A'}
  onSelect={(id) => setSelectedOption(id)}
/>
```

### 3.5 Chat Message Component

**Component:** `<ChatMessage />` (displays bot/user messages with media)

**Props:**
```typescript
interface ChatMessageProps {
  role: 'user' | 'assistant';
  content: string | ReactNode; // Text or rich media components
  timestamp: Date;
  avatar?: string;
  isLoading?: boolean; // Show typing indicator
}
```

**Layout:**

**User Message:**
```
                              ┌──────────────────┐
                              │ Create LinkedIn  │
                              │ post about mesh  │  [avatar]
                              │ spacers          │
                              └──────────────────┘
                                  3:45 PM
```

**Assistant Message with Media:**
```
[avatar]  ┌──────────────────────────────────┐
          │ Here are 3 options for your      │
          │ LinkedIn post:                   │
          │                                  │
          │  [Option A Card with image]      │
          │  [Option B Card with video]      │
          │  [Option C Card with carousel]   │
          │                                  │
          │ Which would you like to publish? │
          │ (Type A, B, or C)                │
          └──────────────────────────────────┘
              3:46 PM
```

**Features:**
- Markdown rendering (bold, italic, lists, links)
- Code blocks with syntax highlighting
- Media embeds (handled by child components)
- Copy button (copy text to clipboard)
- Retry button (if message failed to send)

---

## 4. Platform-Specific Media Requirements

### 4.1 LinkedIn

**Images:**
- **Aspect Ratio:** 1.91:1 (1200x627px recommended)
- **Max Size:** 10 MB
- **Formats:** JPG, PNG, GIF (static)
- **Carousels:** Up to 9 images, all same aspect ratio

**Videos:**
- **Aspect Ratios:** 16:9 (landscape), 1:1 (square), 9:16 (vertical)
- **Duration:** 3 seconds - 10 minutes
- **Max Size:** 5 GB
- **Formats:** MP4, MOV, AVI
- **Codec:** H.264, AAC audio
- **Resolution:** 1080p recommended (minimum 360p)

**Frontend Requirements:**
```typescript
const LinkedInMediaSpecs = {
  image: {
    width: 1200,
    height: 627,
    aspectRatio: 1.91,
    maxSizeMB: 10,
    formats: ['jpg', 'png']
  },
  video: {
    aspectRatios: ['16:9', '1:1', '9:16'],
    minDuration: 3,
    maxDuration: 600,
    maxSizeMB: 5120,
    resolution: '1080p',
    formats: ['mp4', 'mov']
  },
  carousel: {
    maxImages: 9,
    aspectRatio: '1:1' // All images must match
  }
};
```

### 4.2 Twitter/X

**Images:**
- **Aspect Ratio:** 16:9 (1200x675px recommended)
- **Max Size:** 5 MB
- **Formats:** JPG, PNG, GIF (animated supported)
- **Multiple:** Up to 4 images per tweet

**Videos:**
- **Aspect Ratios:** 16:9, 1:1, 9:16
- **Duration:** 2 seconds - 2:20 minutes
- **Max Size:** 512 MB
- **Formats:** MP4, MOV
- **Resolution:** 1920x1080 or 1280x720 recommended
- **Frame Rate:** 30-60 FPS

**GIFs:**
- **Max Size:** 15 MB
- **Resolution:** Up to 1280x1080
- **Animated:** Yes (loops automatically)

**Frontend Requirements:**
```typescript
const TwitterMediaSpecs = {
  image: {
    width: 1200,
    height: 675,
    aspectRatio: 1.78,
    maxSizeMB: 5,
    formats: ['jpg', 'png'],
    maxImages: 4
  },
  video: {
    maxDuration: 140,
    maxSizeMB: 512,
    resolution: '1080p',
    formats: ['mp4', 'mov']
  },
  gif: {
    maxSizeMB: 15,
    maxResolution: '1280x1080',
    animated: true
  }
};
```

### 4.3 Facebook

**Images:**
- **Aspect Ratio:** 1.91:1 (1200x630px recommended)
- **Max Size:** 10 MB
- **Formats:** JPG, PNG, GIF (static)

**Videos:**
- **Aspect Ratios:** 16:9, 1:1, 9:16, 4:5
- **Duration:** 1 second - 240 minutes
- **Max Size:** 10 GB
- **Formats:** MP4, MOV
- **Resolution:** 1080p recommended
- **Captions:** Recommended (auto-generated available)

**Frontend Requirements:**
```typescript
const FacebookMediaSpecs = {
  image: {
    width: 1200,
    height: 630,
    aspectRatio: 1.91,
    maxSizeMB: 10,
    formats: ['jpg', 'png']
  },
  video: {
    aspectRatios: ['16:9', '1:1', '9:16', '4:5'],
    minDuration: 1,
    maxDuration: 14400,
    maxSizeMB: 10240,
    resolution: '1080p',
    captions: true
  }
};
```

### 4.4 Instagram

**Feed Posts (Images):**
- **Aspect Ratio:** 1:1 (1080x1080px), 4:5 (1080x1350px), 1.91:1 (1080x566px)
- **Max Size:** 8 MB
- **Formats:** JPG, PNG
- **Carousels:** Up to 10 images

**Feed Posts (Videos):**
- **Aspect Ratio:** 1:1, 4:5, 16:9
- **Duration:** 3 seconds - 60 minutes
- **Max Size:** 4 GB
- **Formats:** MP4, MOV
- **Resolution:** 1080x1080 (square), 1080x1350 (portrait)

**Reels:**
- **Aspect Ratio:** 9:16 (1080x1920px vertical)
- **Duration:** 15-90 seconds
- **Max Size:** 4 GB
- **Resolution:** 1080x1920 mandatory
- **Audio:** Music library or original

**Stories:**
- **Aspect Ratio:** 9:16 (1080x1920px)
- **Duration:** Up to 60 seconds (auto-splits if longer)
- **Formats:** MP4, MOV, JPG, PNG, GIF

**Frontend Requirements:**
```typescript
const InstagramMediaSpecs = {
  feed: {
    image: {
      aspectRatios: ['1:1', '4:5', '1.91:1'],
      width: 1080,
      heights: [1080, 1350, 566],
      maxSizeMB: 8
    },
    video: {
      aspectRatios: ['1:1', '4:5', '16:9'],
      maxDuration: 3600,
      maxSizeMB: 4096,
      resolution: '1080x1080'
    },
    carousel: {
      maxImages: 10,
      aspectRatio: '1:1' // Must match
    }
  },
  reels: {
    aspectRatio: '9:16',
    width: 1080,
    height: 1920,
    minDuration: 15,
    maxDuration: 90,
    maxSizeMB: 4096
  },
  stories: {
    aspectRatio: '9:16',
    width: 1080,
    height: 1920,
    maxDuration: 60,
    formats: ['mp4', 'mov', 'jpg', 'png', 'gif']
  }
};
```

### 4.5 Media Validation Component

**Component:** `<MediaValidator />` (validates media before upload)

```typescript
const validateMedia = (
  file: File,
  platform: Platform,
  postType: 'image' | 'video' | 'carousel' | 'reel' | 'story'
): ValidationResult => {
  const specs = getMediaSpecs(platform, postType);

  const errors: string[] = [];

  // Check file size
  if (file.size > specs.maxSizeMB * 1024 * 1024) {
    errors.push(`File too large. Max: ${specs.maxSizeMB} MB`);
  }

  // Check format
  const fileExt = file.name.split('.').pop()?.toLowerCase();
  if (!specs.formats.includes(fileExt)) {
    errors.push(`Invalid format. Allowed: ${specs.formats.join(', ')}`);
  }

  // Check dimensions (for images)
  if (postType.includes('image')) {
    const img = new Image();
    img.src = URL.createObjectURL(file);
    img.onload = () => {
      const aspectRatio = img.width / img.height;
      const expectedRatio = specs.aspectRatio;

      if (Math.abs(aspectRatio - expectedRatio) > 0.01) {
        errors.push(`Aspect ratio mismatch. Expected: ${expectedRatio}`);
      }
    };
  }

  // Check video duration
  if (postType.includes('video')) {
    const video = document.createElement('video');
    video.src = URL.createObjectURL(file);
    video.onloadedmetadata = () => {
      if (video.duration < specs.minDuration || video.duration > specs.maxDuration) {
        errors.push(`Duration out of range. Allowed: ${specs.minDuration}-${specs.maxDuration}s`);
      }
    };
  }

  return {
    isValid: errors.length === 0,
    errors
  };
};
```

---

## 5. Dify Backend Integration

### 5.1 Dify Setup (Cloud Professional)

**Plan:** Dify Cloud Professional
- **Cost:** $59/month
- **Includes:**
  - 5,000 messages/month (sufficient for 100 customers × 50 messages/month)
  - 500 documents across all KBs
  - 5 GB storage
  - Unlimited workflows
  - HTTP tool access
  - Knowledge base queries

**Account Setup:**
1. Sign up at https://cloud.dify.ai
2. Create organization: "Company XYZ Social System"
3. Create app: "Social Content Generator"
4. App type: "Chatbot" (multi-turn conversation)
5. Model: GPT-4 Turbo (for workflow orchestration)

### 5.2 Knowledge Bases (4)

#### KB 1: Brand Voice
**Documents (15):**
- Brand guidelines PDF (tone, voice, personality)
- Example high-performing posts (text + metadata)
- Lexicon (industry terms, company-specific phrases)
- Prohibited words/phrases
- Visual brand guide (colors, fonts, logo usage)

**Embedding Model:** OpenAI text-embedding-3-large (3072 dimensions)
- **Why:** Best accuracy for semantic search
- **Cost:** $0.13 per 1M tokens (cheap for 15 docs)

**Retrieval Strategy:** Hybrid (semantic + keyword)
- Query: "What tone should we use for LinkedIn?"
- Returns: Relevant sections from brand guidelines

#### KB 2: Skill Library
**Documents (50):**
- Post templates by type (product announcement, case study, how-to, etc.)
- Layout specifications (image placement, text length)
- Hashtag strategies per platform
- CTA (Call-to-Action) formulas
- Structural patterns (hooks, body, conclusion)

**Embedding Model:** Same as KB1 (OpenAI text-embedding-3-large)

**Retrieval Strategy:** Semantic search
- Query: "Find product announcement template for LinkedIn"
- Returns: Template with structure, example text, image specs

#### KB 3: Product Catalog
**Documents (100):**
- Product specs ([product-name], rebar, etc.)
- Pricing information
- Stock availability (synced from Shopify daily)
- Technical drawings
- Installation guides
- Customer FAQs

**Embedding Model:** Same as KB1

**Retrieval Strategy:** Keyword + semantic
- Query: "A193 [product-name]"
- Returns: Product page, specs, pricing, images

#### KB 4: Example Posts
**Documents (200):**
- Past LinkedIn posts with engagement metrics
- Past Twitter posts with metrics
- Past Facebook posts with metrics
- Past Instagram posts with metrics
- Each document includes: text, media URLs, engagement data, brand voice score

**Embedding Model:** Same as KB1

**Retrieval Strategy:** Semantic similarity
- Query: "High-performing product announcement posts"
- Returns: Top 5 similar posts with metrics
- Used for: Performance prediction, style inspiration

### 5.3 Workflows (5)

#### Workflow 1: Main Chatflow (Master Orchestrator)

**Trigger:** User message from frontend

**Nodes:**
1. **Start Node** (receives user message)
2. **LLM Node** (GPT-4 Turbo, intent classification)
   - Prompt: "Classify user intent: create_post, edit_post, publish, analytics, help, other"
   - Output: Intent category

3. **IF/ELSE Node** (route by intent)
   - **IF intent = "create_post":**
     - Call Workflow 2 (Skill Matcher)
     - Call Workflow 3 (Content Generator)
     - Return options to user

   - **ELSE IF intent = "publish":**
     - Parse user choice (A, B, or C)
     - Call Workflow 4 (Publishing)
     - Return confirmation

   - **ELSE IF intent = "analytics":**
     - Call Workflow 5 (Learning/Analytics)
     - Return insights

   - **ELSE:**
     - LLM Node (general conversation)

**Conversation Variables:**
- `conversation_id`: UUID
- `user_id`: Tenant ID
- `current_step`: "awaiting_choice" | "published" | "idle"
- `options_shown`: Array of option IDs
- `selected_option`: "A" | "B" | "C" | null

#### Workflow 2: Skill Matcher

**Trigger:** Called by Workflow 1

**Nodes:**
1. **Knowledge Retrieval Node** (queries KB 2: Skill Library)
   - Input: User request text
   - Output: Top 3 relevant skill templates

2. **HTTP Tool Node** (calls OpenManus meta-skill builder - Stage H)
   - Method: POST
   - URL: `https://openmanus-api.workers.dev/api/match-skill`
   - Body: `{"request": "...", "templates": [...]}`
   - Headers: `Authorization: Bearer {OPENMANUS_API_KEY}`
   - Output: Recommended skill template

3. **Variable Assigner Node**
   - Stores skill template in `workflow_skill_template`

4. **End Node** (returns to Workflow 1)

#### Workflow 3: Content Generator (Calls 11 Stage D Microservices)

**Trigger:** Called by Workflow 1

**Nodes:**
1. **HTTP Tool: Intent Parser** (Stage D Service 1)
   - URL: `https://mcp-gateway.workers.dev/api/intent-parser`
   - Body: `{"text": "...", "tenant_id": "..."}`
   - Output: Parsed intent + entities

2. **HTTP Tool: Skill Retrieval** (Stage D Service 2)
   - URL: `https://mcp-gateway.workers.dev/api/skill-retrieval`
   - Body: `{"intent": "...", "skill_template": "..."}`
   - Output: Retrieved skill with parameters

3. **HTTP Tool: Multi-Candidate Generator** (Stage D Service 3)
   - **CRITICAL:** Generates 3 options (A, B, C)
   - URL: `https://mcp-gateway.workers.dev/api/generate-candidates`
   - Body: `{"skill": "...", "count": 3, "tenant_id": "..."}`
   - Output:
     ```json
     {
       "candidates": [
         {
           "id": "A",
           "text": "...",
           "mediaType": "image",
           "mediaUrl": "https://r2.cloudflare.com/...",
           "engagementScore": 8.5,
           "brandVoiceScore": 94
         },
         {"id": "B", ...},
         {"id": "C", ...}
       ]
     }
     ```

4. **HTTP Tool: Guards Engine** (Stage D Service 4)
   - URL: `https://mcp-gateway.workers.dev/api/guards`
   - Body: `{"candidates": [...]}`
   - Output: Validated candidates (auto-fixed if issues found)

5. **HTTP Tool: Reranker** (Stage D Service 5)
   - URL: `https://mcp-gateway.workers.dev/api/rerank`
   - Body: `{"candidates": [...]}`
   - Output: Sorted by predicted engagement (A = best, C = worst)

6. **HTTP Tool: Brand Voice Scorer** (Stage D Service 8)
   - URL: `https://mcp-gateway.workers.dev/api/brand-voice`
   - Body: `{"candidates": [...]}`
   - Output: Candidates with brandVoiceScore added

7. **Answer Node** (formats response for frontend)
   - Template:
     ```
     Here are 3 options for your {{platform}} post:

     {{#each candidates}}
     **Option {{this.id}}:**
     [Media: {{this.mediaType}} - {{this.mediaUrl}}]
     Text: "{{this.text}}"
     📊 Engagement Score: {{this.engagementScore}}/10
     🎨 Brand Voice Score: {{this.brandVoiceScore}}/100

     {{/each}}

     Which would you like to publish? (Type A, B, or C)
     ```
   - Output mode: **JSON** (not text)
     ```json
     {
       "type": "options",
       "options": [...],
       "prompt": "Which would you like to publish? (Type A, B, or C)"
     }
     ```

8. **Variable Assigner Node**
   - Stores candidates in `conversation_candidates`

9. **End Node**

#### Workflow 4: Publishing Workflow

**Trigger:** User types "A", "B", or "C"

**Nodes:**
1. **Code Node** (parse user choice)
   ```javascript
   const choice = input.user_message.toUpperCase(); // "A", "B", or "C"
   const candidates = context.conversation_candidates;
   const selected = candidates.find(c => c.id === choice);

   return { selected_candidate: selected };
   ```

2. **HTTP Tool: Publishing Service** (Stage D Service 9)
   - URL: `https://mcp-gateway.workers.dev/api/publish`
   - Body:
     ```json
     {
       "candidate": {...},
       "platform": "linkedin",
       "tenant_id": "..."
     }
     ```
   - Output: `{success: true, post_id: "..."}`

3. **IF/ELSE Node** (check if multi-platform)
   - **IF user wants to publish to multiple platforms:**
     - Parallel HTTP Tools (call Stage E adapters)
       - LinkedIn Adapter
       - Twitter Adapter
       - Facebook Adapter
       - Instagram Adapter
     - Each returns: `{success: true, post_url: "..."}`

4. **Answer Node**
   ```
   ✅ Successfully published to {{platform}}!

   Preview: {{post_url}}

   Would you like to:
   - Publish to other platforms? (type "yes")
   - View analytics? (type "analytics")
   - Create another post? (type "new")
   ```

5. **HTTP Tool: Analytics Service** (Stage D Service 10)
   - URL: `https://mcp-gateway.workers.dev/api/track-publish`
   - Body: `{post_id: "...", tenant_id: "...", timestamp: "..."}`
   - Purpose: Track for learning loop

6. **End Node**

#### Workflow 5: Learning & Analytics

**Trigger:** User types "analytics" or scheduled (daily)

**Nodes:**
1. **HTTP Tool: Analytics Collection** (Stage F Service)
   - URL: `https://analytics.workers.dev/api/collect`
   - Body: `{tenant_id: "...", date_range: "7d"}`
   - Output: Engagement metrics for last 7 days

2. **HTTP Tool: Performance Metrics** (Stage F Service)
   - URL: `https://analytics.workers.dev/api/metrics`
   - Output: KPIs (engagement rate, reach, virality)

3. **LLM Node** (GPT-4, generate insights)
   - Prompt:
     ```
     Analyze these metrics and provide insights:
     {{metrics}}

     Format:
     - Top performing post (why?)
     - Low performing post (why?)
     - Recommendations for next post
     ```

4. **Answer Node**
   ```
   📊 Analytics Summary (Last 7 Days)

   🏆 Top Post: "{{top_post.title}}"
   - Engagement Rate: {{top_post.engagement_rate}}%
   - Why it worked: {{insights.top_reasons}}

   📉 Needs Improvement: "{{low_post.title}}"
   - Engagement Rate: {{low_post.engagement_rate}}%
   - What to fix: {{insights.improvement_areas}}

   💡 Recommendations:
   {{insights.recommendations}}
   ```

5. **HTTP Tool: Learning Engine** (Stage D Service 7)
   - URL: `https://mcp-gateway.workers.dev/api/learn`
   - Body: `{metrics: {...}, tenant_id: "..."}`
   - Purpose: Update skill confidence scores

6. **End Node**

### 5.4 Dify API Integration (Frontend ↔ Backend)

**API Endpoint:** `https://api.dify.ai/v1`

**Authentication:**
```typescript
// Frontend stores Dify API key (per tenant)
const DIFY_API_KEY = process.env.NEXT_PUBLIC_DIFY_API_KEY;

axios.defaults.headers.common['Authorization'] = `Bearer ${DIFY_API_KEY}`;
```

**Send Message:**
```typescript
const sendMessage = async (message: string, conversationId?: string) => {
  const response = await axios.post('https://api.dify.ai/v1/chat-messages', {
    inputs: {},
    query: message,
    response_mode: 'blocking', // Wait for full response
    conversation_id: conversationId || undefined,
    user: userId, // Tenant ID
    files: [] // No file uploads for now
  });

  return response.data;
};
```

**Response Format:**
```json
{
  "event": "message",
  "task_id": "uuid",
  "id": "message-uuid",
  "message_id": "uuid",
  "conversation_id": "conv-uuid",
  "mode": "chat",
  "answer": "{\"type\":\"options\",\"options\":[...]}",
  "metadata": {
    "usage": {"prompt_tokens": 100, "completion_tokens": 200}
  },
  "created_at": 1699999999
}
```

**Frontend Parsing:**
```typescript
const handleDifyResponse = (response: DifyResponse) => {
  try {
    // Try parsing as JSON (for structured responses like options)
    const parsed = JSON.parse(response.answer);

    if (parsed.type === 'options') {
      // Render option cards with media
      setOptions(parsed.options);
      setPrompt(parsed.prompt);
    }
  } catch {
    // Plain text response
    addMessage({
      role: 'assistant',
      content: response.answer,
      timestamp: new Date(response.created_at * 1000)
    });
  }
};
```

**Streaming (Future Enhancement):**
```typescript
// Use Server-Sent Events for streaming responses
const streamMessage = async (message: string) => {
  const response = await fetch('https://api.dify.ai/v1/chat-messages', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${DIFY_API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      inputs: {},
      query: message,
      response_mode: 'streaming', // Stream responses
      user: userId
    })
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    const lines = chunk.split('\n').filter(l => l.startsWith('data: '));

    for (const line of lines) {
      const data = JSON.parse(line.replace('data: ', ''));

      if (data.event === 'message') {
        // Append to current message (typewriter effect)
        appendToMessage(data.answer);
      }
    }
  }
};
```

---

## 6. Conversational Advisory Pattern

### 6.1 Multi-Turn Conversation Flow

**Example Conversation:**

```
User: "Create a LinkedIn post about our new A193 [product-name]"
  ↓
Bot: "I'll create a LinkedIn post about A193 [product-name].
      What's the main goal of this post?

      A) Product announcement
      B) Educational (how to use)
      C) Case study (project example)"
  ↓
User: "A"
  ↓
Bot: [Generating content...]
     "Here are 3 options for your LinkedIn post:

      [Option A Card: Image + Text + Scores]
      [Option B Card: Video + Text + Scores]
      [Option C Card: Carousel + Text + Scores]

      Which would you like to publish? (Type A, B, or C)"
  ↓
User: "B"
  ↓
Bot: "Great choice! Option B has a predicted engagement rate of 7.2/10.

     Would you like to:
     - Publish now (type 'publish')
     - Edit the text first (type 'edit')
     - Try different image (type 'regenerate')"
  ↓
User: "publish"
  ↓
Bot: "✅ Published to LinkedIn!

     Preview: https://linkedin.com/feed/update/123456

     I'll track performance and let you know results in 24 hours.

     What would you like to do next?"
```

### 6.2 State Management

**Dify Conversation Variables:**
```typescript
{
  conversation_id: "uuid",
  user_id: "tenant-123",
  current_step: "awaiting_choice", // idle | selecting_goal | generating | awaiting_choice | publishing | published
  post_goal: "product_announcement",
  options_shown: ["A", "B", "C"],
  selected_option: "B",
  platform: "linkedin",
  history: [
    {step: "goal_selection", choice: "A", timestamp: "..."},
    {step: "option_selection", choice: "B", timestamp: "..."}
  ]
}
```

**Frontend State:**
```typescript
interface ChatState {
  messages: ChatMessage[];
  conversationId: string | null;
  currentStep: ConversationStep;
  options: OptionCard[] | null;
  selectedOption: string | null;
  isLoading: boolean;
  error: string | null;
}

const useChatStore = create<ChatState>((set) => ({
  messages: [],
  conversationId: null,
  currentStep: 'idle',
  options: null,
  selectedOption: null,
  isLoading: false,
  error: null,

  sendMessage: async (text: string) => {
    set({ isLoading: true });

    const response = await sendMessageToDify(text, get().conversationId);

    set({
      messages: [...get().messages, userMessage, assistantMessage],
      conversationId: response.conversation_id,
      isLoading: false
    });
  },

  selectOption: (optionId: string) => {
    set({ selectedOption: optionId });
    // Send selection to Dify
    sendMessage(optionId);
  }
}));
```

### 6.3 Text-Based Options (No GUI Buttons)

**Why Text-Only:**
- **Mobile-first:** Typing "A" is faster than tapping small buttons on phone
- **Accessible:** Screen readers handle text better than interactive buttons
- **Simple:** No complex UI state management
- **Natural:** Feels like conversation, not form filling

**User Experience:**
1. Bot presents options as **text with visual previews** (not clickable buttons)
2. User types "A", "B", or "C" (case-insensitive)
3. Bot confirms selection and proceeds

**Frontend Implementation:**
```typescript
const OptionDisplay: React.FC<{options: OptionCard[]}> = ({options}) => {
  return (
    <div className="space-y-4">
      {options.map(option => (
        <div key={option.id} className="border rounded-lg p-4">
          <h3 className="font-bold">Option {option.id}</h3>

          {/* Media Display */}
          {option.mediaType === 'video' && (
            <VideoPlayer url={option.mediaUrl} />
          )}
          {option.mediaType === 'image' && (
            <img src={option.mediaUrl} alt={option.text} />
          )}
          {option.mediaType === 'carousel' && (
            <ImageGallery images={option.mediaUrls} />
          )}

          {/* Text Preview */}
          <p className="mt-2">{option.text}</p>

          {/* Scores */}
          <div className="flex gap-4 mt-2 text-sm text-gray-600">
            <span>📊 {option.engagementScore}/10</span>
            <span>🎨 {option.brandVoiceScore}/100</span>
          </div>
        </div>
      ))}

      <p className="text-center text-gray-500 mt-4">
        Type A, B, or C to select
      </p>
    </div>
  );
};
```

**No Clickable Buttons:**
- Options are **display-only** (media + text + scores)
- User **types** their choice in chat input
- Dify workflow parses "A", "B", or "C" from user message

**Fallback Handling:**
```typescript
// In Dify workflow, Code Node:
const userInput = input.user_message.trim().toUpperCase();

if (['A', 'B', 'C'].includes(userInput)) {
  return { valid_choice: userInput };
} else {
  return {
    error: "Invalid choice. Please type A, B, or C to select an option."
  };
}
```

---

## 7. Mobile Optimization

### 7.1 Responsive Design

**Breakpoints (Tailwind):**
```typescript
const breakpoints = {
  sm: '640px',  // Mobile landscape
  md: '768px',  // Tablet portrait
  lg: '1024px', // Tablet landscape / Small laptop
  xl: '1280px', // Desktop
  '2xl': '1536px' // Large desktop
};
```

**Mobile-First CSS:**
```css
/* Base styles (mobile) */
.option-card {
  @apply p-4 rounded-lg;
}

/* Tablet and up */
@screen md {
  .option-card {
    @apply p-6 rounded-xl;
  }
}

/* Desktop */
@screen lg {
  .option-card {
    @apply p-8;
  }
}
```

### 7.2 Touch Optimization

**Touch Targets:**
- Minimum size: 44×44px (Apple HIG, Google Material)
- Padding around buttons: 8px minimum
- No clickable elements closer than 8px

**Gestures:**
```typescript
// Swipe to navigate carousel
import { useSwipeable } from 'react-swipeable';

const handlers = useSwipeable({
  onSwipedLeft: () => nextImage(),
  onSwipedRight: () => prevImage(),
  preventScrollOnSwipe: true,
  trackMouse: false // Only touch, not mouse drag
});

<div {...handlers}>
  <ImageGallery />
</div>
```

**Haptic Feedback (iOS):**
```typescript
const triggerHaptic = () => {
  if ('vibrate' in navigator) {
    navigator.vibrate(10); // 10ms vibration on selection
  }
};

const selectOption = (id: string) => {
  triggerHaptic();
  // ... rest of selection logic
};
```

### 7.3 Video Optimization (Mobile)

**Adaptive Bitrate:**
```typescript
// HLS.js for adaptive streaming
import Hls from 'hls.js';

const VideoPlayer = ({url}) => {
  const videoRef = useRef<HTMLVideoElement>(null);

  useEffect(() => {
    if (Hls.isSupported() && url.endsWith('.m3u8')) {
      const hls = new Hls({
        maxBufferLength: 10, // Buffer 10s ahead
        maxMaxBufferLength: 20,
        startLevel: -1, // Auto-select quality
      });

      hls.loadSource(url);
      hls.attachMedia(videoRef.current);

      // Listen for quality changes
      hls.on(Hls.Events.LEVEL_SWITCHED, (event, data) => {
        console.log('Quality switched to', data.level);
      });

      return () => hls.destroy();
    } else if (videoRef.current.canPlayType('application/vnd.apple.mpegurl')) {
      // iOS native HLS support
      videoRef.current.src = url;
    }
  }, [url]);

  return <video ref={videoRef} controls playsInline />;
};
```

**Data Saver Mode:**
```typescript
const [dataSaverMode, setDataSaverMode] = useState(false);

// Detect slow connection
useEffect(() => {
  if ('connection' in navigator) {
    const conn = (navigator as any).connection;

    if (conn.effectiveType === '2g' || conn.effectiveType === 'slow-2g') {
      setDataSaverMode(true);
    }
  }
}, []);

// Show low-res thumbnail instead of auto-playing video
{dataSaverMode ? (
  <img src={thumbnail} onClick={() => setDataSaverMode(false)} />
) : (
  <VideoPlayer url={videoUrl} />
)}
```

**Orientation Lock:**
```typescript
// Lock to landscape for horizontal videos
const playFullscreen = async (videoRef: HTMLVideoElement) => {
  await videoRef.requestFullscreen();

  if ('orientation' in screen && 'lock' in screen.orientation) {
    try {
      await screen.orientation.lock('landscape');
    } catch (err) {
      console.log('Orientation lock not supported');
    }
  }
};
```

### 7.4 Performance Optimizations

**Lazy Loading:**
```typescript
// Intersection Observer for lazy loading media
const LazyMedia: React.FC<{src: string}> = ({src}) => {
  const [isVisible, setIsVisible] = useState(false);
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.disconnect();
        }
      },
      { threshold: 0.1 } // Load when 10% visible
    );

    if (ref.current) observer.observe(ref.current);

    return () => observer.disconnect();
  }, []);

  return (
    <div ref={ref}>
      {isVisible ? (
        <img src={src} alt="" />
      ) : (
        <Skeleton className="w-full h-64" />
      )}
    </div>
  );
};
```

**Image Optimization:**
```typescript
// Cloudflare Images (automatic optimization)
const optimizeImage = (url: string, width: number) => {
  // Cloudflare automatically serves WebP/AVIF to supported browsers
  return `${url}?width=${width}&quality=85&format=auto`;
};

// Usage:
<img
  src={optimizeImage(imageUrl, 1200)}
  srcSet={`
    ${optimizeImage(imageUrl, 400)} 400w,
    ${optimizeImage(imageUrl, 800)} 800w,
    ${optimizeImage(imageUrl, 1200)} 1200w
  `}
  sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 1200px"
  loading="lazy"
  alt="Product image"
/>
```

**Service Worker (Offline Support):**
```typescript
// Cache API responses for offline viewing
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js');
}

// sw.js
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});
```

---

## 8. API Specifications

### 8.1 Frontend → Dify API

**Endpoint:** `POST https://api.dify.ai/v1/chat-messages`

**Request:**
```json
{
  "inputs": {},
  "query": "Create a LinkedIn post about [product-name]",
  "response_mode": "blocking",
  "conversation_id": "conv-abc123",
  "user": "tenant-xyz",
  "files": []
}
```

**Response:**
```json
{
  "event": "message",
  "message_id": "msg-def456",
  "conversation_id": "conv-abc123",
  "answer": "{\"type\":\"options\",\"options\":[...]}",
  "created_at": 1699999999
}
```

### 8.2 Dify → MCP Gateway

**Endpoint:** `POST https://mcp-gateway.workers.dev/api/{service}`

**Example: Generate Candidates**

**Request:**
```json
{
  "service": "generate-candidates",
  "payload": {
    "skill": {
      "template": "product_announcement",
      "parameters": {
        "product": "A193 [product-name]",
        "platform": "linkedin"
      }
    },
    "count": 3,
    "tenant_id": "tenant-xyz"
  }
}
```

**Response:**
```json
{
  "success": true,
  "candidates": [
    {
      "id": "A",
      "text": "Excited to announce our new A193 [product-name]...",
      "mediaType": "image",
      "mediaUrl": "https://r2.cloudflare.com/product-a.jpg",
      "dimensions": "1200x627",
      "engagementScore": 8.5,
      "brandVoiceScore": 94
    },
    {
      "id": "B",
      "text": "See how our A193 [product-name] are transforming construction...",
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
}
```

### 8.3 MCP Gateway → Stage Services

See Stage D, E, F approved plans for detailed API specifications.

---

## 9. Database Schema

### 9.1 Frontend-Specific Tables

**No new database tables required.**

Frontend stores state in:
- Browser localStorage (user preferences, conversation history cache)
- Zustand state (ephemeral chat state)
- Dify handles conversation persistence

### 9.2 Dify Knowledge Base Storage

Dify stores knowledge base documents in its own PostgreSQL database (managed by Dify Cloud).

**We upload:**
- 15 brand voice documents
- 50 skill templates
- 100 product catalog entries
- 200 example posts

**Dify indexes with:**
- OpenAI text-embedding-3-large (3072D vectors)
- Vector database (pgvector extension in PostgreSQL)

---

## 10. Cost Analysis

### 10.1 Development Cost

| Phase | Duration | Engineers | Hours | Cost ($375/hr) |
|-------|----------|-----------|-------|----------------|
| Dify Backend Setup | 3 days | 1 backend | 24 | $9,000 |
| Knowledge Base Upload | 2 days | 1 backend | 16 | $6,000 |
| Workflow Configuration | 4 days | 1 backend | 32 | $12,000 |
| Custom Frontend (React) | 8 days | 1 frontend | 64 | $24,000 |
| Media Components | 4 days | 1 frontend | 32 | $12,000 |
| Dify API Integration | 3 days | 1 full-stack | 24 | $9,000 |
| Mobile Optimization | 3 days | 1 frontend | 24 | $9,000 |
| Testing (All Devices) | 3 days | 2 QA | 48 | $18,000 |
| Deployment & Polish | 2 days | 1 DevOps | 16 | $6,000 |
| **Total** | **23 days** | | **230 hours** | **$86,250** |

### 10.2 Monthly Operational Cost

| Service | Usage | Cost/Month |
|---------|-------|------------|
| **Dify Cloud Professional** | 5,000 messages, 500 docs, 5GB storage | $59 |
| **Cloudflare Pages** | Frontend hosting (unlimited bandwidth) | $0-5 |
| **OpenAI (Dify workflows)** | GPT-4 Turbo for orchestration (~50K tokens/day) | Included in Dify |
| **OpenAI (Knowledge Base)** | text-embedding-3-large (one-time indexing) | $0 (one-time $5) |
| **Media Storage (R2)** | Already budgeted in Stage E | $0 (included) |
| **Google Vision API** | Already budgeted in Stage E | $0 (included) |
| **Monitoring (Datadog)** | Already budgeted in Stage F | $0 (included) |
| **Total** | | **$59-64/month** |

**Per Customer:** $0.59-0.64/customer/month (for 100 customers)

### 10.3 Scaling Costs

**At 1,000 customers:**
- **Dify Cloud Enterprise:** $499/month (50K messages, unlimited docs)
- **Cloudflare Pages:** $5/month (still in free tier likely)
- **Total:** $504/month = **$0.50/customer/month**

**At 10,000 customers:**
- **Dify Cloud Enterprise:** $499/month (should still fit)
- **Additional message packs:** $100/month (500K messages = 50/customer/month)
- **Total:** $599/month = **$0.06/customer/month**

**Conclusion:** Costs decrease per customer as you scale (economies of scale).

### 10.4 Cost Comparison (vs Alternatives)

| Solution | Development | Monthly (100 users) | Monthly (1K users) | Notes |
|----------|-------------|---------------------|-------------------|-------|
| **Custom Frontend + Dify** | $86K | $64 | $504 | Recommended ✅ |
| **Dify-Only (250px images)** | $48K | $59 | $499 | Poor UX ❌ |
| **Custom Full-Stack (no Dify)** | $150K | $300 | $800 | Expensive ❌ |
| **Flowise + Custom Frontend** | $90K | $50 | $400 | Less mature ⚠️ |

**Winner:** Custom Frontend + Dify (best UX, reasonable cost)

---

## 11. Implementation Timeline

### 11.1 Phase 1: Dify Backend Setup (Days 1-3)

**Day 1:**
- Sign up for Dify Cloud Professional
- Create app: "Social Content Generator"
- Configure OpenAI API key (GPT-4 Turbo)
- Test basic chatflow (Hello World)

**Day 2:**
- Upload 4 knowledge bases (15+50+100+200 = 365 documents)
- Configure embedding model (text-embedding-3-large)
- Test knowledge retrieval (query each KB)

**Day 3:**
- Build Workflow 1 (Main Chatflow)
- Build Workflow 2 (Skill Matcher)
- Test conversation flow (user → intent classification → KB query)

**Deliverables:**
- ✅ Dify app live
- ✅ 4 KBs indexed
- ✅ 2 workflows functional

### 11.2 Phase 2: Custom Frontend Development (Days 4-12)

**Days 4-5: React Setup & Basic Chat UI**
- Initialize Vite + React + TypeScript project
- Install dependencies (Tailwind, Zustand, Axios, etc.)
- Build chat interface (message list, input box, send button)
- Test with mock data

**Days 6-7: Media Components**
- Build VideoPlayer component (react-player)
- Build ImageGallery component (react-image-gallery)
- Build Carousel component
- Build GifPlayer component
- Test each component with sample media

**Days 8-9: Option Card System**
- Build OptionCard component (combines media + text + scores)
- Build option selector logic (user types A/B/C)
- Implement conversation state management (Zustand)
- Test option selection flow

**Days 10-11: Dify API Integration**
- Connect to Dify API (sendMessage, parseResponse)
- Handle JSON responses (options structure)
- Handle text responses (general conversation)
- Error handling (API failures, timeouts)

**Day 12: Mobile Optimization**
- Responsive design (test on iPhone, Android)
- Touch gestures (swipe carousel)
- Video touch controls
- Performance optimization (lazy loading, code splitting)

**Deliverables:**
- ✅ React app with chat UI
- ✅ All media components working
- ✅ Connected to Dify API
- ✅ Mobile-responsive

### 11.3 Phase 3: Dify Workflows (Days 13-17)

**Days 13-14: Content Generation Workflow**
- Build Workflow 3 (Content Generator)
- Configure HTTP tools for 11 Stage D microservices
- Test candidate generation (3 options with media)
- Validate guards engine integration

**Days 15-16: Publishing Workflow**
- Build Workflow 4 (Publishing)
- Configure HTTP tools for 4 Stage E platform adapters
- Test LinkedIn publishing
- Test multi-platform publishing (parallel)

**Day 17: Learning & Analytics Workflow**
- Build Workflow 5 (Learning/Analytics)
- Connect to Stage F analytics service
- Test insights generation
- Test feedback loop (learning engine)

**Deliverables:**
- ✅ All 5 workflows complete
- ✅ End-to-end flow working (request → generate → publish)
- ✅ Analytics integration

### 11.4 Phase 4: Testing & Refinement (Days 18-21)

**Day 18: Cross-Browser Testing**
- Test on Chrome, Safari, Firefox, Edge
- Test on iOS Safari, Android Chrome
- Fix browser-specific issues (video playback, CSS quirks)

**Day 19: Media Testing**
- Test all media types (image, video, GIF, carousel)
- Test all platforms (LinkedIn, Twitter, Facebook, Instagram)
- Test edge cases (large videos, slow network, portrait mode)

**Day 20: Performance Testing**
- Measure page load time (target <2s)
- Measure video load time (target <3s to start playback)
- Optimize bundle size (code splitting, lazy loading)
- Test on slow 3G (throttle network in DevTools)

**Day 21: User Acceptance Testing**
- Internal team testing (5 users)
- Collect feedback on UX
- Fix critical bugs
- Polish animations and transitions

**Deliverables:**
- ✅ All browsers working
- ✅ All media types tested
- ✅ Performance targets met
- ✅ UAT feedback incorporated

### 11.5 Phase 5: Production Deployment (Days 22-23)

**Day 22: Frontend Deployment**
- Build production bundle (Vite build)
- Deploy to Cloudflare Pages
- Configure custom domain (app.ndssocial.com)
- Set up SSL certificate (automatic with Cloudflare)
- Test production environment

**Day 23: Final Polish & Go-Live**
- Final smoke test (end-to-end flow)
- Monitor error logs (Sentry)
- Monitor performance (Datadog)
- Create user documentation (screenshots, video tutorial)
- **Go live!**

**Deliverables:**
- ✅ Frontend deployed to production
- ✅ Dify workflows live
- ✅ Monitoring set up
- ✅ Documentation complete
- ✅ **Stage G Complete**

---

## 12. Testing Strategy

### 12.1 Unit Tests (Frontend Components)

**Framework:** Vitest + React Testing Library

**Coverage Target:** >80%

**Example Test:**
```typescript
import { render, screen, waitFor } from '@testing-library/react';
import { VideoPlayer } from '@/components/VideoPlayer';

describe('VideoPlayer', () => {
  it('renders video with controls', () => {
    render(<VideoPlayer url="https://example.com/video.mp4" controls={true} />);

    const video = screen.getByRole('video');
    expect(video).toHaveAttribute('controls');
  });

  it('calls onPlay when video starts', async () => {
    const onPlay = vi.fn();
    render(<VideoPlayer url="https://example.com/video.mp4" onPlay={onPlay} />);

    const video = screen.getByRole('video');
    video.play();

    await waitFor(() => {
      expect(onPlay).toHaveBeenCalled();
    });
  });
});
```

### 12.2 Integration Tests (Frontend ↔ Dify)

**Framework:** Playwright

**Scenarios:**
1. **User sends message → Dify responds**
   - Mock Dify API response
   - Verify message appears in chat
   - Verify conversation ID stored

2. **User requests post → Options displayed**
   - Mock Dify API response (options structure)
   - Verify 3 option cards rendered
   - Verify media (image, video, carousel) displayed

3. **User selects option → Publishing confirmed**
   - User types "B"
   - Mock Dify API response (success)
   - Verify confirmation message

**Example Test:**
```typescript
import { test, expect } from '@playwright/test';

test('user can create and publish post', async ({ page }) => {
  await page.goto('https://app.ndssocial.com');

  // Send message
  await page.fill('input[placeholder="Type your message..."]', 'Create LinkedIn post');
  await page.click('button[type="submit"]');

  // Wait for options
  await page.waitForSelector('.option-card');
  const options = await page.locator('.option-card').count();
  expect(options).toBe(3);

  // Verify media displayed
  const videoPlayer = await page.locator('video').count();
  expect(videoPlayer).toBeGreaterThan(0);

  // Select option B
  await page.fill('input[placeholder="Type your message..."]', 'B');
  await page.click('button[type="submit"]');

  // Wait for confirmation
  await page.waitForSelector('text=✅ Published');
});
```

### 12.3 E2E Tests (Full Stack)

**Framework:** Playwright + Real Dify API

**Scenarios:**
1. **Complete flow: Request → Generate → Publish → Analytics**
   - No mocks, real API calls
   - Verify all 11 Stage D services called
   - Verify 4 Stage E adapters called (if multi-platform)
   - Verify Stage F analytics tracking

2. **Error handling:**
   - Dify API timeout → User sees error message
   - MCP Gateway failure → User sees retry button
   - Media load failure → User sees placeholder

3. **Mobile flow:**
   - Test on real iOS Safari (BrowserStack)
   - Test on real Android Chrome
   - Verify touch gestures (swipe carousel)
   - Verify video fullscreen

### 12.4 Performance Tests

**Tools:** Lighthouse, WebPageTest

**Targets:**
- **Page Load:** <2s (First Contentful Paint)
- **Time to Interactive:** <3s
- **Lighthouse Score:** >90 (Performance, Accessibility, Best Practices)
- **Bundle Size:** <200KB gzipped (initial load)
- **Video Start:** <3s to first frame (1080p, 5 Mbps network)

**Example Lighthouse Test:**
```bash
lighthouse https://app.ndssocial.com \
  --preset=desktop \
  --output=html \
  --output-path=./lighthouse-report.html
```

### 12.5 Media Tests

**Test Matrix:**

| Media Type | Platform | Device | Network | Expected Result |
|------------|----------|--------|---------|-----------------|
| 1200x627 Image | LinkedIn | Desktop | Fast 4G | Loads <1s, full res |
| 1080p Video | LinkedIn | iPhone 14 | 3G | Adaptive bitrate, plays <3s |
| GIF | Twitter | Android | Slow 3G | Lazy loads, plays when visible |
| 9-image Carousel | LinkedIn | iPad | WiFi | Swipe works, fullscreen works |

**Test Procedure:**
1. Open app on device/browser
2. Request post with specific media type
3. Measure load time (DevTools Network tab)
4. Verify quality (visual inspection)
5. Test interactions (play/pause, swipe, fullscreen)

---

## 13. Deployment Plan

### 13.1 Cloudflare Pages Deployment

**Repository:** GitHub (private repo)

**Build Command:**
```bash
npm run build
```

**Output Directory:** `dist`

**Environment Variables:**
```bash
VITE_DIFY_API_KEY=sk-xxxx
VITE_DIFY_API_URL=https://api.dify.ai/v1
VITE_MCP_GATEWAY_URL=https://mcp-gateway.workers.dev
VITE_CLOUDFLARE_R2_PUBLIC_URL=https://pub-xxxx.r2.dev
```

**Cloudflare Pages Setup:**
1. Log in to Cloudflare dashboard
2. Pages → Create project
3. Connect to GitHub repo
4. Configure build settings:
   - Framework preset: Vite
   - Build command: `npm run build`
   - Build output directory: `dist`
5. Add environment variables
6. Deploy

**Custom Domain:**
- Domain: `app.ndssocial.com`
- DNS: CNAME to `project-name.pages.dev`
- SSL: Automatic (Cloudflare Universal SSL)

**Preview Deployments:**
- Every git push to branch creates preview URL
- Example: `https://abc123.project-name.pages.dev`
- Perfect for testing before merging to main

### 13.2 Dify Backend Deployment

**Dify Cloud (Managed):**
- Already deployed (SaaS)
- No manual deployment needed
- Workflows auto-save
- Knowledge bases auto-indexed

**Self-Hosted (Alternative, not recommended):**
If you ever need self-hosted Dify:
1. Clone Dify repo: `git clone https://github.com/langgenius/dify.git`
2. Configure `.env` (PostgreSQL, Redis, OpenAI API key)
3. `docker-compose up -d`
4. Migrate database: `docker exec -it dify-api flask db upgrade`
5. Access at `http://localhost:3000`

**Why Cloud is Better:**
- No DevOps overhead
- Auto-scaling
- Automatic backups
- 99.9% uptime SLA
- Professional support

### 13.3 Monitoring & Observability

**Frontend Monitoring (Sentry):**
```typescript
import * as Sentry from '@sentry/react';

Sentry.init({
  dsn: 'https://xxxx@sentry.io/xxxx',
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1, // 10% of transactions
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0
});
```

**Performance Monitoring (Datadog):**
```typescript
import { datadogRum } from '@datadog/browser-rum';

datadogRum.init({
  applicationId: 'xxxx',
  clientToken: 'pub_xxxx',
  site: 'datadoghq.com',
  service: 'nds-social-frontend',
  env: process.env.NODE_ENV,
  sessionSampleRate: 100,
  sessionReplaySampleRate: 20,
  trackUserInteractions: true,
  trackResources: true,
  trackLongTasks: true
});
```

**Alerts:**
- Error rate >1% → Slack notification
- Page load time >5s → Email alert
- Video playback failure >10% → PagerDuty alert
- Dify API downtime → Immediate notification

### 13.4 Rollback Plan

**Frontend Rollback:**
- Cloudflare Pages keeps 30 days of deployments
- One-click rollback to any previous deployment
- Zero downtime (instant switchover)

**Dify Workflow Rollback:**
- Dify keeps version history (50 versions)
- Click "Restore" on any previous version
- Workflows update instantly

**Emergency Rollback:**
1. Identify issue (Sentry error spike, user reports)
2. Check Cloudflare Pages deployments list
3. Click "Rollback" on last working version
4. Verify fix (check error rate drops)
5. Investigate root cause offline

---

## 14. Success Metrics

### 14.1 Technical Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Page Load Time (FCP) | <2s | Lighthouse, RUM |
| Time to Interactive | <3s | Lighthouse |
| Video Start Time | <3s | Custom tracking |
| API Response Time (p95) | <500ms | Datadog APM |
| Error Rate | <0.5% | Sentry |
| Uptime | >99.5% | Pingdom |
| Mobile Performance Score | >85 | Lighthouse Mobile |

### 14.2 User Experience Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Option Selection Rate | >90% | Analytics (% of users who select A/B/C) |
| Publish Completion Rate | >80% | Analytics (% who complete publish flow) |
| Time to First Publish | <5 min | User timing |
| Mobile vs Desktop Usage | 60/40 | Google Analytics |
| Video Play Rate | >70% | Custom tracking (% of videos played) |

### 14.3 Business Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| User Satisfaction (NPS) | >50 | In-app survey |
| Daily Active Users | 70% | Analytics |
| Posts Created per User/Week | 5 | Database query |
| Multi-Platform Publish Rate | 50% | Analytics (% publishing to 2+ platforms) |

---

## 15. Appendix

### 15.1 Technology Stack Summary

**Frontend:**
- React 18.3 + TypeScript 5.3
- Vite 5.0 (build tool)
- Tailwind CSS 3.4 (styling)
- Zustand 4.4 (state management)
- Axios 1.6 (HTTP client)
- react-player 2.13 (video)
- react-image-gallery 1.3 (carousels)
- framer-motion 11.0 (animations)
- shadcn/ui (UI components)

**Backend:**
- Dify Cloud Professional (workflow orchestration)
- OpenAI GPT-4 Turbo (LLM)
- OpenAI text-embedding-3-large (embeddings)
- PostgreSQL (Dify managed)
- Redis (Dify managed)

**Infrastructure:**
- Cloudflare Pages (frontend hosting)
- Cloudflare Workers (MCP Gateway - Stage D/E/F services)
- Cloudflare R2 (media storage)
- Cloudflare D1 (database - Stage D/E/F)

**Monitoring:**
- Sentry (error tracking)
- Datadog (performance monitoring)
- Google Analytics (user behavior)

### 15.2 Key Files & Directories

```
frontend/
├── src/
│   ├── components/
│   │   ├── Chat/
│   │   │   ├── ChatMessage.tsx
│   │   │   ├── ChatInput.tsx
│   │   │   └── ChatContainer.tsx
│   │   ├── Media/
│   │   │   ├── VideoPlayer.tsx
│   │   │   ├── ImageGallery.tsx
│   │   │   ├── Carousel.tsx
│   │   │   └── GifPlayer.tsx
│   │   ├── Options/
│   │   │   ├── OptionCard.tsx
│   │   │   └── OptionSelector.tsx
│   │   └── UI/
│   │       └── (shadcn/ui components)
│   ├── lib/
│   │   ├── dify-api.ts (Dify API client)
│   │   ├── media-validator.ts
│   │   └── utils.ts
│   ├── stores/
│   │   └── chat-store.ts (Zustand store)
│   ├── types/
│   │   ├── dify.ts
│   │   ├── media.ts
│   │   └── option.ts
│   ├── App.tsx
│   └── main.tsx
├── public/
├── package.json
├── vite.config.ts
└── tailwind.config.ts
```

### 15.3 External Dependencies

**NPM Packages:**
```json
{
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "axios": "^1.6.0",
    "zustand": "^4.4.0",
    "react-player": "^2.13.0",
    "react-image-gallery": "^1.3.0",
    "framer-motion": "^11.0.0",
    "date-fns": "^3.0.0",
    "clsx": "^2.0.0",
    "tailwind-merge": "^2.0.0",
    "hls.js": "^1.5.0"
  },
  "devDependencies": {
    "typescript": "^5.3.0",
    "vite": "^5.0.0",
    "@vitejs/plugin-react": "^4.2.0",
    "tailwindcss": "^3.4.0",
    "vitest": "^1.0.0",
    "@testing-library/react": "^14.0.0",
    "playwright": "^1.40.0"
  }
}
```

### 15.4 Browser Support

**Officially Supported:**
- Chrome 90+ (desktop & mobile)
- Safari 15+ (desktop & mobile)
- Firefox 88+ (desktop & mobile)
- Edge 90+ (desktop)

**Best Effort:**
- Chrome 80-89
- Safari 13-14
- Firefox 78-87

**Not Supported:**
- Internet Explorer (all versions)
- Safari <13
- Chrome <80

---

## 16. Conclusion

**Stage G Approved Plan Complete.**

This plan delivers a **production-ready, media-rich user interface** that combines:
- **Dify's powerful workflow orchestration** (backend)
- **Custom React frontend** (perfect media rendering)
- **Full video, image, GIF, carousel support** (no compromises)
- **Mobile-first design** (60% of users on mobile)
- **Conversational advisory pattern** (text-based A/B/C options)

**No Trade-Offs:**
- ✅ Full-resolution images (1200x627px LinkedIn, 1080x1080px Instagram)
- ✅ 1080p video playback with controls
- ✅ Smooth GIF playback
- ✅ 9-image carousels (LinkedIn)
- ✅ Mobile touch gestures
- ✅ Fast load times (<2s)
- ✅ Production-ready UX

**Timeline:** 23 days (4.6 weeks)
**Cost:** $86,250 development + $64/month operational
**Per Customer:** $0.64/customer/month (100 customers)

**Ready for Stage H (OpenManus Integration) after Stage G implementation.**

---

**Document Status**: ✅ APPROVED
**Version**: 1.0.0
**Last Updated**: November 10, 2025
**Next Review**: After Stage G implementation complete
**Maintained By**: Planning Agent

---

**Stage G Planning Complete!** 🚀
