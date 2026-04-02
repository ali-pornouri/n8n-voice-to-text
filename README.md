# 🎙️ n8n Voice to Text Bot

<div align="center">

[![n8n](https://img.shields.io/badge/n8n-workflow-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://n8n.io)
[![Groq](https://img.shields.io/badge/Groq-Whisper_v3-FF6B35?style=for-the-badge&logoColor=white)](https://groq.com)
[![Telegram](https://img.shields.io/badge/Telegram-Bot-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)](https://telegram.org)
[![Docker](https://img.shields.io/badge/Docker-Self_Hosted-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docker.com)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

**Automatically transcribe voice messages from Telegram to text using Groq Whisper AI.**
Supports any language — Persian, English, Italian, Arabic and more.

[🇬🇧 English](#-english) • [🇮🇷 فارسی](#-فارسی)

</div>

---

## 🇬🇧 English

### 📌 Overview

This project is a **self-hosted automation workflow** built with [n8n](https://n8n.io) that listens for voice messages on Telegram, converts them to text using [Groq's Whisper Large V3](https://groq.com) model, and sends the transcription back to the user — all automatically, in seconds, and completely free.

> **Why self-hosted?**
> Running n8n locally means your voice data never touches third-party servers beyond Groq's transcription API. Full control over your infrastructure.

---

### ✨ Features

- 🎙️ **Voice to Text** — transcribes Telegram voice messages automatically
- 🌍 **Multilingual** — auto-detects and transcribes any language (Persian, English, Arabic, Italian, etc.)
- ⚡ **Fast** — Groq's LPU hardware transcribes a 1-minute voice in under 2 seconds
- 💰 **Free** — uses Groq's free tier (Whisper Large V3)
- 📱 **Telegram integration** — works directly inside any Telegram chat
- 📄 **Long message support** — splits transcriptions longer than 4000 characters into multiple messages
- 🔒 **Privacy-first** — self-hosted on your own server, no cloud lock-in
- 🐳 **Docker-ready** — runs via Docker Compose with ngrok for external access

---

### 🏗️ Architecture

```
User sends voice  ──►  Telegram Bot  ──►  n8n Workflow
                                              │
                    ┌─────────────────────────┘
                    │
                    ▼
            ┌───────────────┐
            │ Input Trigger │  (Telegram Webhook)
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

### 🛠️ Prerequisites

- A Linux server (Ubuntu 22.04 recommended) — VPS, local machine, or VM
- [Docker](https://docs.docker.com/engine/install/ubuntu/) and [Docker Compose](https://docs.docker.com/compose/install/) installed
- A **Telegram Bot Token** — get one from [@BotFather](https://t.me/BotFather)
- A **Groq API Key** — free at [console.groq.com](https://console.groq.com)
- An **ngrok account** — free at [ngrok.com](https://ngrok.com)

---

### 🚀 Installation

#### 1. Clone the repository

```bash
git clone https://github.com/ali-pornouri/n8n-voice-to-text.git
cd n8n-voice-to-text
```

#### 2. Configure environment variables

```bash
cp .env.example .env
nano .env
```

```env
TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here
NGROK_AUTHTOKEN=your_ngrok_authtoken_here
NGROK_DOMAIN=your-subdomain.ngrok-free.app
GROQ_API_KEY=your_groq_api_key_here
```

#### 3. Start the stack

```bash
docker compose up -d
```

#### 4. Import the workflow

1. Open n8n at `https://your-subdomain.ngrok-free.app`
2. Go to **Workflows → Import from file**
3. Select `voice-to-text-workflow.json`
4. Set your **Telegram** and **Groq** credentials
5. Toggle the workflow to **Active**

---

### 📖 Usage

1. Open Telegram and find your bot
2. Record and send a **voice message** (any language)
3. The bot replies with the transcribed text within seconds

```
You:  🎙️ [voice message in Persian]
Bot:  سلام، این یک پیام آزمایشی است که به صورت صوتی ارسال شد.
```

---

### ⚙️ Configuration

| Model | Speed | Accuracy | Notes |
|-------|-------|----------|-------|
| `whisper-large-v3` | Fast | Highest | Recommended — best multilingual support |
| `whisper-large-v3-turbo` | Fastest | High | Good for real-time use cases |
| `distil-whisper-large-v3-en` | Very Fast | High | English only |

---

### 🔒 Security Notes

- **Never commit your `.env` file** — it is excluded via `.gitignore`
- The workflow JSON has all credentials and tokens removed
- Replace all `YOUR_*` placeholders before use

---

### 🤝 Contributing

- ⭐ Give it a star
- 🍴 Fork it and improve it
- 🐛 Report bugs via [Issues](https://github.com/ali-pornouri/n8n-voice-to-text/issues)

---

## 🇮🇷 فارسی

### 📌 توضیحات

این پروژه یک **workflow اتوماسیون self-hosted** با [n8n](https://n8n.io) است که پیام‌های صوتی تلگرام را دریافت کرده، با مدل [Whisper Large V3 گروک](https://groq.com) به متن تبدیل می‌کند و نتیجه را به کاربر برمی‌گرداند — به صورت خودکار، در چند ثانیه و کاملاً رایگان.

> **چرا self-hosted؟**
> با اجرای n8n روی سرور شخصی، داده‌های صوتی شما فقط برای تبدیل به متن به Groq می‌رود و هیچ سرویس شخص ثالث دیگری به آن دسترسی ندارد. کنترل کامل زیرساخت در دست شماست.

---

### ✨ قابلیت‌ها

- 🎙️ **تبدیل صوت به متن** — پیام‌های صوتی تلگرام را خودکار تبدیل می‌کند
- 🌍 **چندزبانه** — هر زبانی را تشخیص داده و تبدیل می‌کند (فارسی، انگلیسی، عربی، ایتالیایی و...)
- ⚡ **سریع** — یک دقیقه صدا در کمتر از ۲ ثانیه توسط Groq پردازش می‌شود
- 💰 **رایگان** — از سطح رایگان Groq (Whisper Large V3) استفاده می‌کند
- 📱 **یکپارچه با تلگرام** — مستقیماً در هر چت تلگرام کار می‌کند
- 📄 **پشتیبانی از متن طولانی** — متن‌های بیش از ۴۰۰۰ کاراکتر را به چند پیام تقسیم می‌کند
- 🔒 **حریم خصوصی محور** — روی سرور شخصی شما اجرا می‌شود
- 🐳 **آماده Docker** — با Docker Compose و ngrok اجرا می‌شود

---

### 🏗️ معماری سیستم

| نود | نوع | وظیفه |
|-----|-----|--------|
| **Input Trigger** | Telegram Trigger | گوش دادن به پیام‌های ورودی تلگرام |
| **Get Voice** | Telegram | دریافت file_id پیام صوتی |
| **Fetch File** | HTTP Request | دانلود فایل صوتی از سرورهای تلگرام |
| **Change Format File** | Code (JS) | تغییر نام فایل به `.mp3` و تنظیم MIME type برای Groq |
| **Groq AI Engine** | HTTP Request | ارسال صدا به API Whisper گروک برای تبدیل به متن |
| **Code in JavaScript** | Code (JS) | تقسیم متن طولانی به بخش‌های ≤۴۰۰۰ کاراکتری |
| **Output** | Telegram | ارسال متن تبدیل شده به کاربر |

---

### 🛠️ پیش‌نیازها

- یک سرور لینوکس (Ubuntu 22.04 توصیه می‌شود) — VPS، ماشین محلی یا VM
- [Docker](https://docs.docker.com/engine/install/ubuntu/) و [Docker Compose](https://docs.docker.com/compose/install/) نصب شده
- **توکن ربات تلگرام** — از [@BotFather](https://t.me/BotFather) دریافت کنید
- **کلید API گروک** — رایگان از [console.groq.com](https://console.groq.com)
- **حساب ngrok** — رایگان از [ngrok.com](https://ngrok.com)

---

### 🚀 نصب و راه‌اندازی

#### ۱. کلون کردن پروژه

```bash
git clone https://github.com/ali-pornouri/n8n-voice-to-text.git
cd n8n-voice-to-text
```

#### ۲. تنظیم متغیرهای محیطی

```bash
cp .env.example .env
nano .env
```

```env
TELEGRAM_BOT_TOKEN=توکن_ربات_تلگرام_شما
NGROK_AUTHTOKEN=توکن_ngrok_شما
NGROK_DOMAIN=your-subdomain.ngrok-free.app
GROQ_API_KEY=کلید_API_گروک_شما
```

#### ۳. اجرای سرویس‌ها

```bash
docker compose up -d
```

#### ۴. ایمپورت workflow

1. n8n را در `https://your-subdomain.ngrok-free.app` باز کنید
2. به **Workflows → Import from file** بروید
3. فایل `voice-to-text-workflow.json` را انتخاب کنید
4. Credential های **تلگرام** و **Groq** را تنظیم کنید
5. workflow را **Active** کنید

---

### 📖 نحوه استفاده

1. ربات تلگرام خود را پیدا کنید
2. یک **پیام صوتی** (به هر زبانی) ارسال کنید
3. ربات در چند ثانیه متن را برمی‌گرداند

```
شما:  🎙️ [پیام صوتی فارسی]
ربات: سلام، این یک پیام آزمایشی است که به صورت صوتی ارسال شد.
```

---

### 📁 ساختار پروژه

```
n8n-voice-to-text/
├── voice-to-text-workflow.json   # workflow اصلی n8n
├── docker-compose.yml            # تعریف Docker stack
├── .env.example                  # قالب متغیرهای محیطی
├── .gitignore                    # قوانین Git ignore
├── LICENSE                       # مجوز MIT
└── README.md                     # این فایل
```

---

### 🔒 نکات امنیتی

- **هرگز فایل `.env` را commit نکنید** — توسط `.gitignore` حذف شده است
- فایل JSON این پروژه از تمام credentials و توکن‌ها پاک شده است
- تمام placeholder های `YOUR_*` را قبل از استفاده جایگزین کنید

---

### 🤝 مشارکت

- ⭐ ستاره بده
- 🍴 Fork کن و بهترش کن
- 🐛 باگ‌ها را در [Issues](https://github.com/ali-pornouri/n8n-voice-to-text/issues) گزارش بده

---

## 📜 License / مجوز

This project is licensed under the [MIT License](LICENSE).

این پروژه تحت [مجوز MIT](LICENSE) منتشر شده است.

---

<div align="center">

**Built with ❤️ by [Ali Pornouri](https://github.com/ali-pornouri)**

🇮🇷 Iranian developer based in 🇮🇹 Italy

⭐ If this project helped you, please give it a star!

⭐ اگه این پروژه مفید بود، ستاره بده!

</div>
