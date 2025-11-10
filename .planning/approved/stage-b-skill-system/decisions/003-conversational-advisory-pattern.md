# Decision 003: Conversational Advisory Pattern

**Date**: 2025-11-06
**Status**: Approved
**Deciders**: Product & UX Team
**Related**: Architecture Analysis Complete

---

## Context

DIFY-AGENT needs a user interface for multi-turn conversations about social media content creation. Users need to:
- Request content generation
- Provide input and preferences
- Review generated content
- Make decisions (approve, edit, regenerate)
- Choose between options

**Question**: What interaction model provides the best user experience?

---

## Options Considered

### Option 1: GUI with Buttons and Forms
**Description**: Traditional web UI with clickable buttons, forms, dropdowns

**Example**:
```
┌─────────────────────────────────────────┐
│  Create Social Media Post               │
├─────────────────────────────────────────┤
│  Product: [SKU-001 Product B  ▼]        │
│  Platform: [LinkedIn         ▼]        │
│  Angle: [✓] Strength  [ ] Price        │
│         [ ] Availability                │
│  [Generate Content]  [Cancel]          │
└─────────────────────────────────────────┘
```

**Pros**:
- Familiar to users
- Visual affordances clear
- Easy to validate inputs
- Structured data collection

**Cons**:
- Requires extensive UI development
- Not mobile-friendly
- Doesn't handle complexity well
- Hard to pause mid-workflow
- Rigid interaction flow
- High development cost

**Cost**: $30k-50k for comprehensive UI

### Option 2: Chatbot with Quick Reply Buttons
**Description**: Chat interface with clickable button suggestions

**Example**:
```
Bot: "Which product should I feature?"
    [SKU-001 Product B] [SKU-002 Product A] [SKU-003 Product C]

You: [SKU-001 Product B] ← clicked button

Bot: "Which angle?"
    [Strength] [Price] [Availability]
```

**Pros**:
- Conversational feel
- Mobile-friendly
- Guided choices
- Familiar pattern (WhatsApp, Messenger)

**Cons**:
- Still requires button UI development
- Limited to predefined options
- Can't handle free-form input easily
- Button layout on mobile difficult
- Complexity increases with options

**Cost**: $15k-25k for chat UI + buttons

### Option 3: Pure Text Conversational (Advisory Pattern) ✅ SELECTED
**Description**: Natural language conversation, options presented as text (A/B/C)

**Example**:
```
You: "Create a LinkedIn post about our SKU-001 product"

Bot: "I found SKU-001 Product B (Grade 500) in stock at £2.85/unit.
     I can create a post about:
     A) Strength & durability for industry applications
     B) Price competitiveness vs competitors
     C) Fast delivery & availability

     Which angle works best for your audience?"

You: "A"

Bot: "Great! Generating content focused on strength...

     [Draft content shown]

     Does this match your brand voice?
     A) Perfect - publish now
     B) Good - minor edits needed
     C) Not quite - try different angle"

You: "B - make it more technical"

Bot: "Adding technical details... Here's the revised version:

     [Updated content shown]

     Better? A) Yes, publish  B) More changes  C) Start over"
```

**Pros**:
- Zero UI development required
- Works on any platform (web, mobile, API, WhatsApp, Slack)
- Natural language interaction
- Handles complexity well
- Easy to pause and resume
- Extremely mobile-friendly
- Can mix structured and free-form
- Fast development (use existing chat)

**Cons**:
- Requires clear option labeling
- Users must type A/B/C (but simple)
- May need typing indicators
- Less visual than buttons

**Cost**: $2k-5k (minimal chat interface)

### Option 4: Voice Interface
**Description**: Speak requests, hear responses, make choices verbally

**Pros**:
- Hands-free operation
- Natural interaction
- Accessibility benefits

**Cons**:
- High technical complexity
- Transcription costs
- Privacy concerns
- Not suitable for all environments
- Hard to review written content via voice

**Cost**: $40k+ for voice interface

---

## Decision

**We choose Option 3: Pure Text Conversational (Advisory Pattern)**

### Rationale

1. **Fastest Time to Market**
   - No UI development required
   - Use existing chat platforms
   - Can start with simple text interface
   - Can upgrade to richer UI later

2. **Platform Agnostic**
   - Works on web (Dify UI)
   - Works on mobile (responsive)
   - Works in WhatsApp, Telegram, Slack
   - Works via API
   - Works anywhere there's text input

3. **Better for Complex Workflows**
   - Can pause and resume easily
   - Can go back and change answers
   - Can handle branching logic
   - Can mix structured and free-form
   - Natural conversation flow

4. **Lower Development Cost**
   - 10x cheaper than GUI ($2k vs $30k)
   - Faster to iterate
   - No UI framework dependencies
   - Dify provides chat interface

5. **User Experience**
   - More natural for advisory pattern
   - User types what they want naturally
   - Bot presents options clearly
   - Simple to choose (A, B, C)
   - Easy to learn

6. **Mobile First**
   - Perfect for mobile use
   - No tiny buttons to tap
   - Natural chat interface
   - Typing is easy on mobile
   - No horizontal scrolling

---

## Conversational Advisory Pattern Principles

### Principle 1: User Initiates with Natural Language
```
✅ GOOD: "Create a post about Product Line B"
✅ GOOD: "I need content for LinkedIn"
✅ GOOD: "Make a post highlighting fast delivery"

❌ BAD: "START_WORKFLOW post_creation product=product-b platform=linkedin"
```

### Principle 2: Bot Presents Options as Labeled Text
```
✅ GOOD:
"Which angle works best?
 A) Strength & durability
 B) Price competitiveness
 C) Fast delivery"

❌ BAD:
"Choose angle: strength, price, delivery"
(unclear how to respond)

❌ BAD:
"[Strength] [Price] [Delivery]"
(requires button implementation)
```

### Principle 3: User Responds with Simple Text
```
✅ GOOD: "A"
✅ GOOD: "B - but emphasize technical specs"
✅ GOOD: "Option C"

❌ BAD: Requires clicking buttons
❌ BAD: Requires filling forms
```

### Principle 4: Bot Confirms and Progresses
```
✅ GOOD:
"Great! Focusing on strength...
 [shows content]
 Ready to publish? A) Yes  B) Edit  C) Regenerate"

❌ BAD:
"Content generated."
(no next steps, user unsure what to do)
```

### Principle 5: Always Show Context and State
```
✅ GOOD:
"Post about SKU-001 Product B, focusing on strength ✓
 Product image verified ✓
 Brand voice score: 92% ✓

 Ready to publish to LinkedIn?
 A) Yes, publish now
 B) Schedule for later
 C) Save as draft"

❌ BAD:
"Publish? Y/N"
(no context, user lost)
```

### Principle 6: Allow Free-Form Overrides
```
User can always type naturally instead of choosing:

Bot: "Which product? A) SKU-001 Product B  B) SKU-002 Product A  C) SKU-003 Product C"

User: "Actually, I want to feature our custom cutting service"
      (not in options)

Bot: "Got it! Tell me more about the custom cutting service..."
      (adapts to unexpected input)
```

---

## Consequences

### Positive
- ✅ 90% faster development (weeks vs months)
- ✅ 95% lower development cost ($2k vs $30k)
- ✅ Works on all platforms (web, mobile, API)
- ✅ Better for complex workflows
- ✅ Natural user experience
- ✅ Easy to iterate and improve
- ✅ Mobile-first by design
- ✅ Accessible (screen readers work great)

### Negative
- ❌ Users must type A/B/C (minor friction)
- ❌ Less visual than GUI (can add images though)
- ❌ Requires clear writing (bot responses)
- ❌ May seem less "professional" to some
- ❌ Need good error handling for unclear inputs

### Neutral
- ⚪ Different paradigm (but users adapt quickly)
- ⚪ Can add rich media (images, videos) in responses
- ⚪ Can upgrade to buttons later if needed
- ⚪ Learning curve minimal (everyone texts)

---

## Implementation Guidelines

### Response Formatting

**Present Options Clearly**:
```
✅ GOOD - Numbered with descriptions:
"Which platform?
 A) LinkedIn - Professional B2B audience
 B) Instagram - Visual industry showcase
 C) Facebook - Broad local reach"

✅ GOOD - Simple list:
"Choose:
 A) Publish now
 B) Schedule
 C) Save draft"

❌ BAD - Unclear format:
"Options: publish, schedule, draft"
```

**Show Progress and Context**:
```
✅ GOOD:
"Creating post (Step 2 of 4)
 Product: SKU-001 Product B ✓
 Angle: Strength ✓
 Platform: LinkedIn (in progress)

 Content generated with 92% brand voice match:
 [content preview]

 Next step - verify product image..."

❌ BAD:
"Step 2. Continue? Y/N"
```

**Handle Ambiguity Gracefully**:
```
User: "Create a post"

✅ GOOD:
"I'd love to help! A few quick questions:
 1. Which product should I feature?
    A) Most popular (SKU-001 Product B)
    B) Best margin (SKU-002 Product A)
    C) You tell me

 2. Which platform?
    A) LinkedIn
    B) Instagram
    C) Multiple platforms"

❌ BAD:
"Error: Missing parameters. Specify product and platform."
```

### Error Handling

**Unclear Input**:
```
Bot: "Which product? A) SKU-001  B) SKU-002  C) SKU-003"
User: "Maybe Product Line B?"

✅ GOOD:
"I see you're interested in Product Line B! We have:
 A) SKU-001 (most popular)
 B) SKU-004 (heavy duty)
 Which one works better?"

❌ BAD:
"Invalid input. Please choose A, B, or C."
```

**Out of Context**:
```
[Mid-workflow creating a post]
User: "What's the price of Product Line C?"

✅ GOOD:
"Quick answer: Product Line C starts at £1.50/sqm

 Want to:
 A) Continue creating the current post
 B) Start new post about Product Line C
 C) Just get pricing info"

❌ BAD:
"Please stay on topic."
```

### Multi-Turn Conversation State

**Always Show Where We Are**:
```
Session context preserved across turns:

Turn 1:
You: "Create LinkedIn post"
Bot: "Which product? A) SKU-001  B) SKU-002  C) SKU-003"

Turn 2:
You: "A"
Bot: "SKU-001 Product B selected ✓
     Which angle? A) Strength  B) Price  C) Delivery"

Turn 3:
You: "Actually, change product to Product Line C"
Bot: "No problem! Switching to Product Line C ✓
     Same angle options:
     A) Strength  B) Price  C) Delivery"
```

### Mobile Optimization

**Keep Options Short**:
```
✅ GOOD (mobile-friendly):
"Platform?
 A) LinkedIn
 B) Instagram
 C) Facebook"

❌ BAD (too verbose on mobile):
"Which social media platform would you like to publish this
 content to? Please choose from the following options:
 A) LinkedIn - Professional Business Network
 B) Instagram - Photo and Video Sharing Platform
 C) Facebook - General Social Networking Service"
```

**Use Emojis for Quick Scanning**:
```
✅ GOOD:
"Next step?
 A) ✅ Publish now
 B) 📅 Schedule
 C) 💾 Save draft"

But don't overdo it - keep professional.
```

---

## User Experience Flow Examples

### Example 1: Simple Post Creation
```
You: "Create a post"

Bot: "I'd love to help! What should I feature?
     A) SKU-001 Product B (in stock, trending)
     B) SKU-002 Product A (low stock, high margin)
     C) Something else"

You: "A"

Bot: "Great! SKU-001 Product B it is.

     Which platform?
     A) LinkedIn (best for B2B)
     B) Instagram (visual impact)
     C) Both"

You: "A"

Bot: "Perfect. Generating LinkedIn post about SKU-001 Product B...

     [Content shown with image]
     Brand voice: 94% match ✓

     What do you think?
     A) Perfect - publish now
     B) Good - minor edits
     C) Try different angle"

You: "A"

Bot: "Publishing to LinkedIn... Done! ✓

     Post live at: [link]
     Want to create another? Y/N"
```

### Example 2: Complex Multi-Turn with Edits
```
You: "I need a technical post for engineers about Product Line B strength"

Bot: "Got it - technical angle for engineers!

     Which Product Line B variant?
     A) SKU-001 (most common)
     B) SKU-004 (heavy duty)
     C) Custom specification"

You: "B"

Bot: "SKU-004 Product Line B selected. Checking product details...

     ✓ In stock: 50 tonnes
     ✓ Price: £3.20/metre
     ✓ Image: verified

     Creating technical content for engineers...

     [Draft shown]

     This focuses on:
     - Grade 500 tensile strength
     - Seismic performance
     - Code compliance BS 4449

     Brand voice: 88% match
     Technical depth: High ✓

     How's this?
     A) Perfect
     B) Too technical
     C) Not technical enough
     D) Different focus needed"

You: "C - add more about stress calculations"

Bot: "Adding stress calculations and load tables...

     [Updated draft with calculations]

     Better? Now includes:
     - Tensile strength formulas
     - Load-bearing calculations
     - Safety factors
     - Comparative analysis

     Brand voice: 92% match
     Technical depth: Very High ✓

     Ready to publish?
     A) Yes
     B) More changes
     C) Show to colleague first"

You: "C"

Bot: "No problem! I've saved this as draft: 'SKU-004-technical-v2'

     Share link: [shareable link]

     What next?
     A) Create another post
     B) Edit existing draft
     C) I'm done for now"
```

### Example 3: Error Recovery
```
You: "Make a post"

Bot: "Happy to help! Which product interests you?
     A) SKU-001 Product B
     B) SKU-002 Product A
     C) SKU-003 Product C"

You: "Yes"  ← Unclear response

Bot: "I didn't quite catch that. Could you:
     - Type A, B, or C to choose from the list
     - Or just tell me which product you want to feature

     Which works for you?"

You: "The Product C one"

Bot: "Perfect! Product Line C it is. ✓

     We have several types:
     A) Standard variant (most popular)
     B) Fine variant (precision work)
     C) Heavy duty variant

     Which type?"
```

---

## Technical Implementation

### Dify Configuration

Use Dify's built-in chatflow:
1. User message → Intent classifier
2. Route to appropriate workflow
3. Present options as text
4. Parse user response (A/B/C or natural language)
5. Continue workflow
6. Preserve context in Dify memory

### Response Templates

```python
# Option presentation template
def present_options(question: str, options: list) -> str:
    """
    question: "Which product?"
    options: [
        {"key": "A", "label": "SKU-001 Product B", "desc": "Most popular"},
        {"key": "B", "label": "SKU-002 Product A", "desc": "High margin"},
        {"key": "C", "label": "SKU-003 Product C", "desc": "Fast delivery"}
    ]
    """
    response = f"{question}\n"
    for opt in options:
        response += f"{opt['key']}) {opt['label']}"
        if opt.get('desc'):
            response += f" - {opt['desc']}"
        response += "\n"
    return response

# Output:
# Which product?
# A) SKU-001 Product B - Most popular
# B) SKU-002 Product A - High margin
# C) SKU-003 Product C - Fast delivery
```

### Input Parsing

```python
def parse_user_choice(user_input: str, options: list) -> dict:
    """
    Flexible parsing of user responses
    """
    user_input = user_input.strip().lower()

    # Direct match (A, B, C)
    for opt in options:
        if user_input == opt['key'].lower():
            return opt

    # Fuzzy match on labels
    for opt in options:
        if opt['label'].lower() in user_input:
            return opt

    # Natural language match
    # Use LLM to interpret intent
    return classify_intent(user_input, options)
```

---

## Success Criteria

### User Experience
- ✅ Users can complete workflow via text only
- ✅ <3 minutes from request to first draft
- ✅ <5 total messages for simple post
- ✅ Clear what to do next at each step
- ✅ Can pause and resume easily

### Technical
- ✅ Works on mobile (responsive)
- ✅ Works on desktop
- ✅ Accessible (screen readers)
- ✅ Fast responses (<2s)
- ✅ Context preserved across turns

### Business
- ✅ 90% reduction in UI development cost
- ✅ 80% faster time to market
- ✅ Works on multiple platforms (web, mobile, API)
- ✅ Easy to iterate and improve

---

## Future Enhancements

### Phase 2 (Optional)
- Add rich media (images, videos) in bot responses
- Add inline editing (highlight text to change)
- Add voice input option (for accessibility)
- Add quick reply suggestions (but remain text-first)

### Phase 3 (If needed)
- Upgrade to GUI for complex workflows
- Add visual content builder
- Add drag-and-drop for images
- Keep text interface as primary

---

## Approval

- [ ] Product Manager: _____________________ Date: _______
- [ ] UX Lead: _____________________ Date: _______
- [ ] Engineering Lead: _____________________ Date: _______

---

**Last Updated**: 2025-11-06
**Next Review**: 2025-11-13 (after user testing)
