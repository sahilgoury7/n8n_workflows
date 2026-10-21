# ⚡ Production-Grade n8n AI WhatsApp Automation Workflows

Welcome to the **n8n Production AI Automation Workflows** repository.  
This repository contains battle-tested, high-conversion WhatsApp AI workflows engineered for B2B client acquisition, lead qualification, and instant sales team notification.

---

## 📁 Repository Structure

```text
n8n_workflows/
│
├── 🎓 coaching-institute-ai-admission/      # High-Ticket Coaching WhatsApp AI Admission System
│   ├── Apex_Lead_Qualifier.json            # Full n8n Workflow (Drive RAG + Supabase + Telegram)
│   └── README.md                           # Architecture, Database Schema, & Setup Guide
│
├── 🏢 real-estate-ai-sales-advisor/         # Real Estate WhatsApp AI Sales Engine
│   ├── Real_Estate_Lead_Qualifier.json     # Full n8n Workflow (21 Nodes, 10s Dedup, Sheets CRM)
│   └── README.md                           # Architecture, 1-Question Rule, & Setup Guide
│
├── .gitignore                              # Clean Git Ignore
└── README.md                               # Master Repository Documentation (this file)
```

---

## 🚀 Workflows Overview

### 1. 🎓 [Coaching Institute AI Admission System](./coaching-institute-ai-admission/)
- **Target Client:** Mid-to-Large Coaching Institutes (JEE / NEET / UPSC / Foundation, Course Fees > ₹40,000).
- **Core Engine:** RAG Vector Search across Institute Brochures & Fee Schedules + Window Buffer Memory + Dynamic Supabase Lead Update/Create.
- **Outcome:** Eliminates 70% of counsellor repetitive queries and alerts counsellors via Telegram Siren within 5 seconds of a high-intent admission inquiry.
- 👉 [View Detailed Coaching Documentation & Setup Guide](./coaching-institute-ai-admission/README.md)

### 2. 🏢 [Real Estate AI Sales Advisor](./real-estate-ai-sales-advisor/)
- **Target Client:** Real Estate Channel Partners (CPs) & Local Builders (Ticket size ₹40L – ₹1.5 Cr).
- **Core Engine:** 21-Node Lean Architecture + **10-Second Webhook Deduplication Guard** + Strict 1-Question Rule Qualification Sequence (`Name ➔ BHK ➔ Budget ➔ Purpose ➔ Site Visit`) + Live Google Sheets CRM + Telegram Siren Alert.
- **Outcome:** Recovers 30%–40% of leads lost to delayed WhatsApp replies and automatically books site visits for active property ad campaigns.
- 👉 [View Detailed Real Estate Documentation & Setup Guide](./real-estate-ai-sales-advisor/README.md)

---

## 🛡️ Enterprise Engineering Standards Applied

1. **Quota-Safe Architecture:** Background polling triggers removed to prevent accidental execution burn on free/starter tiers.
2. **Meta Deduplication Guard:** In-memory 10-second deduplication node prevents duplicate replies caused by WhatsApp webhook retry storms.
3. **Instant 200 OK Webhook Handshake:** Prevents external webhook timeout disconnects.
4. **Strict 1-Question Rule:** Ensures conversational flow without overwhelming prospects with multiple questions.
5. **Direct 1-Click Action Links:** Telegram siren alerts and Google Sheets CRM include direct `wa.me` links for instant one-click outreach.

---

## 🛠️ Quickstart: How to Import into n8n

1. Clone or download this repository:
   ```bash
   git clone https://github.com/sahilgoury7/n8n_workflows.git
   ```
2. In your **n8n Dashboard**, go to **Workflows ➔ Import from File**.
3. Select either:
   - `coaching-institute-ai-admission/Apex_Lead_Qualifier.json`
   - `real-estate-ai-sales-advisor/Real_Estate_Lead_Qualifier.json`
4. Attach your API credentials (WhatsApp BSP / ChakraHQ, LLM API Key, Supabase / Google Sheets, and Telegram Bot).
5. Activate the workflow and configure your inbound webhook!

---

## 👤 Author & Maintainer
- **Agency Founder:** Sahil Goury  
- **LinkedIn:** [Sahil Goury](https://www.linkedin.com/in/sahil-goury-269107370/)  
- **GitHub:** [@sahilgoury7](https://github.com/sahilgoury7)  
- **Email:** sahilgoury7@gmail.com

