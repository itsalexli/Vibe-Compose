# 🎵 Vibe Compose

**Vibe Compose** is an interactive web-based music composition tool that combines the power of AI with standard ABC notation. It allows users to generate, edit, and visualize sheet music in real-time.

# Demo Video

[![Watch the video](https://img.youtube.com/vi/hml_W2O7lKw/hqdefault.jpg "YouTube demo")](https://www.youtube.com/watch?v=hml_W2O7lKw)


## ✨ Features

- **🤖 AI Composition Assistant**: Integrated chatbot powered by **DeepSeek API** to generate music snippets or entire songs from natural language prompts (e.g., "Create a melancholic melody in D minor").
- **📝 Live ABC Editor**: Real-time editor for [ABC notation](https://abcnotation.com/), perfect for tweaking generated music or composing from scratch.
- **🎼 Instant Visualization**: Renders ABC notation into beautiful sheet music instantly using `abcjs`.
- **📄 PDF Export**: Export your compositions to high-quality PDF scores with a single click.
- **🎨 Modern UI**: Built with a sleek, dark-themed responsive design using Tailwind CSS v4.

## 🛠️ Tech Stack

- **Framework**: [Next.js 15](https://nextjs.org/) (App Router)
- **Library**: [React 19](https://react.dev/)
- **Styling**: [Tailwind CSS v4](https://tailwindcss.com/)
- **Music Rendering**: [abcjs](https://www.abcjs.net/)
- **AI Integration**: [OpenAI Node SDK](https://github.com/openai/openai-node) (configured for DeepSeek API)
- **Export**: `jspdf` & `html2canvas`
