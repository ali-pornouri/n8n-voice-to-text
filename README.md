# 🎙️ n8n Voice to Text Bot

<div align="center">

[![n8n](https://img.shields.io/badge/n8n-workflow-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://n8n.io)
[![Groq](https://img.shields.io/badge/Groq-Whisper_v3-FF6B35?style=for-the-badge&logoColor=white)](https://groq.com)
[![Telegram](https://img.shields.io/badge/Telegram-Bot-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)](https://telegram.org)
[![Docker](https://img.shields.io/badge/Docker-Self_Hosted-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docker.com)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

**Automatically transcribe voice messages from Telegram to text using Groq Whisper AI.**  
Supports any language — Persian, English, Italian, Arabic and more.

[Features](#-features) • [Architecture](#-architecture) • [Prerequisites](#-prerequisites) • [Installation](#-installation) • [Usage](#-usage)

</div>

---

## 📌 Overview

This project is a **self-hosted automation workflow** built with [n8n](https://n8n.io) that listens for voice messages on Telegram, converts them to text using [Groq's Whisper Large V3](https://groq.com) model, and sends the transcription back to the user — all automatically, in seconds, and completely free.

> **Why self-hosted?**  
> Running n8n locally means your voice data never touches third-party servers beyond Groq's transcription API. Full control over your infrastructure.

---

## ✨ Features

- 🎙️ **Voice to Text** — transcribes Telegram voice messages automatically
- 🌍 **Multilingual** — auto-detects and transcribes any language (Persian, English, Arabic, Italian, etc.)
- ⚡ **Fast** — Groq's LPU hardware transcribes a 1-minute voice in under 2 seconds
- 💰 **Free** — uses Groq's free tier (Whisper Large V3)
- 📱 **Telegram integration** — works directly inside any Telegram chat
- 📄 **Long message support** — splits transcriptions longer than 4000 characters into multiple messages
- 🔒 **Privacy-first** — self-hosted on your own server, no cloud lock-in
- 🐳 **Docker-ready** — runs via Docker Compose with ngrok for external access

---

## 🏗️ Architecture

```
User sends voice  ──►  Telegram Bot  ──►  n8n Workflow
                                              │
                    ┌─────────────────────────┘
                    │
                    ▼
            ┌───────────────┐
            │  Input Trigger │  (Telegram Webhook)
            └───────┬───────┘
                    │
                    ▼
            ┌───────────────┐
            │   Get Voice   │  (fetch file_id from Telegram)
            └───────┬───────┘
                    │
                    ▼
            ┌───────────────┐
            │  Fetch File   │  (download binary audio)
            └───────┬───────┘
                    │
                    ▼
            ┌──────────────────┐
            │ Change Format    │  (rename to voice.mp3 for Groq)
            └───────┬──────────┘
                    │
                    ▼
            ┌───────────────────┐
            │  Groq AI Engine   │  (Whisper Large V3 transcription)
            └───────┬───────────┘
                    │
                    ▼
            ┌──────────────────────┐
            │  Code (JavaScript)   │  (split text into ≤4000 char chunks)
            └───────┬──────────────┘
                    │
                    ▼
            ┌───────────────┐
            │    Output     │  (send text back to Telegram)
            └───────────────┘
```

### Node Breakdown

| Node | Type | Purpose |
|------|------|---------|
| **Input Trigger** | Telegram Trigger | Listens for incoming Telegram messages |
| **Get Voice** | Telegram | Retrieves the file_id of the voice message |
| **Fetch File** | HTTP Request | Downloads the audio binary from Telegram servers |
| **Change Format File** | Code (JS) | Renames file to `.mp3` and sets correct MIME type for Groq |
| **Groq AI Engine** | HTTP Request | Sends audio to Groq Whisper API for transcription |
| **Code in JavaScript** | Code (JS) | Splits long text into Telegram-compatible chunks (≤4000 chars) |
| **Output** | Telegram | Sends the transcribed text back to the user |

---

## 🛠️ Prerequisites

Before you begin, make sure you have:

- A Linux server (Ubuntu 22.04 recommended) — VPS, local machine, or VM
- [Docker](https://docs.docker.com/engine/install/ubuntu/) and [Docker Compose](https://docs.docker.com/compose/install/) installed
- A **Telegram Bot Token** — get one from [@BotFather](https://t.me/BotFather)
- A **Groq API Key** — free at [console.groq.com](https://console.groq.com)
- An **ngrok account** — free at [ngrok.com](https://ngrok.com) (for webhook access)

---

## 🚀 Installation

### 1. Clone the repository

```bash
git clone https://github.com/ali-pornouri/n8n-voice-to-text.git
cd n8n-voice-to-text
```

### 2. Configure environment variables

Copy the example environment file and fill in your credentials:

```bash
cp .env.example .env
nano .env
```

```env
# Telegram
TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here

# ngrok
NGROK_AUTHTOKEN=your_ngrok_authtoken_here
NGROK_DOMAIN=your-subdomain.ngrok-free.app

# Groq
GROQ_API_KEY=your_groq_api_key_here
```

### 3. Start the stack

```bash
docker compose up -d
```

This starts three containers:
- **n8n** — the workflow automation engine
- **ngrok** — creates a public HTTPS tunnel to your n8n instance

### 4. Access n8n

Open your browser and go to:
```
https://your-subdomain.ngrok-free.app
```

Or locally:
```
http://localhost:5678
```

### 5. Import the workflow

1. In n8n, go to **Workflows → Import from file**
2. Select `voice-to-text-workflow.json`
3. Configure your credentials (see below)

### 6. Configure credentials

#### Telegram credential
1. In n8n, go to **Credentials → New → Telegram**
2. Paste your Bot Token from BotFather
3. Name it (e.g. `MyTelegramBot`)

#### Groq credential
1. In n8n, go to **Credentials → New → Header Auth**
2. Set **Name** to `Authorization`
3. Set **Value** to `Bearer YOUR_GROQ_API_KEY`
4. Name it (e.g. `Groq API`)

### 7. Activate the workflow

1. Open the imported workflow
2. Update the **Credential** fields in the Telegram nodes and Groq node
3. Toggle the workflow to **Active**
4. Send a voice message to your bot — it should respond with transcribed text!

---

## 📖 Usage

Once the workflow is active:

1. Open Telegram and find your bot
2. Record and send a **voice message** (any language)
3. The bot will reply with the transcribed text within seconds

**Example:**

```
You:  🎙️ [voice message in Persian]
Bot:  سلام، این یک پیام آزمایشی است که به صورت صوتی ارسال شد.
```

---

## 📁 Project Structure

```
n8n-voice-to-text/
├── voice-to-text-workflow.json   # n8n workflow (import this)
├── docker-compose.yml            # Docker stack definition
├── .env.example                  # Environment variables template
├── .gitignore                    # Git ignore rules
├── LICENSE                       # MIT License
└── README.md                     # This file
```

---

## ⚙️ Configuration

### Groq Model

The workflow uses `whisper-large-v3` by default. You can change it in the **Groq AI Engine** node:

| Model | Speed | Accuracy | Notes |
|-------|-------|----------|-------|
| `whisper-large-v3` | Fast | Highest | Recommended — best multilingual support |
| `whisper-large-v3-turbo` | Fastest | High | Good for real-time use cases |
| `distil-whisper-large-v3-en` | Very Fast | High | English only |

### Telegram File Size Limit

Telegram's Bot API limits file downloads to **20MB**. For longer voice messages, consider:
- Using the [Telegram Local Bot API](https://core.telegram.org/bots/api#using-a-local-bot-api-server) (supports up to 2GB)
- Splitting recordings into shorter clips before sending

---

## 🔒 Security Notes

- **Never commit your `.env` file** — it is excluded via `.gitignore`
- The workflow JSON in this repo has all credentials and tokens removed
- Replace all `YOUR_*` placeholders before use
- Use environment variables for all sensitive values

---

## 🤝 Contributing

Contributions are welcome! If you find this useful:

- ⭐ Give it a star
- 🍴 Fork it and improve it
- 🐛 Report bugs via [Issues](https://github.com/ali-pornouri/n8n-voice-to-text/issues)
- 💡 Suggest features via [Issues](https://github.com/ali-pornouri/n8n-voice-to-text/issues)

---

## 📜 License

This project is licensed under the [MIT License](LICENSE) — free to use, modify, and distribute.

---

<div align="center">

**Built with ❤️ by [Ali Pornouri](https://github.com/ali-pornouri)**

🇮🇷 Iranian developer based in 🇮🇹 Italy

⭐ If this project helped you, please give it a star!

</div>
