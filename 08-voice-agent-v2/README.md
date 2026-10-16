# 🗄️ Database Agent (PostgreSQL + Google Gemini)

### 📌 Overview
This is an intelligent n8n database assistant workflow that enables natural language interaction with a PostgreSQL database. It operates in two main phases:
1. **Query & Action Interpretation Pipeline:** Receives incoming chat messages, interprets user intent (such as querying tasks, inserting new items, or updating statuses), and formulates accurate SQL statements using **Google Gemini**.
2. **PostgreSQL Tool Execution & Explanation:** Executes generated SQL queries directly against a **PostgreSQL** database using the `postgresTool`, formats the retrieved data or execution status, and provides clear, human-readable explanations back to the user.

---

### 🎬 Workflow Preview
![Workflow Screenshot](./media/workflow-demo.png)
![Workflow Screenshot](./media/workflow-demo2.png)

---

### 🛠️ Architecture & Node-by-Node Explanation

#### 1. 💬 When chat message received (`@n8n/n8n-nodes-langchain.chatTrigger`)
- **Purpose:** Acts as the entry point, receiving user chat inputs and prompts from the n8n Chat interface.
- **Key Settings:**
  * `Type Version`: `1.4`

#### 2. 🤖 AI Agent (`@n8n/n8n-nodes-langchain.agent`)
- **Purpose:** Coordinates intent recognition, determines whether database queries or updates are needed, interacts with connected tools, and translates raw database output into plain language.
- **Key Settings:**
  * `Prompt Type`: `define`
  * `Text`: `={{ $json.chatInput }}`
  * `Type Version`: `3.1`

#### 3. ⚡ Google Gemini Chat Model (`@n8n/n8n-nodes-langchain.lmChatGoogleGemini`)
- **Purpose:** Provides the primary Large Language Model (LLM) intelligence responsible for parsing user queries and generating valid SQL logic.
- **Credentials:** Google Gemini / PaLM API

#### 4. 🗄️ Execute a SQL query in Postgres (`n8n-nodes-base.postgresTool`)
- **Purpose:** Connected as an AI tool to run dynamic SQL queries directly against the target database (`tasks` table).
- **Key Settings:**
  * `Operation`: `executeQuery`
  * `Description Type`: `manual`
  * `Tool Description`: *"Execute a SQL query against the 'tasks' table (columns: id, title, status, created_at) to answer the user's question or make changes."*
  * `Query Parameter`: `{{ $fromAI('query', 'The full SQL query to execute') }}`
- **Credentials:** PostgreSQL Account

---

### 📝 System Prompts & Logic Setup

#### 1. AI Agent System Prompt:
> *"You are a database assistant. The database has a table called 'tasks' with columns: id, title, status (pending/done), created_at.*
> 
> *When the user asks a question, generate the correct SQL query using the Postgres tool to answer it. For updates/inserts, Execute all queries directly without asking for confirmation.. Always explain the result in plain language."*

#### 2. Database Schema Alignment:
- **Table Structure:** All generated SQL queries rely on the `tasks` table schema containing `id`, `title`, `status` (`pending`/`done`), and `created_at`.
- **Direct Execution:** Write operations (`INSERT`, `UPDATE`, `DELETE`) are set to execute automatically without extra confirmation loops for smooth user interaction.

---

### 📥 Step-by-Step Installation & Setup

1. **Database Preparation:**
   * Ensure you have a running PostgreSQL instance.
   * Run the following SQL query to create the required table structure:
     ```sql
     CREATE TABLE IF NOT EXISTS tasks (
         id SERIAL PRIMARY KEY,
         title VARCHAR(255) NOT NULL,
         status VARCHAR(50) DEFAULT 'pending',
         created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
     );
     ```

2. **Repository Setup:**
   * Download the [`workflow.json`](./workflow.json) file.
   * Open your n8n dashboard -> Click **Import from File** and upload `workflow.json`.

3. **Credentials Configuration:**
   * **Google Gemini Credentials:** Attach your **Google Gemini (PaLM) API Key** to the `Google Gemini Chat Model` node.
   * **PostgreSQL Credentials:** Add and configure your **PostgreSQL Credentials** (Host, Port, Database, User, Password) in the `Execute a SQL query in Postgres` node.

4. **Chat Execution:**
   * Open the n8n chat interface connected to the **When chat message received** trigger.
   * Submit natural language commands such as:
     * *"Show all pending tasks"*
     * *"Add a new task: Complete n8n database integration"*
     * *"Mark task 1 as done"*