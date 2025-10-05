# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Language Preference

**IMPORTANT: Always respond in Japanese (日本語) when working in this repository.**

This codebase is primarily in Japanese, with Japanese UI text, comments, and documentation. All interactions with developers should be conducted in Japanese to maintain consistency and clarity.

## Overview

This is a fork of GeminiPWA - a Progressive Web Application (PWA) for chatting with multiple AI APIs including Gemini, DeepSeek, Claude (Anthropic), OpenAI, xAI (Grok), and LLM aggregators like OpenRouter. It's a single-page application (SPA) built entirely with vanilla JavaScript, HTML, and CSS with no build process required.

**Key Characteristics:**
- Pure client-side execution (no backend server)
- All data stored locally in browser IndexedDB
- Single HTML file architecture (~17,000 lines)
- Japanese primary language with extensive feature set

## Core Architecture

### File Structure
- `index.html` - Complete application (HTML, CSS, JavaScript in one file)
- `sw.js` - Service Worker for PWA caching and offline functionality
- `manifest.json` - PWA manifest defining app properties
- `marked.js` - Third-party Markdown rendering library (MIT License)

### Single-Page Application Design

The entire application logic resides in `index.html` within `<script>` tags. The code is organized into several major modules/namespaces:

1. **State Management** - Global `state` object holding:
   - Current session messages
   - All sessions (stored in IndexedDB)
   - Settings and UI state
   - API configurations

2. **Database Layer (`dbUtils`)** - IndexedDB operations:
   - Sessions CRUD operations
   - Settings persistence
   - Data export/import

3. **API Communication (`apiModule`)** - Handles requests to:
   - Google Gemini API
   - DeepSeek API
   - Anthropic Claude API
   - OpenAI API
   - xAI (Grok) API
   - LLM Aggregator endpoints (OpenRouter, Together.ai, Fireworks, etc.)
   - "Dummy AI" (returns empty responses for manual session construction)

4. **UI Utilities (`uiUtils`)** - DOM manipulation, rendering, theming

5. **Event Handlers (`eventHandlers`)** - User interaction management

6. **Streaming Module** - Server-Sent Events (SSE) handling for real-time AI responses

### Data Flow

1. User input → Event handler → API request builder
2. API module sends request with streaming enabled
3. Response chunks processed and rendered in real-time
4. Complete message saved to IndexedDB
5. UI updates to reflect new state

### Multi-API Support

Each API provider has:
- Dedicated configuration section in settings
- Provider-specific system prompts
- Provider-specific parameters (temperature, max tokens, etc.)
- Model selection dropdown
- Optional API key management (supports multiple keys per provider)

## Key Features

### Session Management
- Multiple chat sessions with independent histories
- Session linking ("AI間会話モード") - enables AI-to-AI conversations by linking two sessions
- Session import/export as JSON text files
- Bulk operations (export all, import all, delete all)

### Twin-Engine (Experimental)
- Dual-API architecture: one API for conversation, another for summarization
- Automatic or manual summarization of conversation history
- Reduces token usage by sending summaries instead of full history
- Configurable turn threshold for when summarization begins

### Proofreading Feature ("校正機能")
- Text replacement/correction functionality
- Can be used for automatic text substitutions during AI responses
- Enabled via settings → advanced options

### Advanced UI Customization
- Extensive theming (multiple color schemes including pastels)
- Opacity controls for bubbles, header/footer, overlays
- Font size adjustments for messages, code blocks, thought summaries
- Background image support
- Chat icon/avatar system (SNS-style)
- Message collapse/expand functionality

### Message Handling
- Markdown rendering with syntax highlighting
- Mermaid diagram support
- Image display in markdown
- Code block copy buttons
- Attachment support (files and images)
- Message editing, retrying, duplication
- Clipboard stack for accumulating copied text

### Thought Process Rendering
DeepSeek and some models support "thinking" mode where internal reasoning is displayed separately from the final response.

## Development

### No Build Process
This is a pure static site. To develop:

1. **Local Development (Windows):**
   - Run `__winlocal.bat` to start local HTTP server
   - Access via `localhost` in browser

2. **Deployment:**
   - Push to GitHub Pages
   - No compilation or bundling needed
   - Service Worker handles caching

### Testing
Always test with **free models first** before using paid APIs to avoid unexpected token consumption.

Recommended free models for testing:
- Gemini Flash (with API key)
- OpenRouter free models
- Dummy AI (returns empty responses, costs nothing)

### Key Settings Locations

Settings are stored in `state.settings` and persisted to IndexedDB:

- API keys: Settings screen → respective provider sections
- System prompts: Common prompt (all APIs) + provider-specific prompts
- Parameters: Temperature, max tokens, top-p, etc. (varies by provider)
- UI toggles: Extensive options in "その他設定" (Other Settings) and "挙動調整" (Behavior Adjustments)

### State Persistence

- **IndexedDB stores:** Sessions, settings, API configurations
- **LocalStorage:** Not used (everything in IndexedDB)
- **Session restoration:** Automatic on app load
- **Cache management:** Service Worker with manual update option in settings

## Important Implementation Notes

### API Security
- All API keys stored locally in browser (IndexedDB)
- No server-side storage or transmission except to respective AI providers
- Content Security Policy restricts external domains (whitelist approach)
- XSS protection via DOMPurify sanitization

### Streaming Implementation
Uses Server-Sent Events (SSE) with custom parsing:
- Handles different response formats per provider
- Real-time text rendering with typing effect
- Error recovery and retry logic
- Separate handling for "thinking" vs. "response" content

### Error Handling
- Global error handler with auto-recovery
- If 3 JavaScript errors occur within 1 minute, app auto-clears cache and reloads
- Watchdog system for monitoring app health

### Known Limitations
- No file/image analysis for non-Gemini APIs (except models that support it)
- No internet search integration (Gemini's search is provider-side)
- Claude API: No web fetching feature implemented
- OpenAI API: No internal reasoning flag equivalent to thinking mode
- DeepSeek: Many parameters not supported (see README)

## Modifying the Code

When making changes:

1. **Find the right section:** Use browser DevTools to inspect element IDs and event handlers
2. **Understand the flow:** Most features follow: Settings → State → API → UI rendering
3. **Test incrementally:** With free models or Dummy AI
4. **Clear cache:** Use settings panel to force cache update, or update `CACHE_NAME` in `sw.js`

### Common Modification Points

- **Add new API provider:** Extend `apiModule` with new provider handler, add UI controls in settings
- **UI changes:** Modify CSS variables in `:root` or add new elements in HTML sections
- **New features:** Add to appropriate namespace (uiUtils, eventHandlers, etc.)
- **Settings:** Add to `state.settings` default object, create UI controls, add save/load logic

## Deployment Notes

**GitHub Pages URL:** https://titan823.github.io/geminipwa/#chat

Service Worker caching means users may not see updates immediately. To force update:
- Increment `CACHE_NAME` in `sw.js`
- Users can manually update via settings → cache management
- Auto-update triggers on error threshold (3 errors/minute)

## License

MIT License (inherited from fork source: https://github.com/ona-oni/geminipwa)
