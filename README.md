# 🎻 ScoreVoice
> // This app was built by CeeJay for Chinedum Aranotu – 2026

[![Live Demo](https://img.shields.io/badge/Live-scorevoice.vercel.app-black?style=flat-square)](https://scorevoice.vercel.app)
[![Cloudflare Workers](https://img.shields.io/badge/API-Cloudflare_Workers-orange?style=flat-square)](https://workers.cloudflare.com)
[![Powered by Claude](https://img.shields.io/badge/AI-Claude_Sonnet_4-blueviolet?style=flat-square)](https://anthropic.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](./LICENSE)

**ScoreVoice** converts string orchestra sheet music into playable audio — directly in your browser. Drop a photo or PDF of any score, and the app reads every note using Claude's vision AI, then plays it back with a multi-voice string synthesizer. Export to MIDI, mute or solo individual instruments, and import directly into GarageBand or MuseScore. No plugins. No installs. No music theory required.

---

## ✨ Features

- **Photo & PDF support** — drag in a JPG, PNG, HEIC photograph or a multi-page PDF score
- **AI-powered notation reading** — Claude Sonnet 4 reads every stave, measure, and note across all pages
- **Multi-voice string synthesis** — separate timbres for Violin I, Violin II, Viola, Cello, and Double Bass via Tone.js
- **Full transport controls** — Play, Pause, Stop with an accurate real-time progress timer
- **Tempo control** — adjust BPM before or between plays
- **Mute / Solo per instrument** — isolate or silence any voice during playback; Clear Solo resets all at once
- **MIDI export** — download a `.mid` file of the full transcription, one track per instrument, ready for GarageBand, MuseScore, Sibelius, or any DAW
- **Deterministic transcription** — `temperature: 0` ensures the same score produces consistent results every run
- **JSON repair** — auto-closes truncated AI responses so large scores never break the parser
- **Zero setup for visitors** — API key is server-side only via Cloudflare Workers; users just drop their score and hit play
- **Dark mode** — adapts automatically to system preference
- **PDF page preview** — renders up to 6 pages visually before transcription
- **Score metadata** — detects title, key signature, time signature, measure count, and page count

---

## 🖥️ Demo

1. Visit [scorevoice-eta.vercel.app](https://scorevoice-eta.vercel.app)
2. Drop your sheet music (PDF or image)
3. Click **Transcribe & Play**
4. Hit ▶ — or export the MIDI and open it in GarageBand

---

## 🏗️ Architecture

```
Browser (Vercel)              Cloudflare Worker             Anthropic API
──────────────────────        ──────────────────────        ──────────────────
index.html              →     scorevoice.workers.dev   →    claude-sonnet-4
  - PDF.js preview            - Injects API key              - Vision OMR
  - Tone.js synth       ←     - Forwards response      ←     - Returns JSON
  - MIDI encoder
  - Mute / Solo state
  - Transport controls
```

The frontend never touches the Anthropic API key. All requests flow through a Cloudflare Worker that injects the key server-side. Visitors experience zero friction and zero configuration.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML / CSS / JS — no framework, no build step |
| PDF Rendering | PDF.js 4.2 (Cloudflare CDN) |
| Audio Synthesis | Tone.js 14.7 (PolySynth + Reverb + Filter per voice) |
| MIDI Encoding | Pure JS binary encoder — no library, no dependency |
| AI / OMR | Claude Sonnet 4 via Anthropic Messages API |
| API Proxy | Cloudflare Workers (serverless, 30s timeout, free tier) |
| Hosting | Vercel (static, free tier) |

---

## 📁 Project Structure

```
scorevoice/
├── index.html          # Full frontend — UI, PDF preview, synth, MIDI, mute/solo
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
var res = await fetch('https://YOUR_WORKER.workers.dev', {
```
to:
```js
var res = await fetch('http://localhost:8787', {
```

### 6. Open index.html
```bash
npx serve .
# or just open index.html directly in your browser
```

---

## ☁️ Deploying

### Cloudflare Worker (API proxy)
```bash
wrangler deploy
wrangler secret put ANTHROPIC_API_KEY
```

### Vercel (frontend)
Connect your GitHub repo at [vercel.com/new](https://vercel.com/new):
- Framework preset: **Other**
- Root directory: `/`
- No build command needed

Or via CLI:
```bash
npx vercel --prod
```

---

## 🔐 Security

- The Anthropic API key is **never in the frontend** — it lives exclusively in Cloudflare's encrypted environment variables
- No user data is logged or stored — files are read client-side via FileReader, sent to the AI, then discarded
- CORS is restricted to POST requests only on the worker
- The Cloudflare Worker sits between the browser and Anthropic — your key is never exposed

---

## 🎼 How the OMR Works

ScoreVoice uses Claude's vision capability as an Optical Music Recognition (OMR) engine:

1. The file is base64-encoded in the browser
2. Sent to Claude as either an `image` block (photos) or a `document` block (PDFs — all pages in one call)
3. `temperature: 0` ensures deterministic, consistent extraction every run
4. Claude returns structured JSON: every voice, measure, and note in scientific pitch notation (`C4`, `F#5`, `Bb3`) with Tone.js duration strings (`4n`, `8n`, `d2n`)
5. A JSON repair function closes any truncated brackets from large scores before parsing
6. The app builds a `Tone.Transport` schedule and fires each note at the correct beat offset
7. Each voice gets its own `PolySynth` with a tuned lowpass filter and shared reverb

---

## 🎹 MIDI Export

After transcription, click **Export MIDI** in the player header. ScoreVoice encodes a standard `.mid` file in pure JavaScript:

- Format 1 MIDI (multi-track) — one track per instrument voice
- General MIDI program assignments: Violin=40, Viola=41, Cello=42, Bass=43
- Correct variable-length delta time encoding
- Tempo track embedded from the detected BPM
- Compatible with GarageBand, MuseScore, Sibelius, Logic Pro, Ableton, and any DAW

---

## 🎚️ Mute / Solo

Each instrument row in the player has two buttons:

| Button | Behaviour |
|---|---|
| **M** | Mutes that voice — silent during playback, others unaffected |
| **S** | Solos that voice — only soloed voices play (stack multiple solos) |
| **Clear Solo** | Resets all solo state, returns to full orchestra |

Changing mute/solo stops playback — hit play again to hear the new mix. Solo always overrides mute when any solo is active.

---

## 🧩 Instrument Profiles

| Voice | MIDI Program | Synth Cutoff | Character |
|---|---|---|---|
| Violin I | 40 | 4200Hz | Bright, fast attack |
| Violin II | 40 | 3800Hz | Slightly warmer |
| Viola | 41 | 2800Hz | Mid-range, fuller |
| Cello | 42 | 2000Hz | Warm, slower attack |
| Double Bass | 43 | 1400Hz | Deep, slowest attack |
| Melody (fallback) | 40 | 3500Hz | General string |

---

## ⚙️ Known Limitations

- Synthesis is approximated via sawtooth oscillators — not sampled audio. Articulations like tremolo, pizzicato, and slurs are not yet differentiated
- Handwritten scores work but produce less accurate results than typeset PDFs
- PDFs over ~32 pages may exceed Claude's context window — a page range selector is planned
- Tempo changes mid-score are not yet supported
- Changing mute/solo during playback requires a stop and replay to take effect

---

## 🗺️ Roadmap

- [ ] Mobile PWA — install on iPhone/Android home screen
- [ ] Page range selector for large PDFs
- [ ] Image contrast enhancement for handwritten scores
- [ ] Piano / woodwind instrument profiles
- [ ] Tempo change support mid-score
- [ ] Live mute/solo without stopping transport

---

## 📄 License

MIT — see [LICENSE](./LICENSE)

---

## 🙏 Credits

Built with [Claude](https://anthropic.com) · [Tone.js](https://tonejs.github.io) · [PDF.js](https://mozilla.github.io/pdf.js) · [Cloudflare Workers](https://workers.cloudflare.com) · [Vercel](https://vercel.com)

---

*// This app was built by CeeJay for Chinedum Aranotu – 2026*
