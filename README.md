# VibeCompose

**VibeCompose** is a browser-based music composition tool that connects AI
generation with ABC notation. Write, edit, and render sheet music in real-time.

## Demo

[![Watch the video](https://img.youtube.com/vi/hml_W2O7lKw/hqdefault.jpg)](https://www.youtube.com/watch?v=hml_W2O7lKw)

## Features
 
* **AI Composition**: Chat-based interface powered by the DeepSeek API. Describe what you want ("melancholic melody in D minor") and get ABC notation back.
* **Live ABC Editor**: Edit [ABC notation](https://abcnotation.com/) directly, whether you're tweaking AI output or writing from scratch.
* **Sheet Music Rendering**: Notation renders to sheet music in real-time via `abcjs`.
* **PDF Export**: Export your composition as a PDF score.
* **UI**: Dark theme, responsive, built with Tailwind CSS v4.
## Tech Stack
 
* **Framework**: [Next.js 15](https://nextjs.org/) (App Router)
* **Library**: [React 19](https://react.dev/)
* **Styling**: [Tailwind CSS v4](https://tailwindcss.com/)
* **Music Rendering**: [abcjs](https://www.abcjs.net/)
* **AI Integration**: [OpenAI Node SDK](https://github.com/openai/openai-node) (pointed at DeepSeek API)
* **Export**: `jspdf` + `html2canvas`
## How It Works
 
1. **Prompt Construction Layer (`chatbot.tsx`)**
   Each request includes the previous ABC output as context, so the AI refines existing music rather than generating from scratch. The prompt enforces strict output constraints — valid ABC notation, correct headers, and line length limits — and runs basic input validation before hitting the DeepSeek API via the OpenAI Node SDK.
```mermaid
flowchart TD
    classDef process fill:#1e293b,stroke:#475569,color:#f1f5f9
    classDef guard fill:#1e1b4b,stroke:#818cf8,color:#e0e7ff
    classDef io fill:#0f172a,stroke:#6366f1,color:#a5b4fc
 
    A1([User text prompt]):::io
    A2[Inject previous ABC output as context]:::process
    A3["Enforce: ABC only · ≤8 bars/line · valid syntax · auto-title"]:::process
    A4{Empty input? No API key?}:::guard
    A5[/"Structured prompt → DeepSeek API (deepseek-chat via OpenAI SDK)"/]:::io
 
    A1 --> A2 --> A3 --> A4
    A4 -- Fail --> A1
    A4 -- Pass --> A5
```
 
2. **State Management (`page.tsx`)**
   The root component keeps the raw AI response and the live editor notation as separate state values. A guard prevents a blank or errored AI response from overwriting what's in the editor. Manual edits and AI-generated output both flow into the same notation state, so the rest of the app doesn't care which path produced it.
```mermaid
flowchart TD
    classDef process fill:#1e293b,stroke:#475569,color:#f1f5f9
    classDef guard fill:#1e1b4b,stroke:#818cf8,color:#e0e7ff
    classDef io fill:#0f172a,stroke:#6366f1,color:#a5b4fc
 
    B1([Raw AI response → latestAnswer state]):::io
    B2{"latestAnswer.trim().length > 0?"}:::guard
    B3[["abc state — broadcast to all downstream layers"]]:::process
    B4([Manual user edit — Path B]):::io
 
    B1 --> B2
    B2 -- Yes --> B3
    B2 -- No: drop --> B2
    B4 -.->|"Path B: manual edit"| B3
```
 
3. **Syntax-Highlighted Editor (`ABCEditor.tsx`)**
   The editor layers a syntax-highlighted `div` beneath a transparent `textarea`, keeping them scroll-synced. The header lines render as plain text; everything below gets tokenized — notes and bar lines are each assigned a color and wrapped in spans. The result is a lightweight code-editor feel without pulling in a full editor library.
```mermaid
flowchart TD
    classDef process fill:#1e293b,stroke:#475569,color:#f1f5f9
    classDef io fill:#0f172a,stroke:#6366f1,color:#a5b4fc
 
    C0([abc state]):::io
    C1["Lines 0–4: header block — HTML escape only"]:::process
    C2["Lines 5+: character tokenizer → noteColors lookup → colored spans"]:::process
    C3["Highlight div + transparent textarea, scroll-synced via passive listeners"]:::process
    C4([User edits → Path B back to abc state]):::io
 
    C0 --> C1 & C2
    C1 & C2 --> C3
    C3 --> C4
```
 
4. **Rendering & SVG Normalization (`sheet.tsx`)**
   Every notation change triggers a full re-render via `abcjs`. After rendering, a normalization pass forces all SVG elements to black regardless of the current theme, so the output is export-ready at all times. Audio playback stays in sync by binding directly to the same render output.
```mermaid
flowchart TD
    classDef process fill:#1e293b,stroke:#475569,color:#f1f5f9
    classDef io fill:#0f172a,stroke:#6366f1,color:#a5b4fc
 
    D0([abc state]):::io
    D1["ABCJS.renderAbc() → visualObj"]:::process
    D2["text fill #000"]:::process
    D3["stroked paths → stroke #000"]:::process
    D4["filled shapes → fill #000"]:::process
    D5["SynthController.setTune(visualObj) → audio playback controls"]:::process
    D6(["Normalized SVG (export-ready) + Audio SynthController"]):::io
 
    D0 --> D1
    D1 --> D2 & D3 & D4
    D2 & D3 & D4 --> D5
    D5 --> D6
```
 
5. **PDF Export (`ExportPDFButton.tsx`)**
   Export runs entirely in the browser. The rendered SVG gets serialized, rasterized at 3× resolution for high-DPI output, then assembled into a PDF via `jsPDF`. Orientation is set automatically based on the sheet dimensions.
```mermaid
flowchart TD
    classDef process fill:#1e293b,stroke:#475569,color:#f1f5f9
    classDef io fill:#0f172a,stroke:#6366f1,color:#a5b4fc
 
    E1(["Normalized SVG — exportRef"]):::io
    E2["SVG extraction + getBoundingClientRect()"]:::process
    E3["XMLSerializer → Blob → Object URL"]:::process
    E4["3× upscale canvas rasterization"]:::process
    E5["toDataURL('image/png', 1.0) + revoke Object URL"]:::process
    E6["jsPDF assembly (auto-orientation) → pdf.save()"]:::process
    E7(["Downloaded PDF file"]):::io
 
    E1 --> E2 --> E3 --> E4 --> E5 --> E6 --> E7
```
 
