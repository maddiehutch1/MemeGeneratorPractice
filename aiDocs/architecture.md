# System Architecture: AI-Powered Meme Generator

**Version:** 1.1  
**Last Updated:** 2026-02-16  
**Status:** Implementation Ready  
**Implementation Plan:** `ai/roadmaps/2026-02-16-implementation-roadmap.md`

---

## Overview

A web-based meme generator that uses OpenAI's vision API to analyze uploaded images and generate contextually relevant, funny captions. Users can guide the AI's tone/style and iterate until satisfied, then download the final meme with text overlay.

**Core Flow:** Upload → AI Analysis → Caption Generation → Text Overlay → Download

---

## Tech Stack

### Frontend
- **Framework:** React 18+ with Vite
- **Image Processing:** HTML Canvas API (client-side text rendering)
- **HTTP Client:** Fetch API or Axios
- **Styling:** CSS Modules or Tailwind (TBD)
- **State Management:** React useState/useReducer (no Redux needed for MVP)

### Backend
- **Runtime:** Node.js 18+
- **Framework:** Express.js
- **AI Integration:** OpenAI Node.js SDK (`openai` npm package)
- **Image Processing:** Sharp (for future backend rendering, optional for MVP)
- **File Upload:** Multer middleware
- **Environment:** dotenv for config

### External APIs
- **OpenAI Chat Completions API**
  - Model: `gpt-4o` (vision-capable)
  - Purpose: Image analysis + caption generation
  - Input: Base64-encoded images
  - Output: JSON-structured captions

### Data Storage (MVP)
- **In-memory only** - no database
- Frontend: React state
- Backend: Request-scoped variables (with generation tracking map)
- Uploaded images cleared after generation
- **10-generation limit** per image enforced server-side

---

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                           FRONTEND (React)                       │
│  ┌───────────────┐  ┌──────────────┐  ┌──────────────────────┐ │
│  │ Upload        │→ │ Context      │→ │ Caption              │ │
│  │ Component     │  │ Controls     │  │ Options              │ │
│  └───────────────┘  └──────────────┘  └──────────────────────┘ │
│         │                   │                     │              │
│         └───────────────────┼─────────────────────┘              │
│                             ↓                                    │
│                  ┌──────────────────────┐                        │
│                  │  State Management    │                        │
│                  │  (useState)          │                        │
│                  └──────────────────────┘                        │
│                             │                                    │
│         ┌───────────────────┼─────────────────────┐              │
│         ↓                   ↓                     ↓              │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────────────┐ │
│  │ Meme        │   │ Canvas       │   │ Download             │ │
│  │ Preview     │   │ Renderer     │   │ Button               │ │
│  └─────────────┘   └──────────────┘   └──────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                             │
                             │ HTTP/JSON
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                       BACKEND (Node.js + Express)                │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  POST /api/generate-caption                                │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐ │ │
│  │  │ Multer       │→ │ Validation   │→ │ OpenAI Service  │ │ │
│  │  │ (Upload)     │  │ Middleware   │  │                 │ │ │
│  │  └──────────────┘  └──────────────┘  └─────────────────┘ │ │
│  └────────────────────────────────────────────────────────────┘ │
│                             │                                    │
│                             │ API Call                           │
│                             ↓                                    │
│                  ┌──────────────────────┐                        │
│                  │  OpenAI SDK          │                        │
│                  │  (Chat Completions)  │                        │
│                  └──────────────────────┘                        │
└─────────────────────────────────────────────────────────────────┘
                             │
                             │ HTTPS
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                      OPENAI API (External)                       │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  POST /chat/completions                                   │  │
│  │  Model: gpt-4o                                            │  │
│  │  Input: Base64 image + system prompt + user context      │  │
│  │  Output: JSON with topText/bottomText                    │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Component Architecture

### Frontend Components

```
App
├── UploadZone
│   ├── DragDropArea
│   └── FileInput
│
├── ContextControls (optional, shown after upload)
│   ├── PresetButtons (Sarcastic, Wholesome, Absurd, etc.)
│   └── CustomPromptInput
│
├── GenerateButton
│
├── CaptionOptions (shown after generation)
│   ├── CaptionCard (x3 if multiple options)
│   └── RegenerateButton
│
├── MemePreview
│   ├── CanvasRenderer
│   └── ImageDisplay
│
└── DownloadButton
```

### Backend Routes

```
/api
├── /generate-caption (POST)
│   ├── Accepts: JSON with base64 image + context
│   ├── Returns: JSON with 3 captions + metadata
│   └── Error handling with retry logic
│
└── /health (GET) - health check endpoint
```

---

## Data Flow

### 1. Image Upload Flow
1. User selects image via drag-and-drop or file input
2. Client validates type (JPG/PNG) and size (max 10MB)
3. Convert to base64 using FileReader API
4. Store in React state
5. Show preview thumbnail
6. Enable context controls and generate button

### 2. Caption Generation Flow
1. User optionally selects context (preset or custom)
2. User clicks "Generate Meme"
3. Frontend sends POST to `/api/generate-caption` with image + context
4. Backend validates request
5. Backend builds system prompt based on context type
6. Backend calls OpenAI Chat Completions API (gpt-4o)
   - High detail image analysis
   - JSON response format
   - Temperature: 0.8, Max tokens: 500
7. Backend parses JSON, validates structure
8. Backend returns 3 caption options with metadata
9. Frontend displays options as selectable cards
10. First option auto-applied to preview
11. Download button enabled

### 3. Text Overlay Rendering Flow
1. User selects caption from options
2. Get 2D canvas context
3. Load uploaded image into canvas
4. Calculate responsive dimensions:
   - Font size = image width × 0.05
   - Top text position = height × 0.05
   - Bottom text position = height × 0.90
5. Draw image to canvas
6. Configure text properties (Impact font, white fill, black stroke)
7. Apply word wrapping (max 90% width, 2 lines max)
8. Draw top and bottom text with stroke + fill
9. Update preview

### 4. Download Flow
1. User clicks "Download Meme"
2. Convert canvas to Blob using `canvas.toBlob()`
3. Create temporary download link
4. Set filename: `meme-{timestamp}.png`
5. Trigger download
6. Cleanup temporary URL

---

## API Specifications

### POST /api/generate-caption

**Request Format:**
- Method: POST
- Content-Type: application/json
- Body fields:
  - `image`: Base64-encoded image with data URL prefix (e.g., "data:image/jpeg;base64,...")
  - `context.type`: "none", "preset", or "custom"
  - `context.preset`: (if type=preset) "sarcastic", "wholesome", "absurd", or "roast"
  - `context.customPrompt`: (if type=custom) User's custom text

**Supported Presets:**
- `sarcastic` - Dry, ironic humor
- `wholesome` - Warm, positive vibes
- `absurd` - Chaotic, surreal internet humor
- `roast` - Mean but funny, no mercy

**Success Response Fields:**
- `success`: Boolean (true)
- `captions`: Array of 3 objects, each with `id`, `topText`, `bottomText`
- `tokensUsed`: Number of tokens consumed
- `cost`: Estimated cost in USD

**Error Response Fields:**
- `success`: Boolean (false)
- `error`: Error code string
- `message`: User-friendly error message

**Error Types:**
- `invalid_image` - Unsupported format or corrupted file
- `file_too_large` - Image exceeds 10MB
- `rate_limit_exceeded` - Hit 10-generation limit per image
- `openai_error` - OpenAI API failure
- `server_error` - Internal server error

---

## OpenAI Integration Details

### Prompt Engineering

**Base System Prompt (No Context):**
- Role: Funny meme caption generator
- Task: Create 3 different captions in top/bottom format
- Requirements: Different comedic angles, ALL CAPS, 5-15 words per line, internet-native humor
- Output: JSON array with topText/bottomText objects

**With Preset Style:**
- Append style description to base prompt
- Example: "Style: SARCASTIC. Dry, ironic, 'oh great, just what I needed' energy."

**With Custom Prompt:**
- Append user's creative direction to base prompt
- Example: "User's creative direction: 'make it about Monday mornings'"

**Preset Style Descriptions:**
- **Sarcastic:** "Style: SARCASTIC. Dry, ironic, 'oh great, just what I needed' energy."
- **Wholesome:** "Style: WHOLESOME. Warm, positive, 'faith in humanity restored' vibes."
- **Absurd:** "Style: ABSURD. Random, nonsensical, chaotic, weird internet humor."
- **Roast:** "Style: ROAST. Mean but funny, call-out culture, no mercy."

### API Configuration

**Model:** gpt-4o  
**Messages:**
- System message with dynamic prompt (varies by context)
- User message with base64 image (detail: "high")

**Parameters:**
- `response_format`: { type: "json_object" }
- `max_tokens`: 500 (for 3 captions)
- `temperature`: 0.8 (balanced creativity)

### Cost Estimation

**Per Image (1024×1024, detail: "high"):**
- Image tokens: ~765 tokens
- System prompt: ~150 tokens
- Response: ~50 tokens
- **Total:** ~965 tokens per generation

**Pricing (gpt-4o):**
- Input: $2.50/1M tokens
- Output: $10.00/1M tokens
- **Cost per generation:** ~$0.003 (0.3 cents)
- **With 10 regenerations:** ~$0.03 per image

---

## Canvas Text Rendering Implementation

### Algorithm Overview

1. Set canvas dimensions to match image
2. Draw base image to canvas
3. Calculate responsive font size (5% of image width)
4. Configure text style (Impact font, white fill, black stroke)
5. Word wrap text to max 90% canvas width (2 lines max)
6. Draw top text at 5% from top
7. Draw bottom text at 90% from top
8. Apply stroke then fill for outlined effect

### Text Properties

**Font:** Impact, Arial Black (fallback), sans-serif  
**Size:** Dynamic (5% of image width)  
**Fill:** White  
**Stroke:** Black (width = 5% of font size)  
**Alignment:** Center  
**Transform:** Uppercase  
**Max Width:** 90% of canvas width  
**Line Spacing:** 1.2× font size  

### Word Wrapping Strategy

- Split text by words
- Measure each word with canvas context
- Break to new line when exceeding max width
- Maximum 2 lines per text block
- If text exceeds 2 lines, truncate with ellipsis

---

## Error Handling Strategy

### Frontend Errors
- File type check (JPG, PNG only)
- File size limit (10MB)
- Image load validation
- Network timeout (30s)
- 4xx/5xx responses
- Invalid JSON response
- User-friendly error messages

### Backend Errors
- Image format validation
- Base64 decode errors
- Missing required fields
- OpenAI API errors (rate limit, timeout, invalid response)
  - Retry with exponential backoff (1 attempt)
  - Log and return appropriate status codes
- JSON structure validation
- Input sanitization

---

## Performance Considerations

### Frontend Optimization
- Canvas rendering cached for selected caption
- Use `requestAnimationFrame` for smooth preview updates
- Client-side image compression if > 5MB
- Lazy loading for components (Phase 3)

### Backend Optimization
- JSON payload (base64) instead of multipart for simplicity
- Request timeout (30s)
- In-memory generation tracking (clear old entries)
- Rate limiting: 10 generations per image (cost control)
- System prompts built dynamically (no caching needed for MVP)

### OpenAI API Optimization
- Use `detail: "low"` for cost-sensitive scenarios (85 tokens vs 765+)
- Batch multiple requests if possible (not applicable for MVP)
- Implement caching for identical image+context pairs (future)

---

## Security Considerations

### MVP Security
- Validate image file types (JPG, PNG only)
- Limit file size (10MB client-side + server-side)
- Sanitize user input (custom prompts - prevent prompt injection)
- Environment variables for API keys (never exposed to frontend)
- CORS configuration:
  - Development: `http://localhost:5173` (frontend)
  - Production: Specific domain only
- Rate limiting per image (10 generations)
- OpenAI API budget limits ($50 hard cap, $40 alert)

### Future (Production)
- Rate limiting (per user/IP)
- CAPTCHA for abuse prevention
- Content moderation (OpenAI API safety)
- HTTPS only
- Input sanitization (XSS prevention)
- API key rotation

---

## Deployment Architecture (Future)

## Deployment Architecture

### Development Environment

**Frontend:** Vite dev server (localhost:5173)  
**Backend:** Express server (localhost:3001)  
**Testing:** Manual testing (Postman for API, browser for UI)  

**Environment Variables:**

Backend `.env`:
- `OPENAI_API_KEY` - From OpenAI platform
- `PORT` - Default 3001
- `NODE_ENV` - development/production

Frontend `.env`:
- `VITE_API_URL` - Backend URL (http://localhost:3001)

### Production Deployment (Phase 5)

**Frontend (Vercel):**
- Build command: `npm run build`
- Output directory: `dist`
- Environment: `VITE_API_URL` points to production backend
- CDN: Automatic via Vercel Edge Network

**Backend (Render or Railway):**
- Start command: `node src/server.js`
- Environment: `OPENAI_API_KEY`, `PORT`, `NODE_ENV=production`
- Health check: `GET /api/health`
- Auto-scaling: Platform-provided

**Monitoring:**
- Analytics: Vercel Analytics
- Logs: Platform console logs
- Costs: OpenAI dashboard (daily checks)
- Uptime: Platform monitoring

**Deployment Checklist:**
- Frontend build tested locally
- Production environment variables configured
- CORS set for production domain
- OpenAI budget limits active ($40 alert, $50 stop)
- Health check endpoint verified
- End-to-end production test passed

---

## File Structure

```
meme-generator/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Upload.jsx
│   │   │   ├── ContextControls.jsx
│   │   │   ├── CaptionOptions.jsx
│   │   │   ├── MemePreview.jsx
│   │   │   └── DownloadButton.jsx
│   │   ├── utils/
│   │   │   ├── canvas.js (text rendering helpers)
│   │   │   ├── api.js (backend API calls)
│   │   │   └── validation.js (client-side validation)
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── package.json
│   ├── vite.config.js
│   └── .env.example
│
├── backend/
│   ├── src/
│   │   ├── routes/
│   │   │   └── caption.js
│   │   ├── services/
│   │   │   └── openai.js (OpenAI SDK wrapper)
│   │   ├── middleware/
│   │   │   └── validation.js
│   │   └── server.js
│   ├── package.json
│   └── .env.example
│
├── ai/
│   ├── guides/ (technical reference docs)
│   ├── roadmaps/ (implementation plans)
│   └── changelog.md
│
├── aiDocs/
│   ├── architecture.md (this file)
│   ├── context.md
│   ├── prd.md
│   ├── mvp.md
│   └── market-research.md
│
└── README.md
```

**Note:** Monorepo-lite structure - both frontend and backend in single repo for MVP simplicity.

---

## Technology Decisions & Rationale

### Why React?
- Fast UI updates for preview rendering
- Component reusability
- Large ecosystem
- Easy Canvas integration

### Why Vite?
- Extremely fast dev server
- Modern ESM-based build
- No configuration needed
- Better DX than CRA

### Why Express.js?
- Lightweight and minimal
- Easy OpenAI SDK integration
- Good middleware ecosystem (cors)
- No over-engineering for MVP
- **Note:** Multer removed - using JSON with base64 instead of multipart uploads for simplicity

### Why Canvas (not Sharp on backend)?
- **Simpler architecture:** No image round-trip to backend for rendering
- **Faster UX:** Real-time preview updates
- **Lower server costs:** Client-side processing
- **Good enough quality:** Canvas text rendering is sufficient for memes
- **Immediate feedback:** Users see text overlay as they select captions

### Why gpt-4o?
- **Vision-capable:** Can analyze uploaded images
- **Fast:** ~5-10s response time
- **Cost-effective:** $0.003 per generation
- **JSON mode:** Structured output without parsing hacks
- **Current standard:** Replaces deprecated gpt-4-vision-preview

---

## Implementation Phases

This architecture is implemented across 6 phases (6 weeks total). See `ai/roadmaps/2026-02-16-implementation-roadmap.md` for details.

**Phase 0 (Week 1):** Foundation
- Project setup (Vite + Express)
- OpenAI API integration
- Test caption generation

**Phase 1 (Weeks 2-3):** Core Flow
- Upload UI with validation
- Canvas text rendering
- Caption selection and download
- 10-generation limit enforcement

**Phase 2 (Week 3-4):** Context Controls
- 4 preset tone buttons
- Custom prompt input
- Dynamic prompt generation

**Phase 3 (Week 4-5):** Polish & Testing
- Error handling and messages
- Analytics (Vercel Analytics)
- Cost monitoring
- Production deployment prep

**Phase 4 (Week 5-6):** Beta Testing
- 20-30 Gen Z users (18-24)
- Survey feedback collection
- Metrics validation

**Phase 5 (Week 6):** Public Launch
- Deploy to production
- Product Hunt launch
- Monitor metrics

---

## Post-MVP Enhancements (V2+)

### Technical Improvements
- [ ] Sharp on backend for optional high-quality rendering
- [ ] WebSocket for real-time generation updates
- [ ] Redis caching for repeated image analyses
- [ ] Progressive Web App (PWA) features
- [ ] Video/GIF meme support (FFmpeg integration)
- [ ] Mobile app (React Native)

### Feature Additions
- [ ] User accounts & saved memes (PostgreSQL)
- [ ] Social sharing integrations (Twitter, Instagram APIs)
- [ ] Meme templates library
- [ ] Batch processing (multiple images)
- [ ] Additional preset tones (Gen Z Slang, Work Humor)
- [ ] Caption rating/feedback system
- [ ] Community gallery

---

## Key Architecture Decisions

### Decision Log

**2026-02-16 - Initial Architecture:**
- ✅ Canvas rendering on frontend (not Sharp on backend)
- ✅ gpt-4o model (not deprecated gpt-4-vision-preview)
- ✅ In-memory storage only (no database)
- ✅ React + Vite frontend, Express backend
- ✅ 3 caption options per generation (not single)
- ✅ 10-generation limit per image (cost control)
- ✅ JSON with base64 (not multipart uploads)
- ✅ 4 preset tones + custom prompt support
- ✅ Monorepo-lite structure (single repo, separate folders)

### Anti-Patterns Avoided

This is a **clean-slate project**. We explicitly avoid:
- ❌ Over-engineering for theoretical scale
- ❌ Unnecessary abstractions and middleware
- ❌ "Just in case" features
- ❌ Legacy compatibility layers
- ❌ Premature optimization
- ❌ Complex state management (Redux, etc.)
- ❌ Automated testing for MVP (manual testing sufficient)

**Philosophy:** Ship working software. Learn from users. Add complexity only when actually needed.

---

## References

**Technical Documentation:**
- OpenAI gpt-4o API: https://platform.openai.com/docs/api-reference/chat
- Canvas API: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- React: https://react.dev/
- Vite: https://vitejs.dev/
- Express: https://expressjs.com/

**Project Documentation:**
- Implementation Roadmap: `ai/roadmaps/2026-02-16-implementation-roadmap.md`
- OpenAI Guide: `ai/guides/openai-vision-api-docs.md`
- Canvas Guide: `ai/guides/canvas-api_context7.md`
- PRD: `aiDocs/prd.md`
- MVP Scope: `aiDocs/mvp.md`

---

**Version:** 1.1  
**Last Updated:** 2026-02-16  
**Status:** Implementation Ready  
**Next Review:** After Phase 1 completion
