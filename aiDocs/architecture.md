# System Architecture: AI-Powered Meme Generator

**Version:** 1.0  
**Last Updated:** 2026-02-16  
**Status:** Design Phase

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
- Backend: Request-scoped variables
- Uploaded images cleared after generation

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
│   ├── Accepts: multipart/form-data or JSON with base64
│   ├── Returns: JSON with caption(s)
│   └── Error handling
│
└── /health (GET) - optional health check
```

---

## Data Flow

### 1. Image Upload Flow

```javascript
// Frontend
User selects image
  → File validated (type, size)
  → Convert to base64 (or keep as File)
  → Store in React state
  → Show preview thumbnail
  → Enable context controls + generate button
```

### 2. Caption Generation Flow

```javascript
// Frontend → Backend
User clicks "Generate Meme"
  → Collect: image (base64), context (preset/custom/none)
  → POST to /api/generate-caption
  → Show loading spinner

// Backend → OpenAI
Receive request
  → Validate image format/size
  → Build system prompt based on context type
  → Call OpenAI Chat Completions API
    • Model: gpt-4o
    • Messages: [system prompt, user message with image]
    • Response format: { type: "json_object" }
    • Temperature: 0.8
    • Max tokens: 300-500
  → Parse JSON response
  → Return to frontend

// Frontend displays
Receive caption(s)
  → Update state with options
  → Render caption cards (if multiple)
  → Auto-apply first option to preview
  → Enable download button
```

### 3. Text Overlay Rendering Flow

```javascript
// Frontend (Canvas)
User selects caption
  → Get canvas context
  → Load uploaded image into canvas
  → Calculate dimensions:
    • Font size = image width * 0.05
    • Top text position = height * 0.10
    • Bottom text position = height * 0.90
  → Draw image to canvas
  → Set text properties:
    • Font: "Impact, Arial Black"
    • Fill: white
    • Stroke: black (width = font size * 0.05)
    • Text transform: uppercase
    • Alignment: center
  → Draw top text (with word wrap if needed)
  → Draw bottom text (with word wrap if needed)
  → Update preview
```

### 4. Download Flow

```javascript
// Frontend
User clicks "Download Meme"
  → Get canvas element
  → Convert to Blob: canvas.toBlob(blob => {...})
  → Create download link
  → Trigger download: filename = `meme-${Date.now()}.png`
  → Clear temp state (optional)
```

---

## API Specifications

### POST /api/generate-caption

**Request:**
```json
{
  "image": "data:image/jpeg;base64,/9j/4AAQSkZJRg...",
  "context": {
    "type": "preset",  // "none" | "preset" | "custom"
    "preset": "sarcastic",  // if type = "preset"
    "customPrompt": "make it about Monday mornings"  // if type = "custom"
  }
}
```

**Response (Single Caption):**
```json
{
  "success": true,
  "caption": {
    "topText": "WHEN YOU FIND THE PERFECT PARKING SPOT",
    "bottomText": "AT THE GROCERY STORE"
  }
}
```

**Response (Multiple Captions):**
```json
{
  "success": true,
  "captions": [
    {
      "id": 1,
      "topText": "WHEN YOU FIND THE PERFECT PARKING SPOT",
      "bottomText": "AT THE GROCERY STORE"
    },
    {
      "id": 2,
      "topText": "POV: YOU'RE ABOUT TO MAKE BAD DECISIONS",
      "bottomText": "BUT IT'S GOING TO BE WORTH IT"
    },
    {
      "id": 3,
      "topText": "NOBODY:",
      "bottomText": "ABSOLUTELY NOBODY: THIS GUY"
    }
  ]
}
```

**Error Response:**
```json
{
  "success": false,
  "error": "Invalid image format",
  "message": "Supported formats: JPG, PNG, WebP"
}
```

---

## OpenAI Integration Details

### Prompt Engineering

**Base System Prompt (No Context):**
```
You are a funny meme caption generator. Analyze this image and create ONE meme caption in classic top-text/bottom-text format.

Requirements:
- Top text should set up the joke (context)
- Bottom text should be the punchline
- Use ALL CAPS
- Keep it short (5-15 words per line)
- Be funny and internet-native (not corporate)
- Match the content/context of the image

Return ONLY a JSON object:
{
  "topText": "YOUR TOP TEXT HERE",
  "bottomText": "YOUR BOTTOM TEXT HERE"
}
```

**With Preset Style:**
```
[Base prompt]

Style: SARCASTIC
Tone description: Dry, ironic, "oh great, just what I needed" energy
IMPORTANT: Use this tone/style for the humor.
```

**With Custom Prompt:**
```
[Base prompt]

User's creative direction: "make it about Monday mornings"
IMPORTANT: Incorporate the user's creative direction into the caption.
```

### API Configuration

```javascript
const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    {
      role: "system",
      content: systemPrompt
    },
    {
      role: "user",
      content: [
        {
          type: "image_url",
          image_url: {
            url: `data:image/jpeg;base64,${base64Image}`,
            detail: "high"  // or "low" for cost optimization
          }
        }
      ]
    }
  ],
  response_format: { type: "json_object" },
  max_tokens: 300,  // single caption
  temperature: 0.8  // balanced creativity
});
```

### Cost Estimation

**Per Image (1024×1024, detail: "high"):**
- Image tokens: ~765 tokens
- System prompt: ~150 tokens
- Response: ~50 tokens
- **Total:** ~965 tokens per generation

**Pricing (as of 2026-02-16):**
- gpt-4o: $2.50/1M input tokens, $10.00/1M output tokens
- **Cost per generation:** ~$0.003 (0.3 cents)
- **With 10 regenerations:** ~$0.03 per image

---

## Canvas Text Rendering Implementation

### Algorithm

```javascript
function renderMemeText(canvas, image, topText, bottomText) {
  const ctx = canvas.getContext('2d');
  
  // Set canvas size to match image
  canvas.width = image.width;
  canvas.height = image.height;
  
  // Draw base image
  ctx.drawImage(image, 0, 0);
  
  // Calculate responsive font size (5% of width)
  const fontSize = Math.floor(image.width * 0.05);
  
  // Configure text style
  ctx.font = `bold ${fontSize}px Impact, Arial Black, sans-serif`;
  ctx.fillStyle = 'white';
  ctx.strokeStyle = 'black';
  ctx.lineWidth = fontSize * 0.05;
  ctx.textAlign = 'center';
  ctx.textBaseline = 'top';
  
  // Convert to uppercase
  const topUpper = topText.toUpperCase();
  const bottomUpper = bottomText.toUpperCase();
  
  // Word wrap helper
  const wrapText = (text, maxWidth) => {
    const words = text.split(' ');
    const lines = [];
    let currentLine = words[0];
    
    for (let i = 1; i < words.length; i++) {
      const testLine = currentLine + ' ' + words[i];
      const metrics = ctx.measureText(testLine);
      
      if (metrics.width > maxWidth) {
        lines.push(currentLine);
        currentLine = words[i];
      } else {
        currentLine = testLine;
      }
    }
    lines.push(currentLine);
    return lines;
  };
  
  // Wrap text (max 90% of canvas width)
  const maxWidth = image.width * 0.9;
  const topLines = wrapText(topUpper, maxWidth);
  const bottomLines = wrapText(bottomUpper, maxWidth);
  
  // Draw top text
  const topY = image.height * 0.05;
  topLines.forEach((line, i) => {
    const y = topY + (i * fontSize * 1.2);
    ctx.strokeText(line, image.width / 2, y);
    ctx.fillText(line, image.width / 2, y);
  });
  
  // Draw bottom text
  const bottomY = image.height * 0.90 - (bottomLines.length * fontSize * 1.2);
  bottomLines.forEach((line, i) => {
    const y = bottomY + (i * fontSize * 1.2);
    ctx.strokeText(line, image.width / 2, y);
    ctx.fillText(line, image.width / 2, y);
  });
}
```

---

## Error Handling Strategy

### Frontend

```javascript
// Upload validation
- File type check (jpg, png, webp, gif)
- File size limit (10MB)
- Image dimensions check (optional)

// API call errors
- Network timeout (30s)
- 4xx/5xx responses
- Invalid JSON response
- Show user-friendly error messages
```

### Backend

```javascript
// Request validation
- Image format validation
- Base64 decode errors
- Missing required fields

// OpenAI API errors
- Rate limit (429) → retry with exponential backoff
- Invalid API key (401) → log and return 500
- Timeout → return 504
- Invalid response → retry once, then fail gracefully

// Response handling
- Validate JSON structure
- Sanitize output (escape HTML if needed)
- Log errors for debugging
```

---

## Performance Considerations

### Frontend Optimization
- Lazy load Canvas API
- Debounce text preview updates
- Use `requestAnimationFrame` for smooth rendering
- Compress images client-side if > 5MB

### Backend Optimization
- Stream image uploads (don't buffer entire file in memory)
- Implement request timeout (30s)
- Add rate limiting per IP (10 requests/minute)
- Cache system prompts (don't rebuild on every request)

### OpenAI API Optimization
- Use `detail: "low"` for cost-sensitive scenarios (85 tokens vs 765+)
- Batch multiple requests if possible (not applicable for MVP)
- Implement caching for identical image+context pairs (future)

---

## Security Considerations

### MVP (Minimal)
- Validate image file types
- Limit file size (10MB)
- Sanitize user input (custom prompts)
- Environment variables for API keys
- CORS configuration (localhost only)

### Future (Production)
- Rate limiting (per user/IP)
- CAPTCHA for abuse prevention
- Content moderation (OpenAI API safety)
- HTTPS only
- Input sanitization (XSS prevention)
- API key rotation

---

## Deployment Architecture (Future)

### MVP Deployment
```
Local Development:
- Frontend: Vite dev server (localhost:5173)
- Backend: Node.js server (localhost:3001)
- No cloud deployment needed
```

### Production Deployment (V2)
```
Frontend:
- Host: Vercel or Netlify
- CDN: Automatic
- Environment: Browser

Backend:
- Host: Railway, Render, or Fly.io
- Environment: Node.js container
- Secrets: OpenAI API key in env vars

Infrastructure:
- Frontend → API Gateway → Backend
- Monitoring: Sentry for errors
- Analytics: Plausible or Vercel Analytics
```

---

## File Structure

```
meme-generator/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── UploadZone.jsx
│   │   │   ├── ContextControls.jsx
│   │   │   ├── CaptionOptions.jsx
│   │   │   ├── MemePreview.jsx
│   │   │   └── DownloadButton.jsx
│   │   ├── utils/
│   │   │   ├── canvas.js (text rendering)
│   │   │   ├── api.js (backend calls)
│   │   │   └── validation.js
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── package.json
│   └── vite.config.js
│
├── backend/
│   ├── src/
│   │   ├── routes/
│   │   │   └── caption.js
│   │   ├── services/
│   │   │   ├── openai.js
│   │   │   └── prompt.js
│   │   ├── middleware/
│   │   │   ├── upload.js (multer)
│   │   │   └── validation.js
│   │   └── server.js
│   ├── package.json
│   └── .env.example
│
├── aiDocs/
│   ├── architecture.md (this file)
│   ├── context.md
│   ├── prd.md
│   └── mvp.md
│
└── README.md
```

---

## Technology Decisions & Rationale

### Why React?
- Fast UI updates for preview rendering
- Component reusability
- Large ecosystem
- Easy Canvas integration

### Why Express.js?
- Lightweight and minimal
- Easy OpenAI SDK integration
- Good middleware ecosystem (multer, cors)
- No over-engineering for MVP

### Why Canvas (not Sharp on backend)?
- **Simpler architecture:** No image round-trip to backend for rendering
- **Faster UX:** Real-time preview updates
- **Lower server costs:** Client-side processing
- **Good enough quality:** Canvas text rendering is sufficient for memes

### Why gpt-4o?
- **Vision-capable:** Can analyze uploaded images
- **Fast:** ~5-10s response time
- **Cost-effective:** $0.003 per generation
- **JSON mode:** Structured output without parsing hacks

---

## Future Enhancements (Post-MVP)

### Technical Improvements
- [ ] Sharp on backend for higher-quality rendering
- [ ] WebSocket for real-time generation updates
- [ ] Redis caching for repeated image analyses
- [ ] Progressive Web App (PWA) for offline support
- [ ] Video/GIF meme support (FFmpeg integration)

### Feature Additions
- [ ] User accounts & saved memes (PostgreSQL)
- [ ] Social sharing integrations (Twitter, Instagram APIs)
- [ ] Meme templates library
- [ ] Batch processing (multiple images)
- [ ] A/B testing for caption quality

---

## References

- Sharp API: https://sharp.pixelplumbing.com/
- OpenAI Chat Completions: https://platform.openai.com/docs/api-reference/chat
- Canvas API: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- React Docs: https://react.dev/

---

**Last Reviewed:** 2026-02-16  
**Next Review:** After MVP implementation
