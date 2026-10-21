# 🏢 Real Estate WhatsApp AI Sales Advisor & Lead Qualifier

> **Production-Ready, Ultra-Lean (21 Nodes) 24/7 Real Estate Lead Qualification & Booking Engine**  
> **Tech Stack:** n8n + ChakraHQ (WhatsApp Cloud API) + Groq (Llama-3.3-70b) / Gemini 2.0 Flash + Supabase (RAG Knowledge Base) + Google Sheets CRM + Telegram Instant Siren Alert

---

## 📌 1. Overview & Business Problem Solved

Real estate Channel Partners (CPs) aur local builders active Meta/Instagram property ads par daily ₹2,000 se ₹10,000 kharch karte hain.
Lekin:
- **Delayed Replies:** Shaam ko 8 baje ke baad ya weekend par buyer WhatsApp par message karta hai, aur telecaller available na hone se **30%–40% buyers doosre broker ke paas chale jaate hain.**
- **Telecaller Burnout:** Sales team apna 80% time cold aur casual inquiry se "Budget kitna hai" puchne me barbad karti hai.
- **Missed Site Visits:** Ready buyers ka site visit time par schedule nahi ho pata.

### The Solution:
Ek 24/7 intelligent WhatsApp AI Property Advisor jo:
1. **5-Second Instant Reply:** Buyer ke language (Hindi, English, ya Hinglish) me warm aur accurate jawab deta hai.
2. **Strict 1-Question Rule:** Buyer par sawalo ki baarish nahi karta; ek baar me sirf 1 logical qualification step complete karta hai:  
   `Name ➔ Property Type (2BHK/3BHK) ➔ Budget ➔ Purpose ➔ Site Visit`.
3. **10-Second Webhook Deduplication Guard:** Meta/WhatsApp retry storms ya duplicate clicks ko silently drop karta hai taaki customer ko ek hi message 2 baar na jaye.
4. **Instant CRM Append:** Live Google Sheets CRM me row update hoti hai with 1-click direct WhatsApp chat link.
5. **🚨 Telegram Siren Alert:** Jaise hi buyer high-intent confirm karta hai (Budget & Site Visit), 5 second ke andar Sales Head ke phone par Telegram Siren Alert bajta hai.

---

## 🏗️ 2. Workflow Architecture

```text
                  [ChakraHQ Inbound Webhook]
                              │
                              ▼
                  [Respond 200 OK (Instant)]
                              │
                              ▼
              [10-Second Deduplication Guard]
          (Drops duplicate retries within 10s window)
                              │
                              ▼
            [Edit Fields: Extract Phone, Name, Text]
                              │
                              ▼
                     [AI Real Estate Agent]
                   ├── LLM (Groq Llama-3.3-70b / Gemini)
                   ├── Window Buffer Memory (Session state)
                   └── RAG Tool (Floor plans, pricing & amenities)
                              │
                              ▼
            [Send WhatsApp Reply (ChakraHQ API)]
                              │
                              ▼
                 [Format Lead Qualification Data]
                              │
                              ▼
                  [Google Sheets CRM Sync]
                  (Appends / Updates Buyer Record)
                              │
                              ▼
                   [IF Node: Is Lead HOT?]
                              │
            ┌─────────────────┴─────────────────┐
            ▼                                   ▼
      [TRUE: Hot Lead]                  [FALSE: Cold/Warm]
  [🚨 Telegram Siren Alert]                   [End]
```

---

## 🎯 3. Qualification Sequence & System Prompt Rules

1. **GREETINGS ('hi', 'hello', 'namaste'):** Warm 1-line greet, DO NOT search knowledge base, ask name: *"Kya main aapka shubh naam jaan sakta hoon?"*
2. **PROPERTY QUERIES (pricing, amenities, possession, location):** Call RAG knowledge base and answer strictly from returned facts.
3. **STRICT 1-QUESTION RULE:** Never ask more than 1 question in a single message.
4. **QUALIFICATION LADDER:**  
   `Name ➔ Property Type (2BHK/3BHK) ➔ Budget Range ➔ Purpose (Self-Use/Investment) ➔ Site Visit Date`.

---

## 🛡️ 4. Resilience & Quota-Safe Optimizations

- **0 Execution Leakage:** Ingestion polling triggers have been decoupled. Zero background executions burning your free n8n cloud quota.
- **10-Second Deduplication Node:** JavaScript memory cache node that checks `customer_phone + message` within a 10,000ms window to eliminate duplicate webhook executions.
- **Fast 200 OK Response:** ChakraHQ webhook receives an immediate 200 OK status to avoid WhatsApp message delivery timeout retries.

---

## 🚀 5. How to Setup & Import in n8n

### Step 1: Import Workflow
1. n8n dashboard me jaakar **Import from File** click karein.
2. Select karein: [`Real_Estate_Lead_Qualifier.json`](./Real_Estate_Lead_Qualifier.json).

### Step 2: Configure Credentials in n8n
- **ChakraHQ / WhatsApp API:** Inbound Webhook URL copy karke ChakraHQ portal me paste karein.
- **Groq / Google Gemini:** Configure your LLM API Key in the chat model node.
- **Google Sheets:** Authenticate Google Sheets OAuth2 to sync leads with your Master CRM.
- **Telegram Bot:** Enter your Bot Token and Sales Team Group ID (`-100...`).

### Step 3: Sales Kit & Client Acquisition
Is folder me complete Agency Go-To-Market execution kit shamil hai:
- 📋 [DAILY_EXECUTION_CHECKLIST_AND_ROADMAP.md](./DAILY_EXECUTION_CHECKLIST_AND_ROADMAP.md) — 7-Day Sprint, Cold Calling Script, 25-Min Doctor Framework Closing Meeting, Onboarding Checklist & Credentials Handover Template.
- 🎯 [PROSPECTING_GUIDE_AND_SHEET_TEMPLATE.md](./PROSPECTING_GUIDE_AND_SHEET_TEMPLATE.md) — Meta Ads Library Lead Scraping Blueprint & 12-Column Google Sheet Prospect Tracker.
- 🏗️ [PROJECT_PLAN_AND_TRACKER.md](./PROJECT_PLAN_AND_TRACKER.md) — Architecture & Schemas.
