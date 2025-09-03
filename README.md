# Messenger-AI-Sales-Assistant-n8n-OpenAI-Google-Sheets
This workflow turns your Facebook Page inbox into a smart sales assistant for sewing machines. It reads customer messages on Messenger, looks up product data in Google Sheets, crafts a helpful reply with OpenAI, and sends it back—fast. It can also route shipping quotes to WhatsApp, block test senders, and keep your prompts consistent.

---

<img width="1394" height="473" alt="Screenshot 2025-09-03 at 4 44 59 p m" src="https://github.com/user-attachments/assets/841cd727-4755-4588-81a1-17296261af9b" />

---

What You Can Do
Auto-reply on Messenger with product info, availability, and tailored recommendations powered by your Google Sheet.
Escalate shipping quotes: for any “price with shipping” ask, reply with the owner’s WhatsApp (no price over chat).
Respect FB’s message limits: replies are safely trimmed to avoid Graph API errors (630 chars).
Prevent noise: block a specific sender.id (handy during testing).
Stay consistent: unified agent prompt + a Google Sheets tool description keep answers grounded in your sheet (no hallucinations).
Optional add-ons you can bolt on in minutes:
Conversation logging to a Sheet (append each Q/A).
Feedback capture (👍/👎) for future prompt/model tuning.
Auto-create Sheet if missing (add a “create spreadsheet” branch before first read).

Use Import Workflow and select the JSON you exported. Confirm the nodes appear as: Webhook → If/Respond (verify) → If (message checks) → AI Agent → Set (Edit Fields) → HTTP Request (Graph).
Set Environment Variables (n8n → Settings → Environment Variables)

FB_VERIFY_TOKEN=xxxxxxxx

FB_BLOCKED_SENDER_ID=xxxxxxxx

SALES_CONTACT=xxxxxxxx

GSHEET_ID=xxxxxxxx

GSHEET_SHEET_NAME=xxxxxxxx

OPENAI_API_KEY=xxxxxxxx

These are referenced throughout the workflow (verify step, Messenger send, Google Sheets tool, and the shipping-quote handoff).

---

Connect Credentials
OpenAI: select the credential used by the OpenAI Chat Model nodes (gpt-4.1-mini primary, gpt-3.5-turbo fallback).
Google Sheets (Service Account): upload your JSON key, then share the sheet with that service account email as Editor. Point the tool to GSHEET_ID and GSHEET_SHEET_NAME.
Facebook Graph: pick your Page token credential in the HTTP Request node (POST /me/messages).
Hook up Meta Webhooks
Callback URL = your n8n webhook URL (the workflow’s path is already set).
Verify Token = FB_VERIFY_TOKEN.
The workflow’s verify branch echoes hub.challenge on subscribe.

---

Activate & Test
Message your Page from a different FB account. Check n8n Executions:
If1 ensures the message has text and the sender.id isn’t blocked.
AI Agent queries Google Sheets via the tool and composes the reply.
HTTP Request sends back to Messenger (safe-trim to 630).
How It Works (Under the Hood)
Webhook + Verify: Handles Meta’s hub.challenge with FB_VERIFY_TOKEN guard.
Message Gate: Only proceed if there’s text and sender.id ≠ FB_BLOCKED_SENDER_ID.
AI Agent:
Primary model: gpt-4.1-mini, fallback: gpt-3.5-turbo.
Memory keyed by sender.id for short context.
Google Sheets Tool (Service Account) to fetch model/price/stock from your sheet.
Strict prompt: do not invent, recommend when asked, and for shipping quotes reply with SALES_CONTACT.
Message Build & Send: The Set node composes messaging_type, recipient.id, and the message.text from the agent’s output; HTTP node posts to Graph v23.0 with a 630-char slice.
Recommended Sheet Columns
Modelo | Descripción | Precio | Existencia | Notas
Keep names stable; the agent depends on consistent headers for reliable lookups.

---
Troubleshooting
recipient required → Ensure the Set node maps recipient.id from the incoming Webhook payload.
403/404 from Sheets → Share the doc with the service account email and confirm GSHEET_ID / GSHEET_SHEET_NAME.
Verify fails → Double-check webhook path and FB_VERIFY_TOKEN.
Long replies → Already trimmed to 630 to satisfy Messenger API. Increase only if you switch to templates.
