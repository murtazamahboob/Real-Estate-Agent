# 🏠 Diamond Real Estate — AI-Powered Property Platform Automation

An all-in-one n8n workflow powering a real estate business: admin property management, B2B lead intake, a 24/7 AI chatbot with FAQ + database tools, and a voice-agent (ElevenLabs) integration — all backed by Supabase.

> ⚠️ **Before you deploy:** this exported JSON has all credentials, API keys, and personal emails replaced with placeholders (`YOUR_SUPABASE_CREDENTIAL_ID`, `YOUR_ADMIN_EMAIL@example.com`, etc.). Fill these in with your own n8n credentials after importing — never commit real keys to a public repo.

---

## 🧠 What This Workflow Does

The workflow is organized into four independent modules, all sharing the same Supabase database.

### 1️⃣ Property Admin Management (Add / Update / Delete)
Three simple webhook-triggered flows let an admin manage listings straight from a website UI, with zero SQL knowledge required:
- **Add** — `POST /admin/add-property` → inserts a new row into the `properties` table in Supabase.
- **Update** — `POST /admin/update-property` → updates an existing property record.
- **Delete** — `POST /admin/delete-property` → removes a property record.

Each returns a simple JSON success response.

### 2️⃣ B2B Lead Intake
- `POST /client/b2b-request` — a form for contractors/vendors (electricians, builders, etc.) to submit trade requests.
- Saves the request to a `b2b_requests` table in Supabase.
- Sends an instant email alert to the admin via Gmail with all the lead details.

### 3️⃣ 24/7 AI Chatbot (Text)
- `POST /chat` — main chatbot webhook.
- **FAQ layer first**: a fast, free code-based keyword matcher checks the message against a built-in knowledge base (hours, location, policies, pricing, booking process, etc.) and answers instantly if there's a match — no AI call needed.
- **Falls through to an AI Agent** (GPT-4.1-mini via LangChain) when there's no FAQ match, which has:
  - **Buffer memory** (per-session conversation history)
  - A **custom Supabase tool** the agent can call to search live property listings or insert meeting bookings
- Returns a natural-language reply.

### 4️⃣ Voice Agent Integration (ElevenLabs)
- `POST /client/voice-agent` — webhook designed to plug into an ElevenLabs conversational voice agent.
- Runs its own lightweight FAQ router (fast, no AI cost) and falls back to a Supabase property search + formatted spoken-style response when the query isn't an FAQ match.

---

## 🔧 Tech Stack / Integrations

| Purpose                  | Service                          |
|---------------------------|------------------------------------|
| Database                  | Supabase (Postgres)                |
| AI chat / reasoning       | OpenAI (GPT-4.1-mini) via LangChain |
| Chat memory                | LangChain Buffer Window Memory     |
| Email alerts               | Gmail                              |
| Voice agent front-end      | ElevenLabs (webhook consumer)      |
| Trigger layer               | n8n Webhooks                       |

---

## ⚙️ Setup Instructions

1. **Import** `diamond-real-estate-workflow.json` into your n8n instance.
2. **Create credentials** in n8n for:
   - Supabase (service-role key)
   - OpenAI API
   - Gmail OAuth2
3. **Set environment variables** (used by the custom Supabase tool code node):
   - `SUPABASE_URL`
   - `SUPABASE_SERVICE_ROLE_KEY`
4. Wire up each `Supabase*` node to your Supabase credential, and the Gmail node to your Gmail credential.
5. Update the `sendTo` address in **Alert Admin (B2B)1** to your real admin inbox.
6. Ensure your Supabase project has `properties`, `b2b_requests`, and `meetings` tables matching the field names referenced in the nodes.
7. Activate the workflow — each webhook path (`/admin/add-property`, `/chat`, `/client/b2b-request`, `/client/voice-agent`, etc.) becomes a live endpoint you can call from your website or voice platform.

---

## 📁 Files

- `diamond-real-estate-workflow.json` — the sanitized, importable n8n workflow.

---

## 💡 Customization Ideas

- Add authentication/API-key checks to the admin webhooks so only your website's backend can call them.
- Expand the FAQ database or move it into Supabase for non-technical editing.
- Add SMS/WhatsApp notifications alongside the email alerts.
- Extend the Supabase tool to support full CRUD (currently search + insert) for a richer AI agent.

---

## ⚠️ Disclaimer

This workflow handles personal data (names, phone numbers, national ID numbers) for property bookings. Make sure your Supabase setup, data retention, and processing comply with applicable privacy regulations (e.g. GDPR) for your target market.
