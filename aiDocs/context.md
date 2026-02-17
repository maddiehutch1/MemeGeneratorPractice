# Project Context: AI-Powered Meme Generator

## Quick Reference
- **PRD:** `aiDocs/prd.md`
- **MVP Scope:** `aiDocs/mvp.md`
- **Market Research:** `aiDocs/market-research.md`
- **Architecture:** `aiDocs/architecture.md`

## Behavior
Whenever creating plan docs and roadmap docs, always save them in ai/roadmaps. Prefix the name with the date. Add a note that we need to avoid over-engineering, cruft, and legacy-compatibility features in this clean code project. Make sure they reference each other.
- Whenever finishing with implementing a plan / roadmap doc pair, make sure the roadmap is up to date (tasks checked off, etc). Then move the docs to ai/roadmaps/complete. Then update `ai/changelog.md` accordingly.

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
- **Changelog:** See `ai/changelog.md` for changelog with brief notes about each change to the codebase
- **PRD:** See `aiDocs/prd.md` for full product requirements
- **MVP Spec:** See `aiDocs/mvp.md` for demo scope and implementation plan
- **Market Research:** See `aiDocs/market-research.md` for competitive analysis and market validation
---

**Last Updated:** 2026-02-16
