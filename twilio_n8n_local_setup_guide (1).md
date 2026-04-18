# WhatsApp Chatbot — Twilio + n8n Local Windows Setup
> Stack: Twilio WhatsApp Sandbox + ngrok + n8n (npm) on Windows

---

## Prerequisites (install karni hain pehle)

| Tool | Download Link | Check Command |
|------|--------------|---------------|
| Node.js v18+ | https://nodejs.org | `node -v` |
| ngrok | https://ngrok.com/download | `ngrok version` |
| Twilio Account | https://www.twilio.com/try-twilio | — |

---

## STEP 1 — n8n Install & Start (Windows CMD / PowerShell)

```bash
# Global install
npm install n8n -g

# Start n8n
n8n start
```

n8n ab `http://localhost:5678` pe chalega.

Browser mein open karo → Account create karo (first time) → Dashboard dikhega.

> **Tip:** n8n ko background mein rakhne ke liye ek alag terminal window mein chalaao aur band mat karo.

---

## STEP 2 — ngrok Setup (Twilio ko local n8n tak pahunchane ke liye)

### 2.1 ngrok Install
1. https://ngrok.com/download se Windows zip download karo
2. Kisi folder mein extract karo (e.g. `C:\ngrok\`)
3. Account banao → Dashboard → Authtoken copy karo

### 2.2 ngrok Authtoken Set Karo
```bash
ngrok config add-authtoken YOUR_AUTH_TOKEN_HERE
```

### 2.3 n8n ke liye Tunnel Start Karo
```bash
# Naya terminal window mein (n8n wala band mat karo)
ngrok http 5678
```

Output kuch aisa dikhega:
```
Forwarding  https://abc123def456.ngrok-free.app -> http://localhost:5678
```

**Is HTTPS URL ko copy karo** — yeh Twilio ke liye webhook URL hogi.

> **Important:** Har baar ngrok restart karne pe URL badal jaati hai. Development ke liye ngrok free plan thik hai. Production ke liye paid ngrok plan ya server use karo.

---

## STEP 3 — Twilio WhatsApp Sandbox Setup

### 3.1 Twilio Console
1. https://console.twilio.com login karo
2. Left sidebar → **Messaging → Try it out → Send a WhatsApp message**

### 3.2 Sandbox Join Karna
1. Twilio sandbox number dikhega (e.g. `+1 415 523 8886`)
2. Us number pe WhatsApp message bhejo:
   ```
   join <your-sandbox-keyword>
   ```
   (Keyword Twilio dashboard pe dikhega, e.g. `join bright-tiger`)
3. Twilio se confirmation aayega — sandbox join ho gaya!

### 3.3 Webhook URL Set Karna
1. Twilio Console → **Messaging → Settings → WhatsApp Sandbox Settings**
2. **"When a message comes in"** field mein:
   ```
   https://abc123def456.ngrok-free.app/webhook/twilio-salon
   ```
   *(apni actual ngrok URL daalो)*
3. Method: **HTTP POST**
4. Save karo

### 3.4 Credentials Note Down Karo
Twilio Console → Account → API Keys & Tokens:
- `Account SID` → (ACXXXXXXXXXXXXXXXXX)
- `Auth Token` → (hidden, click to reveal)
- Sandbox WhatsApp number → `+14155238886` (ya jo dikhaye)

---

## STEP 4 — n8n Workflow Import & Configure

### 4.1 Workflow Import
1. n8n Dashboard → **Workflows → Import from File**
2. File upload karo: `twilio_salon_chatbot_workflow.json`
3. **Save** karo

### 4.2 Twilio Credential Add Karo
1. n8n → **Credentials → Add Credential → Twilio API**
2. Fill karo:
   - **Account SID**: `ACXXXXXXXXXXXXXXXXX`
   - **Auth Token**: tumhara auth token
3. Save → Name: `Twilio Salon`

### 4.3 Google Sheets Credential (Optional — for menu/staff/offers)
1. **Credentials → Add → Google Sheets OAuth2**
2. OAuth flow complete karo
3. Worksheet ID apni sheet ka daalो

Agar Google Sheets use nahi karna toh workflow mein hardcoded data use hoga (default).

---

## STEP 5 — Workflow Test Karo

### 5.1 Workflow Activate
n8n mein workflow ka toggle **ON** karo (top right mein).

### 5.2 n8n Webhook URL Check Karo
Workflow mein **Twilio Webhook** node pe click karo → Production URL copy karo:
```
http://localhost:5678/webhook/twilio-salon
```
(Twilio ke liye yeh ngrok URL se wrap hogi)

### 5.3 WhatsApp Pe Test Karo
Sandbox join karne ke baad ye messages bhejo:

| Bhejo | Expected Reply |
|-------|---------------|
| `hi` | Welcome + options menu |
| `menu` | Services list with prices |
| `book` | Booking form prompt |
| `staff` | Stylist list |
| `offer` | Current deals |
| `status` | Booking check |

### 5.4 n8n mein Debug Karo
- n8n → **Executions** tab → har run ka log dekho
- Kisi node pe click karo → Input/Output data dekho
- Error nodes red highlight honge

---

## STEP 6 — Workflow Node Details

### Webhook URL Pattern
```
POST https://YOUR-NGROK-URL/webhook/twilio-salon
```

### Twilio POST body fields (n8n mein available)
```
Body.From    = whatsapp:+919876543210   (customer ka number)
Body.To      = whatsapp:+14155238886    (sandbox number)
Body.Body    = "hi"                      (message text)
Body.ProfileName = "Rahul"              (customer name)
```

### Twilio ko Reply Bhejne ka Format
n8n se Twilio REST API call:
```
POST https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/Messages.json
Body: {
  From: "whatsapp:+14155238886",
  To: "whatsapp:+919876543210",
  Body: "Reply message text"
}
Auth: Basic (AccountSid : AuthToken)
```

---

## Common Errors & Fixes

| Error | Reason | Fix |
|-------|--------|-----|
| `ngrok tunnel not found` | n8n band ho gaya | `n8n start` phir se chalaao |
| `Twilio 11200 webhook error` | Wrong URL in Twilio | ngrok URL check karo, `/webhook/twilio-salon` lagao |
| `401 Unauthorized` | Wrong credentials | Account SID / Auth Token check karo |
| `n8n workflow not active` | Toggle off | Workflow page pe toggle ON karo |
| ngrok URL changed | ngrok restart kiya | Naya URL Twilio dashboard mein update karo |

---

## Development Workflow (Daily Use)

```bash
# Terminal 1 — n8n start karo
n8n start

# Terminal 2 — ngrok start karo
ngrok http 5678

# Ngrok URL copy karo aur Twilio dashboard mein paste karo
# n8n dashboard: localhost:5678 mein changes karo
# Test: WhatsApp pe message bhejo
```

---

## Production Pe Jaana Hai? (Future)

Local se production pe move karne ke liye:
- n8n ko EC2/VPS pe deploy karo (tumhare paas already AWS EC2 experience hai)
- ngrok ki jagah direct domain + Nginx + SSL use karo
- Twilio Sandbox se real Twilio number pe migrate karo (Meta approval nahi chahiye — Twilio directly deta hai)

---

*Local Development Setup — Twilio WhatsApp Sandbox + n8n v1.0*
