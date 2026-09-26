# 📧 Email Assistant & Classifier Agent

### 📌 Overview
An advanced multi-stage n8n AI workflow designed to monitor your Gmail inbox in real time. Powered by **Google Gemini** and **Structured Output Parsers**, this workflow automatically categorizes incoming unread emails, ignores non-essential messages (such as Sales and Promotional content), sends immediate alerts via **Discord** for important emails, drafts context-aware replies, and dispatches auto-responses seamlessly.

---

### 🎬 Workflow Preview
![Workflow video](./media/workflow-demo-vdo.mp4)
![Workflow Screenshot](./media/workflow-demo1.jpeg)
![Workflow Screenshot](./media/workflow-demo2.jpeg)
---

### 🛠️ Architecture & Node Breakdown

#### 1. 📩 Gmail Trigger (`n8n-nodes-base.gmailTrigger`)
* **Purpose:** Polls the Gmail inbox at configured intervals to check for new unread messages.
* **Key Configuration:**
  * `Label`: `INBOX`
  * `Filter Query`: `readStatus:unread`
  * `Mode`: Automated polling / real-time monitoring.

#### 2. 🏷️ Email Classifier Agent (`@n8n/n8n-nodes-langchain.agent`)
* **Purpose:** Evaluates incoming email data (Subject, Sender, Snippet) to determine its category.
* **Connected Components:**
  * **Google Gemini Chat Model:** Handles natural language processing.
  * **Structured Output Parser:** Strictly enforces JSON Schema output.
* **Supported Categories:**
  * `sales`: Cold pitches, sales inquiries, product offers.
  * `promotional`: Newsletters, discount offers, marketing materials.
  * `important`: Legitimate business or personal communications requiring action.
  * `other`: Uncategorized or miscellaneous emails.

#### 3. 🔀 Switch Node (`n8n-nodes-base.switch`)
* **Purpose:** Routes execution based on the parsed classification output (`$json.output.category`).
* **Routing Rules:**
  * `Output 0`: Sales (Skipped)
  * `Output 1`: Promotional (Skipped)
  * `Output 3`: **Important** (Triggers Discord alert & AI reply generation)

#### 4. 🔔 Discord Alert - Update about mail (`n8n-nodes-base.httpRequest`)
* **Purpose:** Sends an immediate notification to a Discord channel when an important email is detected.
* **Payload:** `New important email from {{ $json.From }}: {{ $json.Subject }}`

#### 5. 🤖 AI Reply Agent (`@n8n/n8n-nodes-langchain.agent`)
* **Purpose:** Analyzes email context and drafts a concise, professional reply.
* **Connected Components:**
  * **Google Gemini Chat Model**
  * **Structured Output Parser:** Restricts output to a standardized JSON object:
    ```json
    {
      "subject": "Re: Subject Line",
      "body": "Concise, polite, and professional reply message..."
    }
    ```

#### 6. 📤 Send Email (`n8n-nodes-base.gmail`)
* **Purpose:** Sends the AI-generated reply directly to the original sender.
* **Parameters:**
  * **To:** `={{ $('Gmail Trigger').item.json.From }}`
  * **Subject:** `={{ $json.output.subject }}`
  * **Message Body:** `={{ $json.output.body }}`

#### 7. ✅ Discord Alert - Reply Sent (`n8n-nodes-base.httpRequest`)
* **Purpose:** Posts a confirmation alert to Discord after the auto-reply has been successfully dispatched.

---

### 📝 System Prompts & Logic

#### 1. Classification Prompt:
> *"You are an email classification system. Read the email and classify it into EXACTLY one category: 'sales', 'promotional', 'important', or 'other'. Respond ONLY in JSON format."*

#### 2. AI Reply Prompt:
> *"You are an email assistant. Read the incoming email and write a reply. Respond ONLY in this exact JSON format: `{"subject": "...", "body": "..."}`. Subject should be short and body should contain only the reply text."*

---

### 📥 Step-by-Step Setup Guide

1. **Import Workflow:**
   * Download the [`workflow.json`](./workflow.json) file.
   * In your n8n dashboard, select **Import from File** and upload `workflow.json`.

2. **Configure Credentials:**
   * **Gmail Credentials:** Connect your Gmail OAuth2 account to the `Gmail Trigger` and `Send a message` nodes.
   * **Gemini Credentials:** Attach your **Google Gemini API Key** to both `Google Gemini Chat Model` nodes.

3. **Configure Discord Webhooks:**
   * Create an incoming webhook in your Discord channel settings.
   * Open the `Update about mail` and `reply sent` nodes, then replace `YOUR_DISCORD_WEBHOOK_URL_HERE` with your actual Discord Webhook URL.

4. **Activate Workflow:**
   * Save the workflow and toggle the **Active** switch in the top-right corner.