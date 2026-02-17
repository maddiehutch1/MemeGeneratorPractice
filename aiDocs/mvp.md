# MVP Scope: AI Meme Generator Demo

**Version:** 1.0  
**Last Updated:** February 16, 2026  
**Goal:** Build a functional demo that proves the core concept works

---

## MVP Objective

Create a working demo that shows:
1. AI can analyze an uploaded image
2. AI can generate contextually funny captions
3. User can preview the meme with text overlay
4. User can download the final result

**Target:** Functional demo for stakeholders/early testers, not production-ready product.

---

## Architecture

### Frontend (Web Page)
- **Tech:** React + Vite (or plain HTML/JS if faster)
- **Responsibilities:**
  - File upload UI
  - Display generated captions
  - Render text overlay preview (Canvas)
  - Trigger download

### Backend (Service)
- **Tech:** Node.js + Express
- **Responsibilities:**
  - Receive uploaded image
  - Call OpenAI Vision API (gpt-4o)
  - Return generated captions
  - Handle errors
- **Note:** Text overlay rendering handled on frontend (Canvas). Sharp library referenced for optional future backend rendering only.

### Data Flow
```
User uploads image
    ↓
Frontend sends to backend
    ↓
Backend sends to OpenAI Vision API
    ↓
OpenAI returns caption(s)
    ↓
Backend returns to frontend
    ↓
Frontend renders preview with Canvas
    ↓
User clicks download
    ↓
Canvas exports as PNG
```

---

## Core Features (MVP Only)

### 1. Image Upload
**Implementation:**
- Single file input (`<input type="file" accept="image/*">`)
- Display preview of uploaded image
- Store in memory (no server persistence)
- Supported formats: JPG, PNG
- Max size: 10MB (client-side validation)

**UI:**
- Drag-and-drop OR click to browse
- Show image thumbnail after upload
- "Upload" button to proceed

---

### 2. Caption Generation with Configurable Context
**Implementation:**
- Send image + optional context to backend via multipart/form-data
- Backend calls OpenAI gpt-4o Vision API
- User can provide context in three ways:

**Context Options:**

**A. Upload and Go (Default)**
- No user input required
- AI analyzes image and decides tone/style automatically
- Fastest path to meme

**B. Predefined Styles (Quick Selection)**
- Dropdown or button group with presets:
  - 🎭 Sarcastic
  - 🌈 Wholesome
  - 🤪 Absurd/Surreal
  - 🔥 Roast/Savage
  - 😎 Gen Z Slang
  - 💼 Relatable Work Humor
- User selects before generating
- AI uses style hint in prompt

**C. Custom Prompt (Full Control)**
- Text input field: "What kind of meme? (optional)"
- Examples shown: "make it about Monday mornings", "roast my terrible parking", "wholesome dog content"
- AI incorporates user's instructions

**Caption Output Options:**

**Option A: Single Caption (Simplest)**
- Generate 1 caption per request
- Show immediately in preview
- Add "Regenerate" button for iteration

**Option B: Multiple Captions (Better Demo)**
- Generate 3 caption options per request
- Display as selectable cards
- User clicks one to apply to preview
- Add "More Options" button to generate 3 more

**Recommendation:** 
- Start with Option A (single caption) + all three context options
- Upgrade to Option B (multiple captions) if time allows
- Context control is your differentiator - prioritize this!

---

### 3. Preview with Text Overlay
**Implementation:**
- Use HTML Canvas to composite image + text
- Classic meme style:
  - Impact font (or Arial Black as fallback)
  - White text with black stroke
  - Top text: centered, near top (10% from edge)
  - Bottom text: centered, near bottom (10% from edge)
  - All caps
  - Max 2 lines per text block (word wrap)

**UI:**
- Live preview updates when user selects caption
- Preview canvas should match aspect ratio of original image
- Scale to fit viewport (max 800px width)

**Technical Notes:**
- Use `ctx.strokeText()` for black outline
- Use `ctx.fillText()` for white fill
- Calculate font size relative to image dimensions (~5% of image height)
- Center text horizontally: `ctx.textAlign = 'center'`

---

### 4. Download
**Implementation:**
- "Download Meme" button
- Export canvas as PNG using `canvas.toBlob()`
- Trigger browser download
- Filename: `meme-{timestamp}.png`

**No:**
- No saving to server
- No social media sharing
- No format options (PNG only)

---

### 5. Regenerate/Iterate (Optional but Recommended)
**Why include:** Shows off your differentiator (iterative AI humor)

**Implementation:**
- "Try Again" button (if Option A)
- "More Options" button (if Option B)
- Calls backend again with same image
- Optionally: add tone hint to prompt ("make it more sarcastic")

**Recommendation:** Include this even in MVP. Without it, you're just another meme generator. This is your wedge.

---

## MVP User Flow

### Happy Path (Option A - Single Caption)
1. User lands on page → sees upload zone
2. User uploads image → thumbnail appears
3. **Context selection (optional):**
   - User can leave blank (upload and go)
   - OR select preset style (dropdown/buttons)
   - OR enter custom prompt (text field)
4. User clicks "Generate Meme" → loading spinner
5. AI analyzes → caption appears in preview (5-10s wait)
6. User sees meme preview with text overlay
7. **If funny:** User clicks "Download" → PNG saved
8. **If not funny:** User modifies context and clicks "Try Again" → new caption generated
9. Repeat step 7-8 until satisfied

### Happy Path (Option B - Multiple Captions)
1. User lands on page → sees upload zone
2. User uploads image → thumbnail appears
3. **Context selection (optional):**
   - User can leave blank (upload and go)
   - OR select preset style (dropdown/buttons)
   - OR enter custom prompt (text field)
4. User clicks "Generate Meme" → loading spinner
5. AI analyzes → 3 caption cards appear (5-10s wait)
6. User clicks caption card → preview updates with that option
7. **If funny:** User clicks "Download" → PNG saved
8. **If not funny:** User modifies context and clicks "More Options" → 3 new cards appear
9. Repeat step 6-8 until satisfied

---

## Out of Scope (MVP)

### Not Building:
- ❌ User accounts / authentication
- ❌ Database storage
- ❌ Caption history/favorites
- ❌ Manual text editing
- ❌ Font/color/position controls
- ❌ Video/GIF support
- ❌ Social media integrations
- ❌ Tone controls (wholesome/sarcastic/roast)
- ❌ Refinement prompts ("make it more like X")
- ❌ Mobile responsiveness optimization (works on desktop first)
- ❌ Analytics/tracking
- ❌ Rate limiting (trust yourself not to spam)
- ❌ Production deployment (local/staging only)

### Can Add Later (Post-MVP):
- Multiple caption options (if starting with single caption)
- More preset styles (niche humor types)
- Caption rating/feedback (thumbs up/down)
- Better error handling
- Loading optimizations
- Mobile UX
- Polished design

---

## Technical Decisions

### Frontend
- **Framework:** React (already familiar) OR plain HTML/JS (faster to build)
- **Styling:** Minimal CSS, function over form
- **State management:** React useState (no Redux/Context needed)

### Backend
- **Framework:** Express.js
- **Image handling:** Multer (for file uploads)
- **OpenAI SDK:** `openai` npm package (gpt-4o model)
- **CORS:** Enable for local development
- **Note:** Sharp library docs included for reference but NOT required for MVP (Canvas handles rendering)

### Deployment (MVP)
- **Frontend:** Run locally (`npm run dev`)
- **Backend:** Run locally (separate port, e.g., 3001)
- **No cloud deployment needed for demo**

### Environment Variables
```
OPENAI_API_KEY=sk-...
PORT=3001
```

---

## API Endpoints

### `POST /api/generate-caption`
**Request:**
```json
{
  "image": "<base64-encoded image OR multipart file>",
  "context": {
    "type": "none" | "preset" | "custom",
    "preset": "sarcastic" | "wholesome" | "absurd" | "roast" | "genz" | "work",
    "customPrompt": "make it about Monday mornings"
  }
}
```

**Context Field (Optional):**
- If omitted or `type: "none"` → AI decides automatically
- If `type: "preset"` → use preset style hint
- If `type: "custom"` → incorporate customPrompt into generation

**Response:**
```json
{
  "topText": "WHEN YOU FIND THE PERFECT PARKING SPOT",
  "bottomText": "AT THE GROCERY STORE"
}
```

**OR** (if Option B - multiple captions):
```json
{
  "options": [
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
  "error": "Failed to generate caption",
  "message": "OpenAI API error"
}
```

---

## OpenAI Integration

### Prompt Template (Single Caption)

**Base Prompt (No Context):**
```
You are a funny meme caption generator. Analyze this image and create ONE meme caption in classic top-text/bottom-text format.

Requirements:
- Top text should set up the joke (context)
- Bottom text should be the punchline
- Use ALL CAPS
- Keep it short (5-15 words per line)
- Be funny and internet-native (not corporate)
- Match the content/context of the image

Return ONLY a JSON object with this structure:
{
  "topText": "YOUR TOP TEXT HERE",
  "bottomText": "YOUR BOTTOM TEXT HERE"
}
```

**With Preset Style:**
```
You are a funny meme caption generator. Analyze this image and create ONE meme caption in classic top-text/bottom-text format.

Style: {SARCASTIC/WHOLESOME/ABSURD/ROAST/GEN_Z/WORK}

Requirements:
- Top text should set up the joke (context)
- Bottom text should be the punchline
- Use ALL CAPS
- Keep it short (5-15 words per line)
- Be funny and internet-native (not corporate)
- Match the content/context of the image
- IMPORTANT: Use the {STYLE} tone/style for the humor

Return ONLY a JSON object with this structure:
{
  "topText": "YOUR TOP TEXT HERE",
  "bottomText": "YOUR BOTTOM TEXT HERE"
}
```

**Preset Style Descriptions (add to prompt):**
- **Sarcastic:** Dry, ironic, "oh great, just what I needed" energy
- **Wholesome:** Warm, positive, pure, "faith in humanity restored" vibes
- **Absurd/Surreal:** Random, nonsensical, chaotic, weird internet humor
- **Roast/Savage:** Mean (but funny), call-out culture, no mercy
- **Gen Z:** Slang-heavy, chronically online, "no cap fr fr" energy
- **Work:** Office humor, Monday mornings, corporate dread, relatable work struggles

**With Custom Prompt:**
```
You are a funny meme caption generator. Analyze this image and create ONE meme caption in classic top-text/bottom-text format.

User's creative direction: "{CUSTOM_PROMPT}"

Requirements:
- Top text should set up the joke (context)
- Bottom text should be the punchline
- Use ALL CAPS
- Keep it short (5-15 words per line)
- Be funny and internet-native (not corporate)
- Match the content/context of the image
- IMPORTANT: Incorporate the user's creative direction: "{CUSTOM_PROMPT}"

Return ONLY a JSON object with this structure:
{
  "topText": "YOUR TOP TEXT HERE",
  "bottomText": "YOUR BOTTOM TEXT HERE"
}
```

### Prompt Template (Multiple Captions)

**Base Prompt (No Context):**
```
You are a funny meme caption generator. Analyze this image and create THREE different meme captions in classic top-text/bottom-text format.

Each caption should have a DIFFERENT comedic angle (sarcastic, wholesome, absurd, etc).

Requirements:
- Top text should set up the joke (context)
- Bottom text should be the punchline
- Use ALL CAPS
- Keep it short (5-15 words per line)
- Be funny and internet-native (not corporate)
- Match the content/context of the image

Return ONLY a JSON array with this structure:
[
  {
    "topText": "OPTION 1 TOP TEXT",
    "bottomText": "OPTION 1 BOTTOM TEXT"
  },
  {
    "topText": "OPTION 2 TOP TEXT",
    "bottomText": "OPTION 2 BOTTOM TEXT"
  },
  {
    "topText": "OPTION 3 TOP TEXT",
    "bottomText": "OPTION 3 BOTTOM TEXT"
  }
]
```

**With Context (Preset or Custom):**
```
You are a funny meme caption generator. Analyze this image and create THREE different meme captions in classic top-text/bottom-text format.

{IF PRESET}: Style/tone: {STYLE_NAME} - {STYLE_DESCRIPTION}
{IF CUSTOM}: User's creative direction: "{CUSTOM_PROMPT}"

Each caption should explore the {style/direction} in different ways.

Requirements:
- Top text should set up the joke (context)
- Bottom text should be the punchline
- Use ALL CAPS
- Keep it short (5-15 words per line)
- Be funny and internet-native (not corporate)
- Match the content/context of the image
- IMPORTANT: Follow the user's style/direction while varying the approach

Return ONLY a JSON array with this structure:
[
  {
    "topText": "OPTION 1 TOP TEXT",
    "bottomText": "OPTION 1 BOTTOM TEXT"
  },
  {
    "topText": "OPTION 2 TOP TEXT",
    "bottomText": "OPTION 2 BOTTOM TEXT"
  },
  {
    "topText": "OPTION 3 TOP TEXT",
    "bottomText": "OPTION 3 BOTTOM TEXT"
  }
]
```

### API Configuration
- **Model:** `gpt-4o` (current vision-capable model, replaces deprecated gpt-4-vision-preview)
- **Max tokens:** 300 (enough for captions)
- **Temperature:** 0.8 (creative but not random)

---

## Error Handling (MVP)

### Must Handle:
- File upload fails (wrong format, too large)
- OpenAI API errors (rate limit, timeout, invalid response)
- Network errors (frontend ↔ backend)

### How:
- Show error message to user
- "Try Again" button to retry
- Don't crash the app

### Don't Need (MVP):
- Retry logic
- Graceful degradation
- Detailed error logging
- User analytics on errors

---

## Success Criteria (MVP Demo)

### Must Work:
- ✅ Upload an image
- ✅ Generate at least 1 caption that makes sense
- ✅ Preview shows text overlay correctly
- ✅ Download produces valid PNG file

### Nice to Have:
- ✅ Generate 3 caption options
- ✅ Regenerate/iterate functionality
- ✅ Captions are actually funny (subjective)
- ✅ Text wrapping handles long captions
- ✅ Works on different image aspect ratios

### Not Required:
- ❌ Perfect design/polish
- ❌ Mobile optimization
- ❌ Fast load times (<1s)
- ❌ 100% uptime
- ❌ Security hardening

---

## Development Timeline (Estimate)

### Day 1: Backend
- Set up Express server
- OpenAI API integration
- Test caption generation with sample images
- Verify JSON response format

### Day 2: Frontend Foundation
- Upload UI
- Context controls (preset dropdown + custom text input)
- Connect to backend
- Display loading state
- Show generated captions

### Day 3: Preview & Download
- Canvas rendering (text overlay)
- Font styling (Impact, stroke, positioning)
- Download functionality
- Test with various images

### Day 4: Iteration & Polish
- Add regenerate button
- Multiple caption options (if time)
- Error handling
- Basic styling
- End-to-end testing

**Total:** ~4 days for solo developer

---

## Testing Plan (MVP)

### Manual Testing:
1. Upload 5 different images (portrait, landscape, square, with people, without people)
2. Verify captions make sense for each
3. Check text positioning on different aspect ratios
4. Download and verify PNG quality
5. Test "Try Again" / "More Options" flow
6. Test error cases (no API key, bad image format)

### Test Images:
- Pet photo (dog/cat)
- Selfie
- Landscape/nature
- Group photo
- Random object
- Screenshot/meme template (edge case)

---

## Open Questions

- [ ] **Single caption vs 3 options?** (Recommend 3 for better demo)
- [ ] **Include "Try Again" button?** (Recommend YES - shows your differentiator)
- [ ] **Context UI: Dropdown, buttons, or tabs?** (Recommend: default to "upload and go", collapsible "Advanced" section for presets/custom)
- [ ] **Where to render text - frontend or backend?** (Recommend frontend Canvas for simplicity)
- [ ] **Deployment for demo?** (Local is fine, or Vercel + Railway/Render if showing remotely)
- [ ] **Font licensing for Impact?** (Use web-safe fallback like Arial Black for MVP)

---

## Post-MVP Next Steps

Once demo works:
1. Show to 5-10 target users (Gen Z friends)
2. Collect feedback:
   - Were captions funny?
   - Was iteration useful?
   - Did they want to share the result?
3. Iterate on prompt engineering
4. Decide: Build full product or pivot?

---

## File Structure (Suggested)

```
meme-generator-mvp/
├── frontend/
│   ├── src/
│   │   ├── App.jsx
│   │   ├── components/
│   │   │   ├── Upload.jsx
│   │   │   ├── ContextControls.jsx  ← NEW
│   │   │   ├── CaptionOptions.jsx
│   │   │   ├── MemePreview.jsx
│   │   │   └── Download.jsx
│   │   └── utils/
│   │       └── canvas.js
│   ├── package.json
│   └── index.html
│
├── backend/
│   ├── server.js
│   ├── routes/
│   │   └── generate.js
│   ├── services/
│   │   └── openai.js
│   ├── package.json
│   └── .env
│
└── README.md
```

---

## Notes

- **Focus on proving the concept, not perfecting the product**
- **AI humor quality is the biggest risk** - spend time on prompt engineering
- **Canvas text rendering is tricky** - test early with different images
- **Keep it simple** - resist scope creep during MVP build

---

**Ready to build?** Start with backend (OpenAI integration) since that's the hardest/riskiest part. Once you can generate captions, frontend is straightforward.
