# Browser Autocorrect Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** On user Enter, correct the completed line using the tested free browser model and provide correction-only undo.

**Architecture:** Keep application CSS and JS inline in index.html. Replace the temporary dictionary with a lazily loaded, pinned Transformers.js module and the pinned public ET5 q8 model. Use a serial async correction queue and line-index/version guards so responses cannot overwrite newer input; preserve original text and undo history.

**Tech Stack:** Static HTML/JS, Transformers.js 4.3.1, ONNX WASM in a Web Worker created from inline source, hueyjeong/et5-typos-corrector-onnx-q8 revision 7f6bbb322e691bbba89948fd92c446a28332dff1.

**Spec:** User approved automatic Enter correction with undo after testing the model. No API fees, API keys, model installation or inference server. Known wrong model outputs are accepted, but must be reversible. Sound and image changes are out of scope.

## Global Constraints
- Existing project: single HTML application, preserve Hangul automata, lamp, ink, fonts, sound and demo.
- Approved exception to external-library restriction: the tested browser correction library and model, with visible upstream attribution.
- Keep original on failure; show download/ready/busy/error status and retry control.
- Model initialization is lazy on first nonblank user Enter. Never block the main UI with CPU inference.
- Do not correct demo typing, empty lines or lines above the conservative model input limit; show the latter as skipped.
- The app must still type if the CDN/model is unavailable.
- Do not commit, push or publish before user verifies the integrated page.

## Review Focus
- Late inference after reset/backspace cannot rewrite edited or replacement text.
- Subsequent typed/composing text must survive correction and undo.
- Sequential corrections and multiple undo operations target the right lines despite length changes.
- File URL and HTTP/static hosting paths must use public pinned assets, never scratch paths.
- Timeout, worker/load failure and invalid/empty model output preserve all text and remain retryable.

## Task 1: Integrated automatic correction and undo

**Files:** Modify index.html; keep browser integration tests in external scratch directory so the application gains no build/test installation requirement.

**Interfaces:** ensureCorrectionWorker(): Worker; queueLineCorrection(lineIndex, original): void; undoLastCorrection(): void; invalidateCorrections(): void. Worker protocol: load/progress/ready/correct/result/error with job ID, text and error payload. Main thread accepts results only if job generation and original line still match.

- [x] Write failing Chromium tests for the new UI, Enter correction, undo preserving next-line input, stale response rejection, serialization, reset cancellation, unchanged text, blank/long-line skips, retry and failure preservation. Use a transport-only fake Worker for deterministic queue cases and explicitly distinguish these from real inference checks.
- [x] Run tests against old dictionary implementation and observe feature-related failures.
- [x] Implement lazy browser worker, pinned assets, status/retry UI, automatic correction, correction-only undo and input/control isolation. Remove dictionary. Maintain version/line guards during deletion and reset.
- [x] Run the deterministic browser suite and all existing non-dictionary regressions.
- [x] Run the actual model in the integrated page via public CDN/Hugging Face, check exact observed model output then undo, typing while pending, and file:// and HTTP behavior. Record errors and any unavailable path honestly.
- [x] Inspect changes with git diff --check and request the user's browser confirmation. Commit remains blocked on that confirmation.
