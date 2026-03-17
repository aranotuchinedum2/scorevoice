# 🎻 ScoreVoice
> // This app was built by CeeJay for Chinedum Aranotu – 2026

[![Live Demo](https://img.shields.io/badge/Live-scorevoice.vercel.app-black?style=flat-square)](https://scorevoice.vercel.app)
[![Cloudflare Workers](https://img.shields.io/badge/API-Cloudflare_Workers-orange?style=flat-square)](https://workers.cloudflare.com)
[![Powered by Claude](https://img.shields.io/badge/AI-Claude_Sonnet-blueviolet?style=flat-square)](https://anthropic.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](./LICENSE)

**ScoreVoice** converts string orchestra sheet music into playable audio — directly in your browser. Drop a photo or PDF of any score, and the app reads every note using Claude's vision AI, then plays it back with a multi-voice string synthesizer. No plugins, no installs, no music theory required.

---

## ✨ Features

- **Photo & PDF support** — drag in a JPG, PNG, HEIC photograph or a multi-page PDF score
- **AI-powered notation reading** — Claude Sonnet reads every stave, measure, and note across all pages
- **Multi-voice string synthesis** — separate timbres for Violin I, Violin II, Viola, Cello, and Double Bass using Tone.js
- **Full transport controls** — Play, Pause, Stop with an accurate progress timer
- **Tempo control** — adjust BPM in real time before or between plays
- **Score metadata** — detects title, key signature, time signature, measure count, and page count
- **Note grid** — visual display of every extracted pitch
- **Zero setup for visitors** — API key is server-side only via Cloudflare Workers proxy; users just drop their score and hit play
- **Dark mode** — adapts automatically to system preference
- **PDF page preview** — renders up to 6 pages visually before transcription

---

## 🖥️ Demo

1. Visit [scorevoice.vercel.app](https://scorevoice.vercel.app)
2. Drop your sheet music (PDF or image)
3. Click **Transcribe & Play**
4. Hit ▶

---

## 🏗️ Architecture

```
Browser (Vercel)          Cloudflare Worker          Anthropic API
──────────────────        ─────────────────────       ──────────────────
index.html          →     scorevoice.workers.dev  →   claude-sonnet-4
  - PDF.js preview        - Injects API key           - Vision OMR
  - Tone.js synth   ←     - Forwards response   ←     - Returns JSON
  - Transport ctrl
```

The frontend never touches the Anthropic API key. All requests flow through a Cloudflare Worker that injects the key server-side. Visitors experience zero friction.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML / CSS / JS — no framework, no build step |
| PDF Rendering | PDF.js 4.2 (Cloudflare CDN) |
| Audio Synthesis | Tone.js 14.7 (PolySynth + Reverb + Filter per voice) |
| AI / OMR | Claude Sonnet 4 via Anthropic Messages API |
| API Proxy | Cloudflare Workers (serverless, 30s timeout, free tier) |
| Hosting | Vercel (static, free tier) |

---

## 📁 Project Structure

```
scorevoice/
├── index.html          # Full frontend app — UI, PDF preview, synth, player
├── worker.js           # Cloudflare Worker — secure API proxy
├── wrangler.toml       # Cloudflare Worker config
├── vercel.json         # Vercel deployment config
├── .gitignore
├── LICENSE
└── README.md
```

---

## 🚀 Running Locally

### Prerequisites
- Node.js 18+
- A free [Anthropic API key](https://console.anthropic.com)
- A free [Cloudflare account](https://cloudflare.com)

### 1. Clone the repo
```bash
git clone https://github.com/YOUR_USERNAME/scorevoice.git
cd scorevoice
```

### 2. Install Wrangler
```bash
npm install -g wrangler
wrangler login
```

### 3. Set your API key as a secret
```bash
wrangler secret put ANTHROPIC_API_KEY
# Paste your sk-ant-... key when prompted
```

### 4. Run the worker locally
```bash
wrangler dev
# Worker runs at http://localhost:8787
```

### 5. Update the fetch URL for local dev

In `index.html`, temporarily change:
```js
var res = await fetch('https://scorevoice.workers.dev', {
```
to:
```js
var res = await fetch('http://localhost:8787', {
```

### 6. Open index.html
Open `index.html` directly in your browser or use any static server:
```bash
npx serve .
```

---

## ☁️ Deploying

### Cloudflare Worker (API proxy)
```bash
wrangler deploy
wrangler secret put ANTHROPIC_API_KEY
```

### Vercel (frontend)
```bash
# Connect your GitHub repo at vercel.com/new
# Framework preset: Other
# Root directory: /
# No build command needed
```

Or via CLI:
```bash
npx vercel --prod
```

---

## 🔐 Security

- The Anthropic API key is **never in the frontend** — it lives exclusively in Cloudflare's encrypted environment variables
- No user data is logged or stored — files are read client-side via FileReader and sent directly to the AI, then discarded
- CORS is locked to POST requests only on the worker

---

## 🎼 How the OMR Works

ScoreVoice uses Claude's vision capability as an Optical Music Recognition (OMR) engine. When you upload a score:

1. The file is base64-encoded in the browser
2. Sent to Claude as either an `image` block (photos) or a `document` block (PDFs — all pages in one call)
3. Claude returns a structured JSON object containing every voice, measure, and note in scientific pitch notation (`C4`, `F#5`, `Bb3`) with Tone.js duration strings (`4n`, `8n`, `d2n`)
4. The app builds a Tone.js Transport schedule from the JSON and fires each note at the correct beat time
5. Each voice gets its own PolySynth with a tuned lowpass filter and reverb to approximate the instrument's timbre

---

## 🧩 Supported Instruments

| Voice | Synth Profile |
|---|---|
| Violin I | Sawtooth, 4200Hz cutoff, fast attack |
| Violin II | Sawtooth, 3800Hz cutoff |
| Viola | Sawtooth, 2800Hz cutoff, mid attack |
| Cello | Sawtooth, 2000Hz cutoff, slower attack |
| Double Bass | Sawtooth, 1400Hz cutoff, slowest attack |
| Melody (fallback) | Sawtooth, 3500Hz cutoff |

---

## ⚙️ Known Limitations

- Synthesis is approximated — not sampled audio. Complex articulations (tremolo, pizzicato, slurs) are not yet differentiated
- Handwritten scores work but produce less accurate results than typeset PDFs
- PDFs over ~32 pages may exceed Claude's context window; a page range selector is planned
- Tempo changes mid-score are not yet supported

---

## 🗺️ Roadmap

- [ ] MIDI export (download `.mid` from extracted notes)
- [ ] Page range selector for large PDFs
- [ ] Per-voice mute/solo toggles
- [ ] Image contrast enhancement for handwritten scores
- [ ] Piano / woodwind instrument profiles

---

## 📄 License

MIT — see [LICENSE](./LICENSE)

---

## 🙏 Credits

Built with [Claude](https://anthropic.com) · [Tone.js](https://tonejs.github.io) · [PDF.js](https://mozilla.github.io/pdf.js) · [Cloudflare Workers](https://workers.cloudflare.com) · [Vercel](https://vercel.com)

---

*// This app was built by CeeJay for Chinedum Aranotu – 2026*