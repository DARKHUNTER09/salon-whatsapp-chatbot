# 💈 Salon WhatsApp Chatbot
> A WhatsApp chatbot for a Unisex Salon built with **Twilio** + **n8n** — no custom backend needed!

![n8n](https://img.shields.io/badge/n8n-workflow-orange?style=flat-square)
![Twilio](https://img.shields.io/badge/Twilio-WhatsApp-red?style=flat-square)
![ngrok](https://img.shields.io/badge/ngrok-tunnel-blue?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)

---

## 📱 Demo

Customer WhatsApp pe message karta hai → Chatbot automatically reply karta hai!

| Customer Bhejta Hai | Chatbot Reply |
|--------------------|---------------|
| `hi` | Welcome message + options menu |
| `menu` | Services & prices list |
| `book` | Appointment booking form |
| `staff` | Stylist profiles |
| `offer` | Current deals & discounts |
| `status` | Booking status check |

---

## 🛠️ Tech Stack

| Tool | Use |
|------|-----|
| **Twilio WhatsApp Sandbox** | WhatsApp messages receive & send |
| **n8n** | Workflow automation (no-code/low-code) |
| **ngrok** | Local server ko internet pe expose karna |
| **Google Sheets** | Data store (services, staff, offers, appointments) |
| **JavaScript** | n8n Code nodes mein logic |

---

## ✨ Features

- 🤖 **Auto Intent Detection** — `hi`, `book`, `menu`, `staff`, `offer`, `status` keywords
- 💇 **Services Menu** — Category-wise services with prices & duration
- 📅 **Appointment Booking** — Structured booking form via WhatsApp
- 👩‍🎨 **Staff Info** — Stylist profiles with speciality & experience
- 🎉 **Offers & Discounts** — Latest deals
- 📋 **Booking Status** — Customer apni booking check kar sakta hai
- 🔄 **Fallback Handler** — Unknown messages ke liye smart fallback
- 🌐 **Hinglish Support** — Hindi + English mixed replies

---

## 🏗️ Architecture

```
Customer (WhatsApp)
        ↓
Twilio WhatsApp Sandbox
        ↓
ngrok tunnel (localhost expose)
        ↓
n8n Webhook Trigger
        ↓
Parse Message & Intent Detection
        ↓
Switch / Intent Router
    ├── Greeting  → Welcome message
    ├── Book      → Booking form
    ├── Menu      → Services list
    ├── Staff     → Stylist info
    ├── Offers    → Deals & discounts
    ├── Status    → Booking check
    └── Unknown   → Fallback message
        ↓
Twilio API → WhatsApp Reply
```

---

## 🚀 Local Setup (Windows)

### Prerequisites

| Tool | Download | Check |
|------|----------|-------|
| Node.js v18+ | [nodejs.org](https://nodejs.org) | `node -v` |
| ngrok | [ngrok.com](https://ngrok.com/download) | `ngrok version` |
| Twilio Account | [twilio.com](https://www.twilio.com/try-twilio) | — |

### Step 1 — n8n Install & Start

```bash
npm install n8n -g
n8n start
```

n8n opens at `http://localhost:5678`

### Step 2 — ngrok Tunnel

```bash
# New terminal window mein
ngrok http 5678
```

Copy the HTTPS URL: `https://xxxx.ngrok-free.app`

### Step 3 — Twilio Sandbox Join

1. Twilio Console → **Messaging → Try it out → Send a WhatsApp message**
2. Send from your WhatsApp to `+1 415 523 8886`:
   ```
   join possibly-sort
   ```

### Step 4 — Webhook URL Set Karo

Twilio Console → **Messaging → Settings → WhatsApp Sandbox Settings**

```
When a message comes in:
https://xxxx.ngrok-free.app/webhook/twilio-salon
Method: HTTP POST
```

### Step 5 — n8n Workflow Import

1. n8n → **Workflows → Import from File**
2. Upload `My_workflow_clean.json`
3. Add credentials:
   - **Basic Auth** → Username: `YOUR_TWILIO_ACCOUNT_SID`, Password: `YOUR_TWILIO_AUTH_TOKEN`
4. In **Send WhatsApp Reply** node → update:
   - URL: `https://api.twilio.com/2010-04-01/Accounts/YOUR_SID/Messages.json`
   - From: `whatsapp:YOUR_TWILIO_SANDBOX_NUMBER`
5. **Activate** the workflow

### Step 6 — Test Karo!

Send `hi` from your WhatsApp to the Twilio sandbox number → reply aana chahiye! 🎉

---

## 📁 Project Structure

```
salon-whatsapp-chatbot/
├── My_workflow_clean.json        # n8n workflow (import ready)
├── setup_guide.md                # Local Windows setup guide
├── next_steps_guide.md           # Production & features roadmap
└── README.md                     # You are here!
```

---

## 🔐 Environment Variables

Sensitive data ko directly code mein mat daalo. n8n Credentials use karo:

| Variable | Where |
|----------|-------|
| `TWILIO_ACCOUNT_SID` | n8n → Credentials → Basic Auth → Username |
| `TWILIO_AUTH_TOKEN` | n8n → Credentials → Basic Auth → Password |
| `TWILIO_WHATSAPP_NUMBER` | Send node → From field |

---

## 🗺️ Roadmap

- [x] Basic chatbot with 6 intents
- [x] Twilio WhatsApp Sandbox integration
- [x] n8n workflow automation
- [ ] Google Sheets live data (menu, staff, offers)
- [ ] Appointment booking save to Google Sheets
- [ ] Static ngrok domain (permanent URL)
- [ ] AWS EC2 production deployment
- [ ] Real Twilio WhatsApp Business number
- [ ] AI smart replies (Gemini/OpenAI)

---

## ⚠️ Twilio Sandbox Limitations

- Only numbers that have sent `join possibly-sort` can receive messages
- Max 10 test numbers
- For production → apply for Twilio WhatsApp Business API

---

## 👨‍💻 Author

**Shubham Singh**
Full-Stack Developer | Mumbai

[![GitHub](https://img.shields.io/badge/GitHub-Profile-black?style=flat-square&logo=github)](https://github.com/YOUR_USERNAME)

---

## 📄 License

MIT License — feel free to use and modify!

---

*Built with ❤️ using Twilio + n8n — No custom backend needed!*
