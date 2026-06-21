# Saathi — Voice AI Classroom Companion 🌟

A hands-free, voice-first AI co-pilot for a Haryana government-school smart board.
Built for the **Connecting Dreams Foundation — Round 2 Technical Assignment (Option A)**.

The teacher (or a child) presses one big **BOLO / TALK** button and speaks a topic. Saathi
explains it simply **aloud + projected on the board** in **Hinglish, Hindi, or English**, runs
voice-triggered quizzes, and is built so **every** child — including children with special
needs — can take part.

> See `CONCEPT_NOTE.md` for the research and the *why* (children's, teachers', and spending
> challenges, and CDF mission alignment). This README covers the *how*.

---

## ✨ What it does (the two required capabilities)

1. **Live Concept Simplification** — say a concept ("photosynthesis samjhao", "जल चक्र",
   "why do things fall?"); Saathi speaks a simple explanation and projects a picture + title +
   chalk points on the board.
2. **Voice-Triggered Quizzing** — say "quiz lo on water cycle"; Saathi announces a question
   aloud and shows big tappable answer buttons with gentle, encouraging feedback.

Plus the cross-cutting features the brief asked for:
- **3 language modes:** Hinglish · हिंदी · English (switch anytime; current content re-renders).
- **Broken-language understanding:** infers intent from half-formed child speech; if unsure,
  offers **picture-choices** to clarify instead of guessing wrong. (Try the **🙋 Confused kid** button.)
- **Inclusion by default:** always-on subtitles, audio description for visuals, ALT text,
  **Calm Mode** (autism-friendly), bigger-text mode, and a type-instead fallback.

---

## 🧰 Tech stack

| Layer | Choice | Why |
|---|---|---|
| **Interface** | Single-file **HTML/CSS/JS** (no build step) | A smart board / cheap classroom PC needs something that opens in any browser and hosts as a static URL. Matches the brief's "simple web interface" intent. |
| **Speech-to-Text (STT)** | **Web Speech API** (`SpeechRecognition`), `hi-IN` for Hindi/Hinglish, `en-IN` for English | On-device, free, no audio leaves the browser. Falls back to a **type box** where unavailable (also an inclusion feature for non-verbal kids). |
| **Text-to-Speech (TTS)** | **Web Speech API** (`speechSynthesis`); picks `hi-IN` / `en-IN` voices; slower rate in Calm Mode | Free, offline-capable, voice-selectable per language. |
| **AI brain** | **Anthropic Claude** (`claude-sonnet-4-6`) via the Messages API, returning **structured JSON** | Generates simple, warm, India-grounded explanations, quizzes, and clarification options on demand. |
| **Offline content bank** | Curated trilingual JS dataset (6 topics + 3 quizzes + clarification flows) | Guarantees the **hosted demo URL works with no backend**, and gives an instant, zero-latency, zero-cost path for common topics. Live AI is used when reachable; the bank is the fallback. |
| **Fonts** | **Baloo 2** (display) + **Mukta** (body), both full-Devanagari Google Fonts | Rounded, child-friendly, and render Hindi + Latin cleanly. |

No frameworks, no localStorage, no tracking. One file: `index.html`.

---

## 🧠 Prompt design

Saathi's AI is governed by a strict **system prompt** that encodes the empathy and safety the
brief rewards. Key constraints (full text in `index.html` → `SYSTEM_PROMPT`):

- **Role:** a warm, patient companion for grades 1–8 in a Haryana government school; one teacher,
  40+ mixed-level kids, a smart board.
- **Simplicity guardrail:** vocabulary a 7-year-old knows; short sentences; **no jargon**; always
  encouraging; never makes a child feel wrong.
- **Localisation guardrail:** must answer in the exact requested register — `hinglish`
  (Hindi+English in Latin script, the way kids speak), `hindi` (simple Devanagari), or `english`
  (simple Indian-context English) — and use **village examples** (mango tree, tubewell, chapati, the sun).
- **Broken-language guardrail:** try hard to infer intent from incomplete speech; **if genuinely
  unsure, set `needs_clarification:true` and return 2–4 friendly picture-options** rather than guessing.
- **Structured output:** the model returns **only JSON** matching a fixed schema, so the UI can
  reliably drive the board, captions, quiz, and clarification chips:

```json
{
  "mode": "explain | quiz | clarify",
  "needs_clarification": false,
  "clarification_question": "",
  "options": [{ "emoji": "🌧️", "label": "Baarish kaise hoti hai?" }],
  "spoken_text": "what Saathi says aloud",
  "board_title": "short board heading",
  "board_points": ["short chalk point 1", "short chalk point 2"],
  "visual_emoji": "🌿",
  "alt_text": "one-line description of the picture for a blind child",
  "quiz": { "question": "", "choices": [], "answer_index": 0, "explanation": "" }
}
```

The per-request user message passes `language`, the teacher-selected `mode`, and the raw
transcript, e.g. `child/teacher said: "wo paani upar wala cloud nahi pata"` — and the model
decides whether to explain, quiz, or clarify.

**Production note — accuracy via RAG:** for a real rollout, wrap the same prompt with a
**retrieval layer over NCERT/SCERT textbook content** for the relevant grade, so explanations and
quiz answers stay strictly on-syllabus and verifiable. The JSON contract stays identical; only the
context grows. This is the "RAG or strict prompt guardrails for accuracy" path in the brief.

---

## 🌐 Localization

- **Full trilingual UI** — every label, button, hint, caption, and accessibility string is defined
  for `hinglish`, `hindi`, and `english` (`UI` object in `index.html`). Switching language
  re-renders the **current** board content into the new language, not just the chrome.
- **Trilingual content** — every offline topic and quiz carries `spoken`, `title`, `points`, and
  `alt` text in all three registers.
- **STT language codes** — `hi-IN` for Hindi/Hinglish (handles code-mixed speech and returns
  Devanagari for Hindi), `en-IN` for English. Keyword matching includes **both Latin and Devanagari**
  spellings so a child saying "jal chakra" or "जल चक्र" both work.
- **TTS voice selection** — scores installed voices and picks the most **Indian** one: a
  `hi-IN` (Hindi) voice for Hindi/Hinglish and an `en-IN` (Indian English) voice for English,
  preferring Google/natural voices, with graceful fallback.
- **Fluent Hindi pronunciation in Hinglish mode** — romanised Hinglish read by a TTS engine sounds
  wrong. So in Hinglish mode Saathi **displays the easy Latin caption but *speaks* the Devanagari
  (Hindi) version** of the same line through a Hindi voice — so Hindi words come out fluently, like
  an Indian speaker, while the on-screen text stays child-readable. (The live-AI path returns a
  `spoken_devanagari` field for the same effect.)
- **Emoji are stripped before speaking**, so the voice never reads "fire"/"sun-with-face".
- **Known limitation:** pronunciation quality still depends on the device having an Indian
  `hi-IN` / `en-IN` voice installed (Chrome/Edge on Android, Windows, and ChromeOS ship them;
  some Linux setups don't). Captions always carry the exact words regardless.

---

## ♿ Accessibility & inclusion (built-in, not bolted-on)

| Feature | What it does | For whom |
|---|---|---|
| **Subtitles / captions** | Large persistent caption bar of everything Saathi says (on by default) | Deaf / hard-of-hearing; early readers |
| **Hear-picture button** | Reads the visual's description aloud — and Saathi now **describes every picture automatically** after each explanation, and **reads every quiz option aloud**, since it's hands-free | Low-vision / blind learners |
| **ALT-text switch** | Shows written picture descriptions permanently | Low-vision; screen-reader users |
| **Calm Mode** | Muted low-contrast palette, **no animation/flashing**, **no sound effects**, slower speech | Autistic & sensory-sensitive learners |
| **Bigger text** | Scales all type up | Low-vision learners |
| **Type-instead (always works)** | Enter a topic by keyboard. If the mic is blocked or unavailable, the **TALK button auto-opens this with a clear message**, so there's always a working path | Non-verbal children; noisy rooms; no-mic devices |
| **Big touch targets + visible focus rings** | ≥62px targets; keyboard-navigable | Motor difficulties; keyboard users |

The default palette (slate-green board, warm muted accents) was chosen to be **low-stimulation
even before Calm Mode**, and `prefers-reduced-motion` is respected automatically.

---

## 🎤 Microphone & true hands-free use

Saathi is voice-first. There are two things worth knowing:

**1. Why the mic may not work in an embedded preview.**
Browsers only allow microphone access via the Web Speech API when the page is (a) served over
`https://` or `localhost`, and (b) *not* blocked by an iframe's Permissions Policy. When the app is
shown **inside a sandboxed preview iframe** (e.g. an artifact viewer) that doesn't pass
`allow="microphone"`, the browser blocks the mic **before it can even prompt you** — so it looks
"broken". This is a sandbox limitation, **not** a real permission denial.

**Fix:** open `index.html` as its **own page** — host it on Netlify/Vercel/GitHub Pages (any
`https` URL) or run it on `localhost`, and open it in a normal browser tab. The first **BOLO** press
triggers a one-time "Allow microphone?" prompt → tap **Allow**. After that the permission is
remembered and it's fully hands-free. The app now **detects the exact reason** the mic failed
(sandbox-blocked / permission-denied / no-https / unsupported browser / no hardware) and shows a
plain-language fix instead of silently switching to typing.

**2. Hands-free wake-word mode (default ON).**
With the **🎙️ Hands-free** toggle on, the teacher presses **BOLO once** to start the session, then
just speaks — Saathi listens continuously and acts only when it hears the wake word **“Saathi …”**
(or a clear topic/quiz word), ignoring normal classroom chatter. It **pauses its own mic while it
talks** so it never hears itself, then resumes. Turn the toggle off for simple press-to-talk.
Typing remains only as a last-resort fallback for no-mic devices and non-verbal learners.

> Best browser support: **Chrome / Edge** (desktop or Android). The Web Speech *recognition* API is
> not available in Firefox and is limited in Safari; there, Saathi keeps the type-box fallback.

---

## 🚀 Run & deploy

### Try it locally
Just open `index.html` in Chrome or Edge (best Web Speech API support).
- Press **BOLO / TALK** and say a topic, or use **⌨ Type karein**.
- Try **Quiz** mode, the **🙋 Confused kid** button, the **♿** accessibility sheet, and the language pills.
- The badge shows **Live AI ready** (inside an API-enabled environment) or **Demo mode** (hosted statically — full UX still works via the content bank).

### Host the live URL (static)
Drop `index.html` on any static host — **Netlify, Vercel, GitHub Pages, Cloudflare Pages**.
It runs entirely in the browser; the offline content bank covers the demo with no server.

### Enable live AI (real answers to anything) — run the backend
The browser can't hold an API key, so a tiny backend (`server.js`, included) keeps your key safe and
proxies requests to Claude. It needs **only Node.js** — no `npm install`.

```bash
# 1. put your key in the environment
export ANTHROPIC_API_KEY=sk-ant-xxxxx      # Windows: set ANTHROPIC_API_KEY=sk-ant-xxxxx
# 2. start it (serves the app AND the AI proxy)
node server.js
# 3. open this in Chrome:
#    http://localhost:8787
```

Now the mic → `/api/ask` → Claude → board+voice loop is fully live, and the status badge flips to
**“Live AI (backend) ready.”** The flow: speech (or text) is transcribed, POSTed to `/api/ask` as
`{language, mode, text}`; the server adds your key, calls Claude, and returns the structured JSON;
the frontend renders it on the board and speaks it. `localhost` is a secure origin, so the
microphone works there too.

**Deploy it for real:** host `server.js` on any Node host (Render, Railway, Fly, a VPS) with the
`ANTHROPIC_API_KEY` env var set, or port the `/api/ask` handler to a serverless function
(Vercel/Netlify/Cloudflare). If you host the static `index.html` separately from the backend, set
`window.SAATHI_API = "https://your-backend/api/ask"` before the app script runs.

**Production accuracy (RAG):** inside `server.js`'s `/api/ask` handler, retrieve the relevant
NCERT/SCERT passage for the grade and prepend it to the user message — the JSON contract is
unchanged, so the frontend needs no edits.

---

## 🎬 Suggested 3-minute video walkthrough

1. **0:00–0:30 — The problem.** One teacher, 40 mixed-level kids, an under-used smart board; ASER:
   most can't read at grade level. (See concept note.)
2. **0:30–1:15 — Explain.** Press TALK → "photosynthesis samjhao" → board draws it; show captions
   + Hear-picture. Switch to हिंदी, re-ask; switch to English.
3. **1:15–1:55 — Quiz.** "quiz lo on water cycle" → answer wrong then right → gentle encouragement.
4. **1:55–2:30 — Every child.** Open ♿ → toggle **Calm Mode** (colours soften, motion stops) →
   show ALT text + bigger text + type-instead.
5. **2:30–3:00 — Broken speech.** Hit **🙋 Confused kid** → Saathi offers picture-choices → tap one
   → it explains. Close on: *software on a screen the school already owns — for every child.*

---

## 📁 Files
```
saathi/
├── index.html        # the working prototype (the live URL)
├── server.js         # tiny zero-dependency backend (holds API key, /api/ask proxy)
├── README.md         # this file (tech stack · prompt design · localization)
└── CONCEPT_NOTE.md   # research + how it solves the 3 problems + CDF alignment
```

---

*Saathi — आपका क्लासरूम साथी. Built so the quietest child in the back row still gets a turn.*
