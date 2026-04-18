# 💈 Glamour Unisex Salon — WhatsApp Chatbot Setup Guide
> Stack: WhatsApp Cloud API + n8n + Google Sheets (No custom backend needed!)

---

## STEP 1 — Meta Business & WhatsApp Cloud API Setup

### 1.1 Meta Developer Account
1. Go to https://developers.facebook.com
2. Create an App → Select **Business** type
3. Add **WhatsApp** product to your app

### 1.2 WhatsApp Business Account
1. Go to **App Dashboard → WhatsApp → Getting Started**
2. Note down:
   - `Phone Number ID` (e.g. 1234567890)
   - `WhatsApp Business Account ID`
   - `Access Token` (Temporary — later create Permanent Token)

### 1.3 Create Permanent Access Token
1. Go to **Business Settings → System Users**
2. Create a System User with **Admin** role
3. Generate Token → Select your App → Give `whatsapp_business_messaging` permission
4. Copy and save this token safely!

---

## STEP 2 — Google Sheets Setup

Create a new Google Sheet with these 4 tabs:

### Tab 1: `Services`
| Column A | Column B | Column C | Column D |
|----------|----------|----------|----------|
| Category | Service | Price | Duration |
| Hair | Haircut (Men) | 200 | 30 min |
| Hair | Haircut (Women) | 350 | 45 min |
| Hair | Hair Color | 800 | 90 min |
| Skin | Facial | 600 | 60 min |
| Skin | Cleanup | 300 | 30 min |
| Nails | Manicure | 400 | 45 min |
| Nails | Pedicure | 500 | 60 min |

### Tab 2: `Staff`
| Column A | Column B | Column C | Column D |
|----------|----------|----------|----------|
| Name | Speciality | Experience | Available |
| Priya Sharma | Hair Color & Styling | 5 years | Yes |
| Rahul Verma | Men's Haircut & Beard | 4 years | Yes |
| Sneha Patil | Skin & Facials | 3 years | Yes |

### Tab 3: `Offers`
| Column A | Column B | Column C | Column D |
|----------|----------|----------|----------|
| Title | Description | Discount | ValidTill |
| Weekend Special | Haircut + Wash combo | 20% | 30 Apr 2025 |
| First Visit Offer | Any service first time | 15% | 31 May 2025 |
| Summer Glow | Facial + Cleanup | 25% | 15 Jun 2025 |

### Tab 4: `Appointments`
| Column A | Column B | Column C | Column D | Column E | Column F | Column G |
|----------|----------|----------|----------|----------|----------|----------|
| Phone | Name | Date | Time | Service | Stylist | Status |
| 919876543210 | Priya | 20/04/2025 | 11:00 AM | Haircut | Rahul | Confirmed |

> Note: Phone number format should be country code + number (no + sign)
> Example: 919876543210 for Indian number +91 98765 43210

---

## STEP 3 — n8n Setup

### 3.1 Install n8n (Self-hosted on your server/EC2)
```bash
npm install n8n -g
n8n start
# OR with Docker:
docker run -it --rm --name n8n -p 5678:5678 n8nio/n8n
```

### 3.2 Import the Workflow
1. Open n8n → **Workflows → Import from File**
2. Upload `salon_whatsapp_n8n_workflow.json`
3. Click **Save**

### 3.3 Configure Google Sheets Credential
1. In n8n → **Credentials → Add Credential → Google Sheets OAuth2**
2. Follow OAuth flow to connect your Google account
3. Go to each Google Sheets node → Select this credential
4. Replace `YOUR_GOOGLE_SHEET_ID` with your actual Sheet ID
   - Sheet ID is in the URL: `docs.google.com/spreadsheets/d/SHEET_ID_HERE/edit`

### 3.4 Configure WhatsApp Token
Option A — Use n8n Credentials:
1. n8n → Credentials → Add → **Header Auth**
2. Name: `WhatsApp Token`
3. Value: `Bearer YOUR_PERMANENT_TOKEN`
4. Assign to **Send WhatsApp Reply** HTTP node

Option B — Use n8n Environment Variable:
```bash
N8N_WHATSAPP_TOKEN=your_token_here n8n start
```
Then reference as `{{ $env.N8N_WHATSAPP_TOKEN }}` in the node.

### 3.5 Activate the Workflow
1. Toggle workflow to **Active**
2. Note your Webhook URL:
   `https://your-n8n-domain.com/webhook/salon-whatsapp`

---

## STEP 4 — Connect Webhook to WhatsApp

### 4.1 Set Webhook in Meta Dashboard
1. Go to **WhatsApp → Configuration → Webhook**
2. Click **Edit**
3. Set:
   - **Callback URL**: `https://your-n8n-domain.com/webhook/salon-whatsapp`
   - **Verify Token**: Any random string (e.g. `salon123`)
4. Subscribe to: `messages`

### 4.2 Handle Webhook Verification
WhatsApp sends a GET request with `hub.challenge` to verify your webhook.
The workflow handles this automatically via the Parse Message node.

> ⚠️ Your n8n must be HTTPS (use Nginx + Let's Encrypt or ngrok for testing)

---

## STEP 5 — Test Your Chatbot

Send these messages to your WhatsApp Business number:

| Message | Expected Response |
|---------|------------------|
| `hi` | Welcome + menu options |
| `menu` | Full services with prices |
| `book` | Booking form prompt |
| `staff` | Stylist profiles |
| `offer` | Current deals |
| `status` | Booking lookup |
| `random text` | Fallback menu |

---

## STEP 6 — Handle Booking Submissions

When a customer sends booking details, add a **second Switch node** in n8n:
- Detect if message contains `Naam:` pattern
- Parse the booking details using a Code node
- Append to Google Sheets `Appointments` tab
- Send confirmation reply

Example Code node for parsing booking:
```javascript
const text = $json.text;
const lines = text.split('\n');
const booking = {};
lines.forEach(line => {
  if (line.includes('Naam:')) booking.Name = line.split('Naam:')[1].trim();
  if (line.includes('Date:')) booking.Date = line.split('Date:')[1].trim();
  if (line.includes('Time:')) booking.Time = line.split('Time:')[1].trim();
  if (line.includes('Service:')) booking.Service = line.split('Service:')[1].trim();
  if (line.includes('Stylist:')) booking.Stylist = line.split('Stylist:')[1].trim();
});
booking.Phone = $json.from;
booking.Status = 'Confirmed';
return [{ json: { ...$json, booking } }];
```

---

## Production Checklist

- [ ] Permanent WhatsApp access token generated
- [ ] Google Sheets connected and populated
- [ ] n8n running on HTTPS domain
- [ ] Webhook verified in Meta dashboard
- [ ] All 4 Google Sheet tabs created with correct column names
- [ ] Workflow active in n8n
- [ ] Tested all 6 intents manually
- [ ] Added salon name in greeting message (currently "Glamour Unisex Salon")

---

## Folder Structure Reference

```
n8n Workflow Nodes:
├── WhatsApp Webhook (Trigger)
├── Respond 200 OK (Instant ack to Meta)
├── Parse Message & Detect Intent (Code)
├── Skip if no message (IF)
├── Intent Router (Switch) — 7 outputs
│   ├── Greeting → Send Reply
│   ├── Book → Booking Prompt → Send Reply
│   ├── Menu → Google Sheets → Format → Send Reply
│   ├── Staff → Google Sheets → Format → Send Reply
│   ├── Offers → Google Sheets → Format → Send Reply
│   ├── Status → Google Sheets → Format → Send Reply
│   └── Unknown → Fallback Reply → Send Reply
└── Send WhatsApp Reply (HTTP Request to Meta API)

Google Sheets Tabs:
├── Services (Category, Service, Price, Duration)
├── Staff (Name, Speciality, Experience, Available)
├── Offers (Title, Description, Discount, ValidTill)
└── Appointments (Phone, Name, Date, Time, Service, Stylist, Status)
```

---

*Built for Glamour Unisex Salon — WhatsApp Chatbot v1.0*
