# 🎓 Coaching Institute WhatsApp AI Admission & Lead Qualifier System

> **Enterprise-Grade n8n Production Workflow for High-Ticket Coaching Institutes (JEE / NEET / UPSC / Foundation)**  
> **Tech Stack:** n8n + ChakraHQ (WhatsApp BSP) + Supabase (pgvector & CRM) + Google Gemini / OpenAI + Google Sheets + Telegram Siren Alert

---

## 📌 1. Overview & Business Problem

High-ticket coaching institutes (course fee ₹40,000 – ₹1,50,000+) spend lakhs on Facebook/Instagram ads, but lose up to **40% of prospective student leads** due to:
1. **Delayed Replies:** Students enquiry ke baad raat ko ya weekends par ghanto intezar karte hain aur doosre institute me register ho jaate hain.
2. **Counsellor Fatigue:** Counsellors apna 70% time same basic queries (fees, batch timing, hostel, scholarship) repeat karne me waste karte hain.
3. **No Intent Filtering:** Cold inquiries aur genuine admission-ready students me farak nahi pata chalta.

### The Solution:
Ek 24/7 intelligent WhatsApp AI Admission Counsellor jo:
- Student ke queries ka natural Hinglish mein instant reply karta hai using **RAG Vector Search** (Brochure & Fee Schedule).
- Student ko conversational style mein qualify karta hai: **Student Name ➔ Target Course ➔ Mode (Offline/Online) ➔ Admission Timeline**.
- Supabase database aur Google Sheets CRM me lead dynamically update ya create karta hai.
- Agar student **HOT / Admission-Ready** hai, toh 5 second ke andar Counsellor ke Telegram group par **🚨 Instant Siren Alert** trigger karta hai (with direct 1-click Call & WhatsApp links).

---

## 🏗️ 2. Workflow Architecture

```text
                  [ChakraHQ Inbound Webhook]
                              │
                              ▼
                [Set Node: Extract Phone, Name, Text]
                              │
                              ▼
                     [AI Agent Counsellor]
                      ├── LLM (Gemini 2.0 Flash / OpenAI)
                      ├── Window Buffer Memory (Session Continuity)
                      └── Vector Store Tool (Supabase RAG Knowledge Base)
                              │
                              ▼
            [Send WhatsApp Reply (ChakraHQ API)]
                              │
                              ▼
                 [Format Lead Qualification Data]
                              │
                              ▼
                 [Supabase: Check Existing Lead]
                              │
               ┌──────────────┴──────────────┐
               ▼                             ▼
        [Lead Exists?]                [New Student]
        Update Supabase               Create Supabase
               └──────────────┬──────────────┘
                              │
                              ▼
                  [Append to Google Sheets CRM]
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

## 🧠 3. Lead Qualification Matrix

| Qualification Metric | Field Name | Criteria |
|---|---|---|
| **🔴 HOT Lead** | `lead_status: "HOT"` | `admission_timeline` = *"Immediate"* YA *"Demo Class"*, Name & Target Course confirmed. **Action:** Instant Telegram Siren Alert + CRM Log. |
| **🟡 WARM Lead** | `lead_status: "WARM"` | Student asked fees & batches, `admission_timeline` = *"Next Month"*. **Action:** CRM Log for counsellor scheduled follow-up. |
| **⚪ COLD Lead** | `lead_status: "COLD"` | Generic "Hi/Hello" or incomplete qualification. **Action:** Silent CRM Log. |

---

## 🗄️ 4. Supabase Database Schema

```sql
CREATE EXTENSION IF NOT EXISTS vector;

-- Table: public.leads
CREATE TABLE public.leads (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    phone_number TEXT NOT NULL UNIQUE,
    lead_name TEXT,
    target_course TEXT,
    preferred_mode TEXT,
    admission_timeline TEXT,
    lead_status TEXT DEFAULT 'WARM',
    ai_summary TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Table: public.documents (RAG Knowledge Chunks)
CREATE TABLE public.documents (
    id BIGSERIAL PRIMARY KEY,
    content TEXT,
    metadata JSONB,
    embedding VECTOR(3072)
);
```

---

## 🚀 5. How to Setup & Import in n8n

### Step 1: Import Workflow JSON
1. n8n dashboard open karein.
2. Top-right menu se **Import from File** choose karein.
3. Select karein: [`Apex_Lead_Qualifier.json`](./Apex_Lead_Qualifier.json).

### Step 2: Configure Credentials in n8n
- **ChakraHQ / WhatsApp API:** Inbound Webhook URL copy karke ChakraHQ portal me paste karein. Outbound API Auth Header set karein.
- **Supabase Credentials:** URL and Service Role Key add karein.
- **Google Gemini / OpenAI API:** API Key configure karein chat model node ke liye.
- **Google Sheets:** OAuth2 credentials connect karein CRM sheet sync ke liye.
- **Telegram Bot:** Bot Token aur Counsellor Group Chat ID configure karein.

### Step 3: Knowledge Base Ingestion
- Upload institute brochure, fee structure, and batch schedule via the automated Drive-to-Supabase ingestion branch or run vector store indexing.

---

## 👤 Author & Maintainer
- **Agency Founder:** Sahil Goury  
- **LinkedIn:** [Sahil Goury](https://www.linkedin.com/in/sahil-goury-269107370/)  
- **GitHub:** [@sahilgoury7](https://github.com/sahilgoury7)  
- **Email:** sahilgoury7@gmail.com

