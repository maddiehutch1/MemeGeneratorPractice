# Project Context: AI-Powered Meme Generator

## Quick Reference
- **PRD:** `aiDocs/prd.md`
- **MVP Scope:** `aiDocs/mvp.md`
- **Market Research:** `aiDocs/market-research.md` (moved from ai/guides/)

## Tech Stack
- **Frontend:** React + Vite
- **Backend:** Node.js + Express  
- **AI:** OpenAI GPT-4o (Chat Completions API)
- **Image Processing:** HTML Canvas (client-side text overlay)

## Core Value Proposition
Upload any photo → AI generates contextually funny captions → Iterate with configurable context → Download meme

**Key Differentiator:** Iterative AI-powered humor with user-guided context (presets or custom prompts)

## Current Focus
- [ ] MVP development - backend OpenAI integration
- [ ] Prompt engineering for caption quality
- [ ] Canvas text rendering implementation
- [ ] User flow: Upload → Context → Generate → Preview → Download

## Scope Decisions (MVP)
- ✅ Classic top/bottom text format only
- ✅ Configurable context (upload-and-go, presets, custom)
- ✅ In-memory storage (no database)
- ✅ Single or 3-caption generation options
- ❌ No manual text editing
- ❌ No video/GIF support
- ❌ No user accounts

## Success Metrics
- Caption quality (first-pick rate)
- Iteration usage (context refinement)
- Completion rate (upload → download)

## Documentation
- **Architecture:** See `aiDocs/architecture.md` for system design details

---

**Last Updated:** 2026-02-16
