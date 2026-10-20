# Right-side Image Panel Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans for inline execution.

**Goal:** Add an image area inside the paper at right-middle, 25vw wide on desktop, with free text-keyword recommendations, manual web search, local photo selection and source/license attribution.

**Architecture:** Keep application HTML/CSS/JS in index.html. Search/recommend/upload controls and results stay outside the paper in imagePanel. Only the selected image is shown inside paperViewport in a separate paperPhoto frame, centered vertically at its right edge and 25vw wide on desktop, with a reserved text column. Narrow screens place search tools below the desk while keeping the selected photo inside the paper. Wikimedia Commons supplies bitmap thumbnails and metadata through unauthenticated CORS. Korean query terms are mapped via Korean Wikipedia's English interlanguage title when available; there is no paid API or new model.

**Spec:** User requested internet image recommendations matching writing and manual image search/insertion; approved right-middle placement and width equal to one-quarter of the viewport. No changes to already confirmed correction or sound behavior. Source text recommendation is keyword-based, not semantic AI; only the editable query goes to Wikimedia on explicit button press. Local uploads stay in memory and are not sent to a server.

## Global Constraints
- Retain the single-file application, existing Hangul typing, lamp, effects, audio and AI correction/undo.
- No keys, paid services, image generation, installed server or build step.
- Preview selected images with source, creator and license; use safe URLs and plain text for remote metadata.
- Never overwrite the current photo on failed search/load; reject stale results and stale image loads.
- Local raster images only (JPEG/PNG/WebP/GIF), at most 10MiB; release Blob URLs on replacement/reset.
- Search/control keyboard events must not type into the paper.
- Do not commit or upload before user browser confirmation.

## Review Focus
- Searching Korean and English, no result, service failure, timeouts and rapid repeated requests.
- Whole writing is not transmitted; query extraction handles particles, empty text and meaningful single-character Korean nouns.
- Untrusted attribution HTML/URLs cannot execute script; license and author stay visible after selecting an image.
- Selected local photo survives network failures, replacement races cannot restore old photos, reset cancels pending work.
- paperPhoto must be inside paperViewport and contain no form controls; imagePanel with search/recommend/upload controls must remain outside the paper. The selected image stays within paper bounds and a separate text column is reserved, including on narrow screens.

## Task 1: Image workflow
**Files:** index.html; README.md feature notes; deterministic/live Chromium tests in external scratch directory.
**Interfaces:** extractImageKeyword(text): string; searchPanelImages(query): Promise<void>; selectPanelImage(candidate, localURL?): void; clearPanelImage(): void; resetImagePanel(): void.

- [x] Write failing browser tests for layout, query isolation, recommendation, search/selection/attribution, upload, failure preservation, reset and stale responses.
- [x] Run tests against existing app and observe missing-feature failure.
- [x] Implement inline image panel, responsive layout, CORS search, plain-text attribution, Blob-based local uploads and input isolation.
- [x] Run deterministic image tests, real Wikimedia query and actual image decoding from file:// and HTTP, and old correction/audio regressions.
- [x] Obtain a fresh read-only review, resolve important issues with RED→GREEN tests, check git diff and ask user to verify before committing.
