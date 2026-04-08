# ChattyBuddy
This project proposes an intelligent, child-friendly toy with a built‑in Arabic large language model (LLM) that can interact naturally with kids through conversation, stories, and games. The toy acts as a personal companion, answering questions, encouraging curiosity in a safe, age‑appropriate way. 


# Initial workflow:


## 1. Transcribe speech (Speech to Text) :
- An Arabic speech-to-text model is used to recognize Arabic speech and convert it into text
  
## 2. Enter the text into an Arabic LLM
-jais
- The transcribed text is sent to a large language model such as ChatGPT, Gemini, Llama, or any other available model.

## 3. generate response and ensure its safe for children
Do we need to fine tune the model to make it child friendly, or can this be done through prompt engineering??

## 4. text to speach (stream response)
- The LLM’s text response is then passed to an Arabic text-to-speech model, which converts the generated Arabic text into audio.



### helpful sources?:
- -> [be-more-agent](https://github.com/brenpoly/be-more-agent)
- -> [arabic LLMs benchmarks](https://github.com/tiiuae/Arabic-LLM-Benchmarks)



# ChattyBuddy 🤖🧒

An intelligent, child-friendly Arabic-speaking toy built on a Raspberry Pi. ChattyBuddy holds natural conversations with kids — telling stories, answering questions, and encouraging curiosity — in a safe, age-appropriate way.

---

## Table of Contents

- [Overview](#overview)
- [System Pipeline](#system-pipeline)
- [Technology Stack](#technology-stack)
- [Safety Approach](#safety-approach)
- [Project Roadmap](#project-roadmap)
- [Cost Estimates](#cost-estimates)
- [Hardware Requirements](#hardware-requirements)
- [Getting Started](#getting-started)
- [Folder Structure](#folder-structure)
- [Contributing](#contributing)

---

## Overview

ChattyBuddy is a Raspberry Pi-powered interactive toy that:

- Listens to a child speaking Arabic
- Transcribes speech using a local or cloud STT model
- Generates a safe, child-appropriate response via an Arabic-capable LLM
- Speaks the response back using a natural-sounding Arabic TTS voice
- (Optionally) displays visuals or expressions on a small screen

The entire pipeline runs end-to-end in under 4 seconds, targeting a conversational, responsive experience for children aged 4–10.

---

## System Pipeline

```
Child speaks Arabic
        │
        ▼
┌───────────────────┐
│  Speech-to-Text   │  faster-whisper (local) or OpenAI Whisper API
└────────┬──────────┘
         │ Arabic text
         ▼
┌───────────────────┐
│   Safety Filter   │  Input keyword blocklist + OpenAI Moderation API
└────────┬──────────┘
         │ Clean text
         ▼
┌───────────────────┐
│    Arabic LLM     │  GPT-4o / Gemini 2.0 Flash / Jais
│  (system prompt)  │  Child-safe persona via prompt engineering
└────────┬──────────┘
         │ Arabic response text
         ▼
┌───────────────────┐
│   Safety Filter   │  Output moderation pass
└────────┬──────────┘
         │ Clean response
         ▼
┌───────────────────┐
│  Text-to-Speech   │  ElevenLabs / OpenAI TTS / Coqui XTTS (local)
└────────┬──────────┘
         │ Audio stream
         ▼
    Speaker output
```

All components run as a single async Python event loop on the Raspberry Pi.

---

## Technology Stack

### Recommended stack (development)

| Component | Service | Cost |
|-----------|---------|------|
| STT | `faster-whisper` large-v3 (local on Pi) | Free |
| LLM | Gemini 2.0 Flash API | ~$0.10 / 1M tokens |
| TTS | Coqui XTTS v2 (local on Pi) | Free |
| Safety | OpenAI Moderation API | Free |

### Recommended stack (production quality)

| Component | Service | Cost |
|-----------|---------|------|
| STT | OpenAI Whisper API | $0.006 / min |
| LLM | GPT-4o mini or GPT-4o | $0.15–$2.50 / 1M tokens |
| TTS | ElevenLabs (streaming) | $5–$11 / month |
| Safety | OpenAI Moderation API | Free |

### Arabic-native LLM alternative

[Jais](https://huggingface.co/inceptionai/jais-13b-chat) by G42/MBZUAI is an Arabic-native open-source LLM worth evaluating for more natural dialectal Arabic output.

---

## Safety Approach

> **We use prompt engineering, not fine-tuning.**

 Modern LLMs can be made reliably child-safe through a well-crafted system prompt alone. Our layered safety approach:

### Layer 1 — System prompt persona

The system prompt defines ChattyBuddy's character and hard constraints:

```
You are ChattyBuddy (رفيق الأطفال), a friendly, curious Arabic-speaking companion for children aged 4–10.

Rules you must always follow:
- Respond only in simple, clear Arabic (Gulf dialect preferred, with simple MSA)
- Keep all responses under 3 sentences — short and engaging
- Never discuss violence, adult content, politics, or religion
- If asked something inappropriate, redirect warmly: "هذا سؤال للكبار، لكن تعالَ نحكي قصة!"
- Always be encouraging, positive, and curious
- Never reveal that you are an AI unless directly asked
```

### Layer 2 — Input moderation

Run every transcribed child utterance through OpenAI's Moderation API before it reaches the LLM.

### Layer 3 — Output moderation

Run every LLM response through the Moderation API and a custom Arabic keyword blocklist before passing to TTS.

### Layer 4 — Arabic keyword blocklist

A simple Python list of Arabic words/phrases that should never appear in output, as a final safety gate.

---

## Project Roadmap

### Week 1 — Hardware & environment setup

- [ ] Flash Raspberry Pi OS Bookworm on Pi 5
- [ ] Connect and test USB microphone (or ReSpeaker HAT)
- [ ] Test audio I/O with `arecord` / `aplay`
- [ ] Set up Python 3.11+ virtual environment
- [ ] Install `pyaudio`, `sounddevice`, `faster-whisper`
- [ ] Verify audio record → playback loop works

### Week 2 — Speech-to-text (STT)

- [ ] Integrate `faster-whisper` large-v3 locally on Pi
- [ ] Tune voice activity detection (VAD) threshold for children's voices
- [ ] Benchmark transcription latency (target: < 2s for 6s of speech)
- [ ] Test with real child voice samples in Arabic
- [ ] Add fallback to OpenAI Whisper API if local latency is too high

### Week 3 — LLM + safety layer

- [ ] Set up OpenAI / Gemini API client
- [ ] Design and test system prompt (multiple iterations)
- [ ] Integrate OpenAI Moderation API for input and output
- [ ] Build Arabic keyword blocklist
- [ ] Red-team test: attempt to bypass safety via roleplay, hypotheticals, repetition
- [ ] Evaluate Jais as an Arabic-native alternative

### Week 4 — Text-to-speech (TTS)

- [ ] Integrate Coqui XTTS v2 locally for development
- [ ] Evaluate ElevenLabs streaming API for production quality
- [ ] Implement response streaming (speak first sentence before full response is ready)
- [ ] Test perceived latency — target under 4s end-to-end

### Week 5 — Integration, testing & polish

- [ ] Wire full async pipeline: listen → STT → filter → LLM → filter → TTS → play
- [ ] Measure and optimise end-to-end latency
- [ ] Add graceful error handling (network outages, timeouts, fallback phrases)
- [ ] Full safety red-team session
- [ ] (Optional) Add 3.5" Waveshare LCD screen with `pygame` for expressions/story art

---

## Cost Estimates

Monthly API cost at **20 conversations/day, 8 turns each, ~6s speech per turn:**

| Stack | STT | LLM | TTS | Total / month |
|-------|-----|-----|-----|---------------|
| Full local (dev) | Free | ~$1.40 | Free | **~$1.40** |
| Balanced | Free | ~$1.40 | $5.00 | **~$6.40** |
| High quality | $2.60 | ~$9.20 | $5.00 | **~$16.80** |
| Max quality | $2.60 | ~$55 | $11.00 | **~$68.60** |

**Development budget (5 weeks of testing):** estimate $10–20 total in API calls. Use Gemini Flash during development and switch to GPT-4o only for final quality comparison.

> Note: ElevenLabs Starter ($5/mo) includes 30,000 characters. At 8 responses/day × 200 chars, you'll use ~48,000 chars/month — consider the Creator plan ($11/mo, 100K chars) for production.

---

## Hardware Requirements

| Component | Recommendation |
|-----------|---------------|
| SBC | Raspberry Pi 5 (4GB or 8GB) |
| Microphone | USB microphone or ReSpeaker 2-Mic HAT |
| Speaker | Any USB or 3.5mm speaker |
| Storage | 32GB+ microSD (Class 10 / A2) |
| Power | Official Pi 5 USB-C PSU (5V/5A) |
| Screen (optional) | Waveshare 3.5" SPI TFT LCD |

---

## Getting Started

### 1. Clone the repo

```bash
git clone https://github.com/your-username/chattybuddy.git
cd chattybuddy
```

### 2. Create a virtual environment

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 3. Set up environment variables

```bash
cp .env.example .env
# Edit .env and add your API keys
```

```env
OPENAI_API_KEY=your_key_here
GEMINI_API_KEY=your_key_here
ELEVENLABS_API_KEY=your_key_here
```

### 4. Test audio I/O

```bash
python scripts/test_audio.py
```

### 5. Run the pipeline

```bash
python main.py
```

---

## Folder Structure

```
chattybuddy/
├── main.py                  # Main async event loop
├── requirements.txt
├── .env.example
├── pipeline/
│   ├── stt.py               # Speech-to-text (Whisper)
│   ├── llm.py               # LLM client + system prompt
│   ├── tts.py               # Text-to-speech (ElevenLabs / Coqui)
│   └── safety.py            # Moderation + keyword filter
├── prompts/
│   └── system_prompt.txt    # ChattyBuddy persona prompt
├── scripts/
│   ├── test_audio.py        # Audio I/O sanity check
│   └── red_team.py          # Safety stress testing
├── screen/                  # Optional LCD display module
│   └── display.py
└── assets/
    └── blocklist_ar.txt     # Arabic keyword blocklist
```


*Built with Raspberry Pi, Python, and a love for curious kids.*
