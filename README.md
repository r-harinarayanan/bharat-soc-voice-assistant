# 🎙️ Bharat SOC — Offline Privacy-Preserving Hindi Voice Assistant

> **"Your voice. Your device. No cloud. No compromise."**

A fully offline Hindi voice assistant built for Raspberry Pi using Edge AI. Every layer — speech recognition, intent classification, command execution — runs entirely on-device. No internet. No data leaves the hardware. Just fast, private, intelligent voice control in Hindi.

Built as part of the **Bharat AI-SoC Student Challenge 2026** by students of Amrita Vishwa Vidyapeetham, Chennai.

---

## 👥 Team

| Name | Role |
|------|------|
| R Hari Narayanan | System Architecture, ML Pipeline |
| Sreenand TM | Hardware Integration, Testing |
| Ramavarshini N | Dataset Design, Model Training |

**Institution:** Amrita Vishwa Vidyapeetham, Chennai Campus

---

## 🧠 What Is This?

Most voice assistants you've used — Alexa, Siri, Google Assistant — send your voice to the cloud to understand what you said. That means your conversations are processed on remote servers, creating privacy risks and internet dependency.

**Bharat SOC** flips that model entirely.

Everything runs on a Raspberry Pi sitting on your desk:
- Your voice is recognized using **Vosk** (an offline speech engine)
- Your intent is classified using **FastText** (a lightweight ML model trained on Hindi commands)
- Actions are executed immediately on the device
- Results are spoken back in Hindi using **espeak-ng** and shown on an **LCD display**

Zero cloud. Zero latency from network calls. Zero data exposure.

---

## ✨ Features

### 🏠 Device Control
- Turn lights **on/off** — supports 20+ command variations (लाइट ऑन, बत्ती जलाओ, बल्ब चालू...)
- Control fans — (पंखा चालू, फैन ऑन, हवा चलाओ...)
- Handles both Hindi and Hinglish phrasing

### 🔢 Math Engine
- Supports addition, subtraction, and division
- Full Hindi number vocabulary from 0 to 1 crore (शून्य to करोड़)
- Fuzzy number matching for pronunciation variations

### 📅 Date & Time
- Current time query (अभी कितने बजे हैं?)
- Today's, tomorrow's, and day-after-tomorrow's date
- Speaks response in Hinglish + shows on LCD

### 🧍 Personalized Memory
- Learns and remembers your name across sessions (`personal_memory.json`)
- Spells your name back character by character
- Persistent across reboots

### 🔐 Privacy by Design
- No microphone data ever leaves the device
- No API keys, no accounts, no telemetry
- Full offline operation — works without Wi-Fi

### 💬 Natural Language Robustness
- Wake word detection: **"Bharat"** or **"भारत"** (+ 6 phonetic variants: barat, bart, varat, parrot, birth, baarat)
- Fuzzy string matching via `SequenceMatcher` to handle mispronunciation
- 15-second active window after wake word — returns to idle automatically
- Dual-layer intent resolution: hardcoded safety net → AI brain fallback

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Bharat AI SOC System                │
├──────────────┬──────────────┬──────────────┬─────────┤
│  Hardware    │    Audio     │   AI Brain   │ Output  │
│  Layer       │  Processing  │              │ Layer   │
├──────────────┼──────────────┼──────────────┼─────────┤
│ Raspberry Pi │ sounddevice  │ FastText     │ espeak  │
│ Microphone   │ Vosk ASR     │ Intent Cls.  │ LCD I2C │
│ LCD 16x2     │ Wake Word    │ Confidence   │         │
│ Speaker      │ Detection    │ Threshold    │         │
│ GPIO Devices │              │ Fuzzy Match  │         │
└──────────────┴──────────────┴──────────────┴─────────┘
```

**Processing Pipeline:**
```
Microphone → Vosk STT → Wake Word Check → Safety Net → FastText Brain → Execute → LCD + espeak
```

---

## 🤖 AI Model Details

### Speech Recognition — Vosk
- Model: `vosk-model-small-hi-0.22` (Hindi)
- Runs fully on-device, no GPU required
- Sample rate: 44100 Hz, block size: 4000 frames
- Partial word recognition enabled for responsiveness

### Intent Classification — FastText
- Custom supervised model trained on a labeled Hindi command dataset
- **Training parameters:**
  - Learning rate: `1.0`
  - Epochs: `25`
  - Word n-grams: `2`
  - Embedding dimension: `50`
  - Loss: `softmax`
- **Confidence threshold:** `0.4` (below this, command is silently ignored)
- **Model quantization:** Applied via `model.quantize()` → compressed to `.ftz` format for faster inference and lower RAM usage

### Supported Intent Labels

| Label | Description |
|-------|-------------|
| `fan_on` / `fan_off` | Fan control |
| `light_on` / `light_off` | Light control |
| `time` | Current time query |
| `date` | Date query (today/tomorrow/day after) |
| `math_query` | Mathematical operations |
| `greet` | Greeting |
| `stop` | Shutdown assistant |
| `ask_name` | Recall stored user name |
| `ask_identity` | Ask who the assistant is |
| `weather` | Weather intent (dataset ready, handler extensible) |

---

## 📦 Dataset

- Custom-built Hindi command dataset (`training.txt`) in FastText format
- **800+ labeled samples** across all intent classes
- Covers formal Hindi, casual Hindi, and Hinglish variations
- Includes phonetic misspellings and transliterations
- Number coverage: 0–100 in Hindi words + English equivalents for math intent
- Format: `__label__<intent> <command phrase>`

Example:
```
__label__fan_on फैन चालू करो
__label__fan_on गर्मी लग रही है फैन ऑन करो
__label__light_off बत्ती बुझाओ
__label__math_query पाँच प्लस तीन कितना होता है
```

---

## 🔧 Hardware Setup

| Component | Specification |
|-----------|--------------|
| Processor | Raspberry Pi 4 Model B |
| Microphone | USB microphone (device index 1) |
| Display | 16×2 LCD via I2C (PCF8574, address 0x27) |
| Speaker | 3.5mm audio output via espeak-ng |
| OS | Raspberry Pi OS (Debian-based) |

### Wiring — LCD I2C to Raspberry Pi

| LCD I2C Pin | Raspberry Pi Pin |
|-------------|-----------------|
| VCC | 5V (Pin 2) |
| GND | GND (Pin 6) |
| SDA | GPIO 2 / SDA (Pin 3) |
| SCL | GPIO 3 / SCL (Pin 5) |

---

## 🚀 Installation & Setup

### Prerequisites
```bash
sudo apt update
sudo apt install python3-pip espeak-ng portaudio19-dev
```

### Python Dependencies
```bash
pip install vosk sounddevice fasttext RPLCD
```

### Project Structure
```
bharat-soc-voice-assistant/
├── runbharat.py          # Main assistant loop
├── trainmodel.py         # FastText training script
├── reduced_size.py       # Model quantization script
├── training.txt          # Labeled Hindi command dataset
├── brain_model.bin       # Trained FastText model (generate via trainmodel.py)
├── brain_model.ftz       # Quantized model (generate via reduced_size.py)
├── model/                # Vosk Hindi model directory
│   └── (download separately — see below)
└── personal_memory.json  # Auto-created on first run
```

### Download Vosk Hindi Model
```bash
wget https://alphacephei.com/vosk/models/vosk-model-small-hi-0.22.zip
unzip vosk-model-small-hi-0.22.zip
mv vosk-model-small-hi-0.22 model
```

### Train the Intent Model
```bash
python3 trainmodel.py
python3 reduced_size.py
```

### Run the Assistant
```bash
python3 runbharat.py
```

---

## 🗣️ How to Use

1. **Power on** → System loads models, LCD shows `SYSTEM ONLINE`, assistant speaks *"System Online"*
2. **Say the wake word** → `"Bharat"` or `"भारत"` → Assistant responds *"Ji?"*
3. **Give your command within 15 seconds:**

| You say | Assistant does |
|---------|---------------|
| लाइट ऑन करो | Turns light on, LCD: `LIGHT → ON` |
| फैन बंद करो | Turns fan off, LCD: `FAN → OFF` |
| टाइम क्या है | Speaks current time, LCD shows time |
| आज की तारीख | Speaks today's date |
| पाँच प्लस तीन | Speaks "5 plus 3 hota hai 8", LCD: `5+3=8` |
| मेरा नाम राहुल है | Saves name, confirms with spelling |
| मेरा नाम क्या है | Recalls saved name |
| तुम कौन हो | *"Main Bharat SOC hoon"* |
| रुक / बस / स्टॉप | Displays `THANK YOU`, speaks *"Alvida"*, exits |

4. **After 15 seconds of inactivity** → Returns to idle, waits for wake word again

---

## 📐 Technical Design Decisions

**Why Vosk over Whisper?**
Whisper requires significantly more RAM and compute. Vosk's small Hindi model runs comfortably on a Pi 4 with <200MB RAM and produces acceptable WER for command-style speech.

**Why FastText over a transformer?**
FastText's inference is sub-millisecond on CPU. For closed-domain intent classification with ~10 classes, a shallow model with n-gram features matches transformer accuracy while using 100x less memory. The `.ftz` quantized model is under 1MB.

**Why a dual-layer intent system?**
The hardcoded safety net (`check_hardcoded_intent`) catches high-frequency commands deterministically, bypassing the ML model entirely. This ensures critical commands (stop, light, fan) are never misclassified due to model confidence issues. FastText handles the long tail.

**Why fuzzy matching?**
Vosk occasionally transcribes spoken Hindi with minor character variations. `SequenceMatcher` with a 0.75–0.85 threshold catches these without requiring exact string matches, making the system robust to ASR noise.

---

## 📊 Model Performance

- Intent classification accuracy: **high** on in-distribution commands
- Wake word detection: fuzzy match across 8 phonetic variants
- Math engine: full coverage of Hindi numerals 0–10,00,00,000
- Response latency: **<500ms** end-to-end (STT + inference + TTS) on Pi 4

---

## 🔮 Future Work

- [ ] Replace `espeak-ng` with **Coqui TTS** for natural-sounding Hindi voice
- [ ] Add multiplication and modulo to math engine
- [ ] Implement `weather` intent handler (label already in dataset)
- [ ] Timer/alarm support via `threading`
- [ ] Actual GPIO control for physical devices (relay module)
- [ ] Upgrade to larger Vosk Hindi model for better ASR accuracy
- [ ] Multi-language support (Tamil, Telugu, Bengali)
- [ ] Local web dashboard (Flask) for live command monitoring
- [ ] Standalone AI-SoC deployment on custom hardware

---

## ⚠️ Known Limitations

- Speech recognition accuracy degrades with background noise
- Math engine limited to addition, subtraction, and division
- Device control currently simulated (no physical GPIO relay wiring)
- Single user memory (no multi-user support)
- `espeak-ng` Hindi TTS sounds robotic

---

## 📄 License

This project is open source. Feel free to fork, extend, and deploy. If you use this work in research or a product, a mention would be appreciated.

---

## 🔗 References

- [Vosk Speech Recognition](https://alphacephei.com/vosk/)
- [FastText by Meta AI](https://fasttext.cc/)
- [RPLCD Library](https://rplcd.readthedocs.io/)
- [Bharat AI-SoC Student Challenge 2026](https://bharataisoc.in)

---


