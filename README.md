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
