# Product Requirements Document: AI-Powered Meme Generator

**Version:** 1.0  
**Last Updated:** February 16, 2026  
**Status:** Draft  
**Owner:** Product Team

---

## Executive Summary

An AI-powered meme generator that uses computer vision and LLMs to analyze uploaded images and generate contextual, funny text overlays in classic meme format. Unlike template-first competitors, we enable users to turn *any* personal photo into a meme with AI-generated captions that understand the context.

**Key Differentiator:** Iterative AI-powered humor refinement — users get multiple caption options and can guide the AI's tone and style until the joke lands perfectly.

---

## Problem Statement

### User Pain Points

Current meme generators force users into one of two unsatisfying workflows:

1. **Template-first tools** (Imgflip, Mematic) require finding a pre-existing template that matches your idea, limiting creativity and making personal photos nearly impossible to meme-ify.

2. **Manual caption tools** (Canva, Kapwing) provide blank canvases but offer no creative assistance — users must come up with funny text themselves.

**The Gap:** No tool currently analyzes a user's uploaded image and generates contextually relevant, funny captions that understand what's actually in the photo.

### Market Validation

From our market research:
- **Pain point #1:** "Templates aren't contextual to personal photos"
- **Pain point #2:** "Too many steps to create and share"
- **Pain point #3:** Users are skeptical of AI humor feeling too polished or "corporate"

**Our Solution:** Upload any photo → AI understands context → generates multiple caption options → user iterates until it's funny → instant meme.

---

## Target Users

### Primary: Gen Z Casual Meme Creators (18-24)
- Create memes of personal photos (friends, pets, selfies, everyday moments)
- Share in private group chats (iMessage, WhatsApp, Discord)
- Want speed and convenience over professional polish
- Value: "Make my photo funny instantly"

### Secondary: Young Millennials (25-34)
- Similar use cases but slightly more polished output expectations
- More likely to share publicly (Instagram, Twitter/X)
- Willing to iterate for the perfect joke

### Future: B2B Social Media Managers
- Need brand-safe, on-brand reactive content
- Value speed-to-post for trending topics
- *Out of scope for V1*

---

## Competitive Landscape

### Template-First Competitors
- **Imgflip** (Web, $14.95/mo): Largest meme template library, but no AI, no personal photo support
- **Mematic** (Mobile, 10M+ downloads): Classic mobile creator, manual text entry only
- **Canva** (260M users): Meme templates exist but buried in broader design tool

### AI-Powered Competitors
- **Supermeme.ai** (1M+ signups): Text-to-meme generation (creates images from prompts), does not analyze user photos
- **Google Photos AI Meme Generator** (in development): Unknown quality, likely template-based

### Our Competitive Advantage
1. **Photo-first, not template-first**: Upload any image
2. **Vision-powered context understanding**: AI analyzes the actual content
3. **Iterative humor refinement**: Multiple options + user-guided tone control
4. **Speed**: Optimized for fast iteration and sharing

---

## Core Features (V1 MVP)

### 1. Image Upload
**User Story:** As a user, I want to upload any photo so the AI can make it funny.

**Requirements:**
- Drag-and-drop or click-to-upload interface
- Support common image formats: JPG, PNG, GIF (static frame), WebP
- Max file size: 10MB
- Image preview before generation
- Client-side image validation

**Technical Notes:**
- Store uploaded images in memory (no database persistence)
- Clear memory when user closes browser or navigates away

---

### 2. AI Caption Generation
**User Story:** As a user, I want the AI to analyze my photo and suggest funny captions that make sense in context.

**Requirements:**
- Send image to OpenAI Vision API for analysis
- Generate **3 distinct caption options** per request:
  - Top text + bottom text (classic meme format)
  - Each option should have different comedic angles
- Display options as selectable cards
- Show loading state during generation (5-10 second estimate)

**Caption Guidelines:**
- Internet-native humor (not corporate/polished)
- Contextually relevant to image content
- Short, punchy text (ideally 5-15 words per line)
- Avoid offensive content (OpenAI safety guidelines)

**Technical Notes:**
- Use GPT-4 Vision API
- Prompt engineering to ensure:
  - Classic top/bottom format
  - Variety in comedic style across options
  - Contextual relevance to image

---

### 3. Caption Refinement (Iterative Humor)
**User Story:** As a user, I want to refine the AI's suggestions if they don't land, so I get a meme that's actually funny.

**Requirements:**
- "Generate more options" button (generates 3 new captions)
- Tone controls:
  - Wholesome
  - Sarcastic
  - Absurd/Surreal
  - Roast/Savage (brand-unsafe)
- Text prompt field: "Make it more like _____" (free text refinement)
- Track generation history (show previous options in case user wants to go back)

**Technical Notes:**
- Include user's selected tone in system prompt
- If user provides refinement text, include in prompt: "Previous attempts didn't work. User wants: {refinement_text}"
- Limit: 10 generations per image (cost control)

---

### 4. Text Overlay & Preview
**User Story:** As a user, I want to see the meme with captions overlaid before downloading.

**Requirements:**
- Classic Impact font (or closest web-safe equivalent)
- White text with black stroke/outline
- Top text: centered, near top edge
- Bottom text: centered, near bottom edge
- Automatic text wrapping (max 2 lines per text block)
- Real-time preview as user selects caption options

**Out of Scope (V1):**
- Manual text editing
- Font selection
- Text positioning
- Color customization

**Technical Notes:**
- Render using HTML Canvas (client-side)
- Text sizing: responsive to image dimensions
- Ensure text is readable against varied backgrounds

---

### 5. Download/Export
**User Story:** As a user, I want to download my finished meme to share immediately.

**Requirements:**
- "Download Meme" button
- Export as PNG (high quality, no compression artifacts)
- Filename format: `meme-{timestamp}.png`
- One-click download (no additional steps)

**Out of Scope (V1):**
- Direct social media sharing integrations
- Multiple format exports (MP4, GIF)
- Dimension presets (Instagram, TikTok)

**Technical Notes:**
- Use Canvas.toBlob() for export
- Maintain original image aspect ratio
- Minimum export resolution: 800px width

---

## User Flow

### Happy Path
1. **Landing page** → "Upload Image" CTA
2. **Upload interface** → User drags/selects photo
3. **Loading state** → "AI is analyzing your image..." (5-10s)
4. **Caption options** → 3 cards displayed with preview thumbnails
5. **Select caption** → Live preview updates with text overlay
6. **Iterate (optional)**:
   - User clicks "Generate more options" → new 3 cards appear
   - User selects tone filter → regenerates with new tone
   - User enters refinement prompt → regenerates with guidance
7. **Download** → User clicks "Download Meme" → PNG saved locally
8. **New meme** → "Make Another" button returns to upload

---

## Technical Architecture

### Tech Stack
- **Frontend:** React (Vite)
- **Backend:** Node.js + Express
- **AI:** OpenAI GPT-4 Vision API
- **Image Processing:** HTML Canvas (client-side rendering)
- **Hosting:** TBD (Vercel/Netlify for frontend, separate backend deploy)

### Data Storage (V1)
- **In-memory only**: No database, no user accounts
- Uploaded images stored in browser memory
- Generated captions stored in React state
- All data cleared on page refresh/close

### API Flow
```
Client                    Backend                   OpenAI
  |                          |                         |
  |--1. Upload image-------->|                         |
  |                          |--2. Send to Vision API->|
  |                          |<-3. Image analysis------|
  |                          |--4. Generate captions-->|
  |                          |<-5. Caption options-----|
  |<-6. Return 3 captions----|                         |
  |                          |                         |
  |--7. Refine request------>|                         |
  |  (tone/prompt)           |--8. Refined prompt----->|
  |                          |<-9. New captions--------|
  |<-10. Return 3 captions---|                         |
```

### Cost Considerations
- GPT-4 Vision: ~$0.01-0.03 per image analysis + caption generation
- Rate limiting: 10 generations per image
- No free tier in V1 (build monetization in V2)

---

## Success Metrics

### Primary (Product-Market Fit)
- **Organic sharing rate**: % of generated memes shared outside the app
- **Iteration rate**: Average # of caption generations before download
- **Completion rate**: % of users who upload → download

### Secondary (Engagement)
- **Multi-meme creators**: % of users who create 2+ memes in one session
- **Return users**: % of users who return within 7 days (requires analytics, not accounts)

### Quality (AI Performance)
- **First-pick rate**: % of users who select one of the first 3 caption options
- **Refinement usage**: % of users who use tone controls or text prompts
- **Generation limit hits**: % of users who hit 10-generation cap

---

## Out of Scope (V1)

### Features for Future Versions
- User accounts & saved meme history
- Database persistence
- Video/GIF meme support
- Manual text editing
- Custom fonts/colors/positioning
- Social media integrations (direct posting)
- Meme templates library
- Community/gallery features
- Mobile app (native iOS/Android)
- B2B team plans & branding controls
- API access

### Explicitly NOT Building (V1)
- Image generation (we only caption uploaded images)
- Image filters/effects
- Multi-image memes (side-by-side formats)
- Animated text
- Stickers/emoji overlays

---

## Open Questions & Risks

### Technical Risks
1. **OpenAI API latency**: 5-10 second wait times may feel slow
   - *Mitigation*: Progressive loading UX, set expectations upfront
   
2. **AI humor quality**: Captions may not be funny consistently
   - *Mitigation*: Iterative refinement allows users to guide AI
   
3. **Cost per generation**: $0.01-0.03 per image is not sustainable at scale
   - *Mitigation*: Add monetization in V2, rate limiting in V1

### Product Risks
1. **User expectations**: Users may expect instant results
   - *Mitigation*: Clear loading messages, preview thumbnails
   
2. **"AI doesn't get my humor"**: Users may find captions too safe/generic
   - *Mitigation*: Tone controls + refinement prompts give user control

### Open Questions
- [ ] What's the maximum acceptable latency? (need user testing)
- [ ] Should we show confidence scores for captions? (e.g., "AI thinks this is the funniest")
- [ ] Should we allow manual text override? (breaks simplicity but adds control)
- [ ] What analytics can we collect without user accounts?

---

## Launch Plan

### Phase 1: Internal Alpha (Week 1-2)
- Build core flow: upload → generate → preview → download
- Test with team's personal photos
- Validate AI caption quality
- Iterate on prompt engineering

### Phase 2: Closed Beta (Week 3-4)
- Invite 20-30 Gen Z users (friends, Discord communities)
- Collect feedback on:
  - Caption quality & humor
  - Iteration workflow (too many steps? too few options?)
  - UX clarity
- Track success metrics

### Phase 3: Public Launch (Week 5+)
- Launch on Product Hunt, Reddit (r/memes, r/InternetIsBeautiful)
- Monitor for viral sharing in group chats
- Collect analytics on usage patterns
- Plan V2 features based on data

---

## Appendix: Competitive Pricing Reference

*(From market research)*

| Tool          | Pricing        | Audience   | Notes                     |
|---------------|----------------|------------|---------------------------|
| Imgflip Pro   | $14.95/mo      | 18-24      | No watermark, API access  |
| Kapwing Pro   | $16/mo         | 18-24      | Team collaboration        |
| Canva Pro     | ~$13/mo        | 18-35      | Full design suite         |
| Supermeme.ai  | Unknown        | Unknown    | AI text-to-meme           |

**Suggested V2 Pricing:** $9.99-$14.99/mo for unlimited generations, no watermark

---

## Approval Sign-Off

- [ ] Product Lead
- [ ] Engineering Lead
- [ ] Design Lead

**Next Steps:** Kickoff meeting, design mockups, technical spike on OpenAI Vision API integration.
