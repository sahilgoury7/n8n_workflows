# 🏆 Apex Lead Qualifier: Live Production Specification

> **Workflow File:** [`Apex Lead Qualifier.json`](file:///d:/n8n%20workflows/Apex%20Lead%20Qualifier.json)  
> **Workflow Name:** `Apex Lead Qualifier`  
> **Target Client:** Apex Academy / High-Ticket Coaching Institutes  
> **Status:** 🟢 **LIVE & TESTED (End-to-End Verified by Sahil)**  
> **Test Results:** WhatsApp Reply ✅ | Supabase DB Update/Create ✅ | Google Sheet Sync ✅ | Telegram Hot Alert ✅  
> **Date Verified:** October 15, 2026  

---

## 🏗️ 1. Complete Workflow Architecture

Workflow do major engines ko ek saath autonomously execute karta hai:
1. **Engine A: RAG Ingestion Pipeline (Drive ➔ Supabase pgvector)**
2. **Engine B: Live WhatsApp Conversation & Lead Qualification Engine**

```text
[Google Drive Trigger: Master Doc] 
        │
        ├──► [Delete Old Chunks (Supabase)] ──► [Download Doc] ──► [Save to Supabase Vector (Gemini 3072)]
        
[WhatsApp Webhook (ChakraHQ)]
        │
        ▼
[Extract Data (Phone, Text, PhoneId)] ──► [Filter: Only User Messages]
        │
        ▼
[AI Agent (Groq / Gemini LLM)] ◄─── (Postgres Chat Memory: 'whatsapp_chat')
        │                      ◄─── (Vector Tool: 'search_knowledge_base' Top-K 6)
        │                      ◄─── (Lead Output Parser: JSON Schema)
        ▼
[Send WhatsApp Reply (ChakraHQ REST API)] ──► (Student receives instant natural answer)
        │
        ▼
[Get Lead by Phone (Supabase 'leads')]
        │
        ▼
[IF: Lead Exists in Supabase?]
        │
        ├──► [TRUE: Existing Lead] 
        │         │
        │         ▼
        │    [Update Lead in Supabase]
        │         │
        │         ▼
        │    [Sync Google Sheet (Existing Lead Row)]
        │         │
        │         ▼
        │    [IF: Newly Hot Lead? (status='hot' AND old_status!='hot')]
        │         │
        │         └──► [TRUE] ──► [🚨 Alert Coaching Staff (Telegram Group -1004348193810)]
        │
        └──► [FALSE: New Lead]
                  │
                  ▼
             [Create Lead in Supabase]
                  │
                  ▼
             [Sync Google Sheet (New Lead Row - Status: 'New')]
                  │
                  ▼
             [Check Newly Hot] ──► [🚨 Alert Coaching Staff if HOT]
```

---

## 🧩 2. Node-by-Node Technical Configuration

### A. Ingestion Engine Nodes:
1. **`New File in Drive Folder`** (`n8n-nodes-base.googleDriveTrigger`)
   - Watches Google Doc: `Apex_Academy_Admission_Assistant_Master_Doc` (`1C7GFPHunDW_vbod-EPq-wqMirjkQ6JZvWXsJf4rgivo`).
   - Trigger mode: `everyMinute` on specific file change.
2. **`Delete Old Chunks`** (`n8n-nodes-base.supabase`)
   - Table: `documents`
   - Filter: `metadata->>file_id=eq.$1` (prevents duplicate chunks on update).
3. **`Download File`** (`n8n-nodes-base.googleDrive`)
   - Export conversion: Google Doc to `text/plain`.
4. **`Save to Supabase`** (`@n8n/n8n-nodes-langchain.vectorStoreSupabase`)
   - Table: `documents`
   - Query function: `match_documents`
   - Chunker: `Recursive Character Text Splitter` (chunk overlap: 200).
   - Embeddings: `Gemini Embeddings` (Model: `gemini-embedding-001`, 3072 dims).

---

### B. WhatsApp Chat & Brain Nodes:
1. **`WhatsApp Webhook`** (`n8n-nodes-base.webhook`)
   - Path: `sahil` (POST)
2. **`Extract Data`** (`n8n-nodes-base.set`)
   - `customer_phone`: `={{ $json.body?.entry?.[0]?.changes?.[0]?.value?.messages?.[0]?.from || $json.body?.payload?.contacts?.[0]?.wa_id }}`
   - `customer_message`: `={{ $json.body?.entry?.[0]?.changes?.[0]?.value?.messages?.[0]?.text?.body || $json.body?.payload?.message?.text?.body }}`
   - `phone_number_id`: `={{ $json.body?.entry?.[0]?.changes?.[0]?.value?.metadata?.phone_number_id || '1125854700621933' }}`
3. **`Only User Messages`** (`n8n-nodes-base.filter`)
   - Filters out status delivery receipts, ensures both message & phone are not empty.
4. **`AI Agent1`** (`@n8n/n8n-nodes-langchain.agent`)
   - **System Prompt:** Apex Academy admission assistant; searches `search_knowledge_base` first; replies in student's language; asks 1 missing detail at a time; outputs structured JSON.
   - **LLM:** `Groq Chat Model` (`openai/gpt-oss-120b`) / Gemini.
   - **Memory:** `Postgres Chat Memory` (`whatsapp_chat` table, sessionKey: `customer_phone`, window: 10).
   - **RAG Tool:** `search_knowledge_base` (Supabase `documents` table, top-K: 6).
   - **Output Parser:** `Lead Output Parser` (`reply`, `lead_name`, `target_course`, `preferred_mode`, `admission_timeline`, `lead_status`, `ai_summary`).
5. **`Send WhatsApp Reply1`** (`n8n-nodes-base.httpRequest`)
   - ChakraHQ endpoint: `https://api.chakrahq.com/v1/ext/plugin/whatsapp/.../messages`
   - Payload: `{"messaging_product":"whatsapp","to":"...","type":"text","text":{"body": reply }}`

---

### C. State Management & Counsellor Alerts:
1. **`Get Lead`** (`n8n-nodes-base.supabase`)
   - Look up `leads` table by `phone_number == customer_phone`.
2. **`Lead Exists?`** (`n8n-nodes-base.if`)
   - Condition: `$json.id exists`.
3. **`Update Lead` / `Create Lead`** (`n8n-nodes-base.supabase`)
   - Smart field fallback: Keeps existing values if new message didn't specify them (`|| $("Get Lead").item.json...`).
   - Updates `updated_at` with ISO timestamp.
4. **`Sync Sheet (Existing / New)`** (`n8n-nodes-base.googleSheets`)
   - Document ID: `1r9Umi6BsbnN8LUw-HkpWTxfyVd_hOUzOnvc5NU3nIdk` (`Sheet1`).
   - Operation: `appendOrUpdate` matching on `WhatsApp Number`.
   - Fields: Date & Time, Lead Name, WhatsApp Number, Target Course, Preferred Mode, Admission Timeline, AI Lead Status, AI Summary, 1-Click WhatsApp Link (`https://wa.me/...`), Counsellor Status (`New`).
5. **`Newly Hot Lead?`** (`n8n-nodes-base.if`)
   - **Genius Edge-Case Prevention:** Checks `lead_status == 'hot' AND old_lead_status != 'hot'`.
   - Prevents duplicate alerts if an already hot lead sends further follow-up questions.
6. **`Alert Coaching Staff`** (`n8n-nodes-base.telegram`)
   - Destination Group: `-1004348193810`
   - Content: Rich lead notification card with 1-click WhatsApp chat link.

---

## 🔐 3. Verified Credentials Used in Workflow
| Credential Name in n8n | Credential Type | Used In |
|---|---|---|
| `Google Drive account` | `googleDriveOAuth2Api` | Drive Trigger & Document Download |
| `Supabase account` | `supabaseApi` | RAG Ingest, Vector Tool, Leads Get/Insert/Update |
| `Google Gemini(PaLM) Api account` | `googlePalmApi` | Gemini Embeddings (3072 dims) |
| `Groq account` | `groqApi` | AI Agent LLM |
| `Postgres account` | `postgres` | `whatsapp_chat` Memory |
| `Google Sheets account` | `googleSheetsOAuth2Api` | Sheet CRM Append / Update |
| `Telegram account` | `telegramApi` | Counsellor Group Alerts |
| `ChakraHQ Bearer Token` | HTTP Header Auth | Outbound WhatsApp Delivery |

---

## 📈 4. Real-world Verification Proof
- Student WhatsApp Chat: Received instantaneous, accurate course response.
- Supabase `leads`: Successfully updated student details & status.
- Google Sheets: Created row with 1-Click WhatsApp link.
- Telegram Group (`-1004348193810`): Instant siren alert delivered.
