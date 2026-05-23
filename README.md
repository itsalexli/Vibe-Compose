# VibeCompose

**VibeCompose** is a browser-based music composition tool that connects AI
generation with ABC notation. Write, edit, and render sheet music in real-time.

## Demo

[![Watch the video](https://img.youtube.com/vi/hml_W2O7lKw/hqdefault.jpg)](https://www.youtube.com/watch?v=hml_W2O7lKw)

## Features

- **AI Composition**: Chat-based interface powered by the **DeepSeek API**.
  Describe what you want ("melancholic melody in D minor") and get ABC notation
  back.
- **Live ABC Editor**: Edit [ABC notation](https://abcnotation.com/) directly,
  whether you're tweaking AI output or writing from scratch.
- **Sheet Music Rendering**: Notation renders to sheet music in real-time via
  `abcjs`.
- **PDF Export**: Export your composition as a PDF score.
- **UI**: Dark theme, responsive, built with Tailwind CSS v4.

## Tech Stack

- **Framework**: [Next.js 15](https://nextjs.org/) (App Router)
- **Library**: [React 19](https://react.dev/)
- **Styling**: [Tailwind CSS v4](https://tailwindcss.com/)
- **Music Rendering**: [abcjs](https://www.abcjs.net/)
- **AI Integration**: [OpenAI Node SDK](https://github.com/openai/openai-node)
  (pointed at DeepSeek API)
- **Export**: `jspdf` + `html2canvas`

## Layer Breakdown

### 1. Prompt Construction & Constraint Layer (`chatbot.tsx`)
Before each API call, a structured prompt injects the **previous ABC output as
context** (`prevResponse.current`), so the AI refines existing music rather than
generating from scratch each time. The prompt enforces four hard output
constraints: ABC notation only, auto-generate/update the `T:` header, hard-wrap
at ≤ 8 bars per line, and valid `abcjs` syntax. Two pre-flight guards run before
the request: an empty-input check and an env var check for
`NEXT_PUBLIC_OPENAI_API_KEY`. The API client uses the OpenAI Node SDK with
`baseURL` overridden to `https://api.deepseek.com`.

### 2. Response Propagation & State Guard Layer (`page.tsx`)
The root component manages two decoupled state values: `latestAnswer` (raw AI
response) and `abc` (the live notation consumed by downstream components). A
`useEffect` bridges them with a non-empty guard — `latestAnswer.trim().length >
0` — so a blank or errored response never overwrites the editor. Both the AI
path and the manual edit path converge on `abc`.

### 3. ABC Tokenizer & Dual-Layer Editor (`ABCEditor.tsx`)
The editor runs two absolutely-positioned layers with synchronized scroll: a
`div` highlight layer rendered via `dangerouslySetInnerHTML`
(`pointer-events: none`) sitting beneath a transparent `textarea`.
`renderHighlightedContent()` does a character-level tokenization pass — lines
0–4 (the `X:`, `T:`, `M:`, `L:`, `K:` header block) are HTML-escaped only;
lines 5+ are tokenized against a `noteColors` lookup table that wraps notes
`A–G` and bar lines `|` in colored `<span>` elements. Scroll sync is handled by
passive `scroll` and `input` listeners mirroring `scrollTop`/`scrollLeft`
between layers.

### 4. abcjs Rendering & SVG Normalization Pipeline (`sheet.tsx`)
Each `abc` change triggers a full re-render via `ABCJS.renderAbc()`, returning a
`visualObj` used to initialize audio. A **three-pass SVG color normalization**
then runs on the live DOM — targeting text elements, stroked paths/lines, and
filled shapes separately — forcing everything to black regardless of theme. This
keeps the SVG export-ready without a separate processing step. Audio playback is
bound by passing `visualObj` directly to `SynthController.setTune()`.

### 5. PDF Export Pipeline (`ExportPDFButton.tsx`)
Export is entirely browser-side, running five stages: SVG extraction from the
shared `exportRef` → `XMLSerializer` serialization to a Blob URL → 3× upscale
canvas rasterization for high-DPI output → `toDataURL("image/png", 1.0)`
encoding → `jsPDF` assembly with `unit: "px"` and auto-orientation (`landscape`
if `width > height`).
