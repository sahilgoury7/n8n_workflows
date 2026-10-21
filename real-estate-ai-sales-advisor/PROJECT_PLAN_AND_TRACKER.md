# 🏢 Real Estate WhatsApp AI Sales Engine & Lead Qualifier
## Master Project Plan & Execution Tracker

> **Client Domain:** Real Estate Channel Partners (CPs) & Local Builders (Ticket Size > ₹40L - ₹1.5 Cr)  
> **Agency Lead:** Sahil  
> **Architect:** Senior n8n Automation Architect  
> **Tech Stack:** ChakraHQ (WhatsApp Cloud BSP) + n8n + Supabase + Google Sheets + Groq / Gemini + Telegram Siren Alert  
> **Created Date:** October 2026  
> **Current Status:** ✅ Production Ready & Client Acquisition Sprint Active  

---

## 📌 1. Project Overview & Objective

### The Core Problem (Real Estate Pain):
- **Delayed Replies:** Raat ko 8 baje ke baad ya weekend par buyer ads dekh kar WhatsApp par "Hi / Details please" likhta hai, lekin 4-5 ghante koi reply nahi milta.
- **Lead Leakage:** Buyer 5 minute me dusre broker ya competitor project par chala jata hai (30-40% lead drop).
- **No Instant Qualification:** Telecallers din bhar random inquiry ko call karke time waste karte hain aur genuine high-budget buyers miss ho jate hain.
- **Missed Site Visits:** Ready buyers ki site visit schedule nahi ho pati.

### The Solution:
Ek 24/7 intelligent WhatsApp Real Estate AI Sales Advisor jo:
1. Buyer ke queries ka natural Hindi/English/Hinglish mein 5-second me instant reply karega (using RAG Knowledge Base).
2. Strict 1-question rule se buyer ko qualify karega: **Name ➔ Property Type (2BHK/3BHK) ➔ Budget ➔ Purpose (Self-use/Investment) ➔ Site Visit Date**.
3. Google Sheets CRM me lead dynamically append / update karega (1-click direct WhatsApp chat link ke saath).
4. Jaise hi lead **HOT** hoti hai (Budget & Site Visit confirm), turant Sales Head ke Telegram par **🚨 Instant Siren Alert** bhejega.

---

## 🏗️ 2. Approved Workflow Architecture (Real_Estate_Lead_Qualifier.json)

```text
[Webhook (ChakraHQ)]
       │
       ▼
[Respond 200 OK (Instant)] 
       │
       ▼
[10-Sec Deduplication Guard] ─── (Drops duplicate webhook retries within 10s)
       │
       ▼
[Edit Fields / Set Node] ─────── (Extracts Phone, Name, Message)
       │
       ▼
[AI Agent] ◄──────────────────── (Groq Llama-3.3-70b / Gemini 2.0 Flash)
       │   ◄──────────────────── (Window Buffer Memory)
       │   ◄──────────────────── (Vector Store Tool - Real Estate Knowledge Base)
       │
       ▼
[Send WhatsApp Message (ChakraHQ)] ── (Instant personalized reply to buyer)
       │
       ▼
[Format Lead Data] ─────────────── (Prepares Name, BHK, Budget, Purpose, Site Visit, Intent)
       │
       ▼
[Google Sheets CRM Append] ─────── (Appends/Updates Master Lead Sheet)
       │
       ▼
[IF Node: Is Lead HOT?]
       │
    [TRUE] ───────────┴─────────── [FALSE]
       │                              │
       ▼                              ▼
[Telegram Siren Alert]              [End]
(🚨 HOT REAL ESTATE LEAD ALERT!)
```

---

## 🔑 3. Tools & Credentials Status Tracker

| Tool / Service | Purpose | Status | Notes |
|---|---|---|---|
| **ChakraHQ** | WhatsApp Inbound Webhook & Outbound API | ✅ Tested & Working | Webhook verified, additional webhooks cleared |
| **n8n Workflow** | Core Automation & Logic Engine | ✅ 21 Nodes Optimized | 0 background execution leaks, 10s deduplication guard active |
| **LLM (Groq / Gemini)** | Real Estate AI Conversation | ✅ Active | Configured with Strict 1-Question Rule & Real Estate System Prompt |
| **Knowledge Base (RAG)** | Floor Plans & Project Amenities | ✅ Active | Real estate property details ingested in Supabase vector store |
| **Google Sheets CRM** | Live Buyer CRM for Client | ✅ Connected & Tested | Dynamic row append with 1-click WhatsApp link |
| **Telegram Alert Bot** | Instant Siren Alert for Sales Team | ✅ Active | Alerts sales executives within 5 seconds of qualification |

---

## 📅 4. Agency Go-To-Market & Sales Execution

- 📋 **Daily Execution & Sprint Roadmap:** [DAILY_EXECUTION_CHECKLIST_AND_ROADMAP.md](file:///d:/n8n%20workflows/DAILY_EXECUTION_CHECKLIST_AND_ROADMAP.md) ⭐ *(Daily Routine, 20-25 Calls, 25-Min Doctor Framework Closing, Onboarding Checklist)*
- 🎯 **Lead Sourcing & Prospect Tracker Guide:** [PROSPECTING_GUIDE_AND_SHEET_TEMPLATE.md](file:///d:/n8n%20workflows/PROSPECTING_GUIDE_AND_SHEET_TEMPLATE.md)
- **Active Workflow File:** [`Real_Estate_Lead_Qualifier.json`](file:///d:/n8n%20workflows/Real_Estate_Lead_Qualifier.json)
