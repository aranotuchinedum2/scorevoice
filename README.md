# ScoreVoice 🎻
// This app was built by CeeJay for Chinedum Aranotu – 2026

Convert string orchestra sheet music photos into playable audio using Claude Vision AI + Tone.js synthesis.

## Run
Open index.html in any browser — no build step needed.

## Deploy
[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com)
- `vercel --prod` from project root

## Stack
- Claude Vision API (OMR)
- Tone.js 14.7 (string synthesis)
- Vanilla HTML/CSS/JS
```

---

## 🐛 DEBUG REPORT

| Issue | Fix |
|---|---|
| Claude wraps JSON in backticks despite instructions | Regex strip: `/```(?:json)?\s*([\s\S]+?)\s*```/` before parse |
| Tone.js needs user gesture to start | `await Tone.start()` inside click handler |
| Flat note names (Bb4, Eb3) | Tone.js accepts standard flat notation natively — no conversion needed |
| Sawtooth sounds harsh | Chained `Tone.Filter` lowpass per voice instrument + reverb |
| Different instruments same timbre | Per-voice configs: cutoff freq, attack, release mapped to violin/viola/cello/bass |
| API image too large | FileReader reads as-is — Claude handles up to ~5MB base64 cleanly |
| PolySynth memory leak | `synths` object rebuilt with `.dispose()` on rebuild |

---

## ✅ SIGNATURE CONFIRM
```
// This app was built by CeeJay for Chinedum Aranotu – 2026