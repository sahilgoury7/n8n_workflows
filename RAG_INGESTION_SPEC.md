# 📚 RAG Ingestion Pipeline: Google Drive to Supabase

> **Workflow Name:** [RAG Ingestion: Google Drive to Supabase](https://goury.app.n8n.cloud/workflow/pkMurkGi2xIaZyfM)  
> **Workflow ID:** `pkMurkGi2xIaZyfM`  
> **Status:** Draft (Dev / Ingestion Complete)  
> **Project:** Sahil (Personal)  
> **Master Document:** `Apex_Academy_Admission_Assistant_Master_Doc` (Google Doc)

---

## 🎯 1. Maqsad (Objective)
Google Drive ke master doc (`Apex_Academy_Admission_Assistant_Master_Doc`) ko automate karke read karna, chunks mein todna, Google Gemini se embeddings banakar **Supabase pgvector** database mein save karna. 

Is data ko **WhatsApp Admission AI Agent** as a tool use karta hai taaki courses, fees, batch timings aur syllabus se jude sawaalon ke accurate aur verified answers de sake.

---

## ⚙️ 2. Flow 1: Ingestion Pipeline (Drive ➔ Supabase)

| Step | Node Name | Node Type | Kaam & Configuration |
|:---:|---|---|---|
| **1** | **New File in Drive Folder** | `Google Drive Trigger` | `Apex_Academy_Admission_Assistant_Master_Doc` ko har minute check karta hai, aur update/change hone par trigger hota hai. |
| **2** | **Download File** | `Google Drive` | Google Doc ko plain text format mein export aur download karta hai. |
| **3** | **Save to Supabase** | `Supabase Vector Store` | Operation: **Insert Documents**. Chunks aur embeddings ko Supabase ki `documents` table mein insert karta hai. |

### ↳ Sub-nodes (Connected to "Save to Supabase"):
1. **Document Loader:** File ka plain text leta hai, aur metadata mein `file_name` aur `file_id` enrich karta hai.
2. **Chunk Text (`Recursive Character Text Splitter`):**
   - Chunk Size: `1000` (Default)
   - Chunk Overlap: `200`
3. **Gemini Embeddings (`Embeddings Google Gemini`):**
   - Model: `gemini-embedding-001`
   - Vector Dimensions: **3072**

---

## 🤖 3. Flow 2: Chat Agent (Reference Testing Flow)

| Node Name | Node Type | Kaam & Configuration |
|---|---|---|
| **When chat message received** | `Chat Trigger` | User ka test chat message receive karta hai. |
| **AI Agent** | `AI Agent` (LangChain) | Prompt: `{{ $json.chatInput }}`.<br>**System Message:** *Apex Academy ka admission assistant, pehle knowledge tool use karna, guess nahi karna, user ki bhasha mein jawab dena.* |
| **Google Gemini Chat Model** | `Google Gemini Chat Model` | Core LLM engine jo response formulate karta hai. |
| **Chat Memory** | `Window Buffer / Simple Memory` | Pichhle 10 messages yaad rakhta hai context ke liye. |
| **Supabase Vector Store (Tool)** | `Vector Store Tool` | Operation: **Retrieve as Tool**.<br>Table: `documents`.<br>Top K: `6` chunks dhoondhta hai. |
| **Embeddings Google Gemini** | `Embeddings Google Gemini` | Sawaal ko embed karta hai (Same model: `gemini-embedding-001`, 3072 dims). |

---

## 🔑 4. Connected Credentials
1. **Google Drive OAuth2:** Google Drive Account (doc access ke liye)
2. **Google Gemini / PaLM API:** Google Gemini(PaLM) Api account (LLM + Embeddings ke liye)
3. **Supabase API:** Supabase Account (pgvector storage ke liye)

---

## 🗄️ 5. Supabase Database Configuration
- **Extension:** `pgvector`
- **Table Name:** `public.documents`
  - Columns:
    - `id` (bigint generated always as identity)
    - `content` (text)
    - `metadata` (jsonb)
    - `embedding` (`vector(3072)`)
- **Search Function:**
  ```sql
  create or replace function match_documents (
    query_embedding vector(3072),
    match_count int default 6,
    filter jsonb default '{}'
  ) returns table (
    id bigint,
    content text,
    metadata jsonb,
    similarity float
  )
  language plpgsql
  as $$
  #variable_conflict use_column
  begin
    return query
    select
      id,
      content,
      metadata,
      1 - (documents.embedding <=> query_embedding) as similarity
    from documents
    order by documents.embedding <=> query_embedding
    limit match_count;
  end;
  $$;
  ```
> ⚠️ **Technical Alert:** 2000 se zyada dimensions (`3072`) par Postgres mein standard `HNSW` ya `IVFFlat` index nahi banta. Direct cosine similarity (`<=>`) sequential scan use hota hai (medium datasets < 50k rows ke liye ye super fast aur standard hai).

---

## 📊 6. Execution & Test Status
- **Live Ingestion Test:** ✅ **PASSED** (Execution #60 mein **36 chunks** successfully embed hokar Supabase mein save hue).
- **Knowledge Data:** Apex Academy ke courses, fees, syllabus aur batch timings Supabase mein active hain.

---

## ⚠️ 7. Known Limitations & Production Notes
1. **Trigger Scope:** Drive trigger currently sirf specific file dekhta hai, poora folder nahi (MVP ke liye perfect hai).
2. **Re-run Deduplication:** Agar Google Doc dobara update hota hai, toh purane chunks delete kiye bina naye chunks insert hote hain. (Production mein update se pehle `DELETE FROM documents WHERE metadata->>'file_id' = ...` laga sakte hain).
3. **Draft Status:** Workflow currently unpublished hai (manual test run par verified).
4. **Integration with Lead Agent:** Main WhatsApp Lead Qualifier workflow (`AQjegnLPnqKAWERi`) mein ye same **Vector Store Tool (Supabase `documents` + `gemini-embedding-001` 3072)** as a tool plug hoga!
