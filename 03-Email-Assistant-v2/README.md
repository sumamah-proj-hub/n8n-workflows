# ✉️ Autonomous AI Email Auto-Responder

An automated n8n workflow that monitors unread Gmail messages, filters self-sent emails to prevent infinite loops, processes message contents using Google Gemini via LangChain, and sends an automated, professional reply back to the sender.

---

### 🎬 Workflow Architecture
![Workflow Screenshot](./media/workflow-demo1.jpeg)
![Workflow Screenshot](./media/workflow-demo2.jpeg)
---

### 🌟 Key Features
* **Real-time Email Monitoring:** Polls Gmail every minute for new `unread` inbox messages.
* **Infinite Loop Safeguard:** Uses both Gmail search query filters (`-from:`) and an explicit **If Node** validation step to ensure self-sent auto-responses do not trigger endless loops.
* **LangChain Integration:** Utilizes Google Gemini via an AI Agent node for context-aware email processing.
* **Automated Reply Dispatch:** Extracts sender address dynamically and dispatches clean message bodies without manual intervention.

---

### ⚙️ How It Works (Step-by-Step)
1. **Gmail Trigger:** Listens for unread emails in `INBOX`.
2. **If Condition:** Validates `$json.from.value[0].address` to verify the sender is not your own email address.
3. **AI Agent Processing:** Evaluates subject, sender, and text body. Passes prompt constraints to output only reply text.
4. **Gmail Dispatch:** Sends generated text to `$('Gmail Trigger').item.json.from.text` with a dynamic `Reply to:` subject line.

---

### 📝 System Prompts

**System Message:**
> "You are an email assistant. Read the incoming email and write a short, polite, professional reply. Only output the reply text — no subject, no extra commentary, just the message body."

---

### 📥 Import & Configuration Guide
1. Import [`workflow.json`](./workflow.json) into n8n.
2. Update `YOUR_OWN_EMAIL@gmail.com` in both the **Gmail Trigger** filter and **If Node** condition.
3. Attach **Gmail OAuth2** credentials to *Gmail Trigger* and *Send a message* nodes.
4. Attach **Google Gemini API Key** to *Google Gemini Chat Model*.
5. Turn workflow **Active**.