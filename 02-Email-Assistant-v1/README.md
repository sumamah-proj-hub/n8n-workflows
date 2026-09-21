# ✉️ AI-Powered Email Summarizer & Auto-Responder Assistant

An automated n8n workflow that monitors incoming emails, generates concise summaries and suggested responses using Google Gemini AI, and forwards the generated response to a designated destination email.

---

### 🎬 Workflow Overview
![Workflow Screenshot](./media/workflow-demo.jpeg)

---

### 🌟 Key Features
* **Automated Email Polling:** Triggers every minute whenever a new email arrives via Gmail.
* **AI Processing:** Uses Google Gemini (via LangChain Agent) to process email body context.
* **Structured Outputs:** Automatically formats response into a 2-line summary and a recommended polite reply.
* **Email Forwarding:** Sends the synthesized output directly to your primary inbox or recipient.

---

### 📝 System Prompts & Logic

**System Message:**
> "You are an email assistant. Read the email and:
> 1. Give a 2-line summary
> 2. Suggest a polite, short reply
> Keep your response structured and clear."

**Input Payload Prompt:**
```text
Summarize this email and suggest a short reply:
Subject: {{ $json.Subject }}
From: {{ $json.From }}
Body: {{ $json.snippet }}