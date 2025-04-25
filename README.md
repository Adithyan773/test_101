# **Decentralized Multi-Agent Swarm Chat System (DELVING AI)** 🚀  
---

**A decentralized chat system powered by specialized AI agents collaborating through swarm intelligence!**  
*Explore the collaboration of Assistant, Researcher, and Planner agents in this innovative project.*

---

## **Table of Contents** 📋  
---

- [Abstract](#abstract) ✨
- [Motivation](#motivation) 💡
- [System Architecture](#system-architecture) 🏗️
- [Core Components Deep Dive](#core-components-deep-dive) 🔍
  - [FastAPI Backend (main.py)](#fastapi-backend-mainpy)
  - [Streamlit Frontend (app.py)](#streamlit-frontend-apppy)
  - [Multi-Agent System (swarm_system.py)](#multi-agent-system-swarm_systempy)
  - [Agent Logic (agent.py)](#agent-logic-agentpy)
  - [API Service (api_service.py)](#api-service-api_servicepy)
  - [Tools (tools.py)](#tools-toolspy)
  - [Database (database.py) & Auth (auth.py)](#database-databasepy--auth-authpy)
  - [Configuration (config.py)](#configuration-configpy)
- [Swarm Intelligence Implementation](#swarm-intelligence-implementation) 🐝
  - [Concept Overview](#concept-overview)
  - [The Belief System: Modeling Trust and Reliability](#the-belief-system-modeling-trust-and-reliability)
    - [Parameters](#parameters)
    - [Belief Decay (Mathematical Basis)](#belief-decay-mathematical-basis)
    - [Belief Updates (Success/Failure)](#belief-updates-successfailure)
  - [Probabilistic Communication](#probabilistic-communication)
  - [Intelligent Task Delegation](#intelligent-task-delegation)
- [Agent Workflow Example](#agent-workflow-example) 🛠️
- [File Handling Feature](#file-handling-feature) 📂
- [Advantages Over Standard Chatbots](#advantages-over-standard-chatbots) ✅
- [Running the Project](#running-the-project) 🏃‍♂️
  - [Prerequisites](#prerequisites)
  - [Environment Variables](#environment-variables)
  - [Running the Application](#running-the-application)
- [Conclusion](#conclusion) 🎯
- [Visuals](#visuals) 📸
- [Troubleshooting Image Display Issue](#troubleshooting-image-display-issue) ⚠️

---

## **Abstract** ✨  
---

This project implements a sophisticated chat system powered by a **decentralized Multi-Agent System (MAS)**. Inspired by swarm intelligence principles, it features multiple specialized AI agents (**Assistant**, **Researcher**, **Planner**) that collaborate dynamically to handle user queries. The system incorporates a belief mechanism, allowing agents to assess each other's reliability over time, influencing communication success and task delegation. Users interact via a **Streamlit web interface**, which communicates with a **FastAPI backend**. The system supports tool usage (**Calculator**, **Web Search**, **ArXiv Search**) and allows users to upload PDF documents for context-aware interactions. This architecture enables complex task decomposition, specialized expertise, and potential resilience beyond monolithic chatbot models.

---

## **Motivation** 💡  
---

Traditional chatbots often employ a single, large language model (LLM) to handle all types of user queries. While powerful, this monolithic approach can struggle with:

- **Specialized Tasks:** A generalist model might lack the depth or specific tools needed for tasks like complex calculations, academic research, or structured planning.
- **Scalability:** Integrating diverse new capabilities (tools, knowledge domains) into a single model can be complex and require extensive retraining.
- **Resilience:** A single point of failure can render the entire system unusable.
- **Context Management:** Effectively utilizing vast amounts of context (like large documents) alongside conversational history can be challenging for a single model.

This project explores a Multi-Agent System approach to overcome these limitations. By creating specialized agents with distinct capabilities and tools, and enabling them to collaborate dynamically using swarm intelligence principles, we aim to build a more robust, adaptable, and capable conversational AI system. The belief mechanism adds a layer of dynamic adaptation based on observed agent performance.

---

## **System Architecture** 🏗️  
---

The system comprises several key components interacting asynchronously:

- **Streamlit Frontend (app.py):** User interface for registration, login, chat interaction, and PDF file upload.
- **FastAPI Backend (main.py):** Handles API requests (user auth, chat), manages user sessions, interacts with the database, processes file uploads, and interfaces with the Multi-Agent System.
- **Multi-Agent System (swarm_system.py):** Orchestrates the overall agent swarm. Initializes agents, manages belief parameters, and initiates tasks by routing them to an appropriate starting agent.
- **Agent (agent.py):** Represents individual agents (Assistant, Researcher, Planner). Contains the core logic for task analysis, decision-making (local execution vs. delegation), tool usage, communication (sending/receiving messages), and belief management.
- **LLM Service (api_service.py):** Wraps the interaction with the external LLM API (GitHub Models in this case), handling request formatting and response parsing.
- **Tools (tools.py):** Defines and implements specialized functions (Web Search, Calculator, ArXiv Search) that agents can utilize. Encapsulated in MCPTool class.
- **Database (database.py):** Uses SQLAlchemy and SQLite (or PostgreSQL if configured) to store user credentials and chat history.
- **Authentication (auth.py):** Provides password hashing and verification utilities.
- **Configuration (config.py):** Manages settings using Pydantic, loading from environment variables or a .env file.

Below is a textual representation of the system architecture (originally a Mermaid diagram):

```
User interacts/uploads to Streamlit Frontend
Streamlit Frontend sends HTTP requests (Login/Chat/File) to FastAPI Backend
FastAPI Backend accesses Database for auth/history
FastAPI Backend sends user query/file content to Multi-Agent System
Multi-Agent System initiates task to Agent 1
Agent 1 analyzes/decides with LLM API Service
Agent 1 delegates task probabilistically to Agent 2
Agent 2 analyzes/decides with LLM API Service
Agent 1 and Agent 2 use Tools as needed
Tools access External Services for data
Agents generate responses via LLM API Service
Agent 2 returns result to Agent 1
Agent 1 synthesizes final response to Multi-Agent System
Multi-Agent System returns response to FastAPI Backend
FastAPI Backend stores history in Database
FastAPI Backend sends HTTP response to Streamlit Frontend
Streamlit Frontend displays response to User
```

---

## **Core Components Deep Dive** 🔍  
---

### **FastAPI Backend (main.py)**  
**Role:** Serves as the main API gateway.

**Functionality:**

- Defines API endpoints for `/register`, `/login`, `/chat`.
- Handles user authentication using simple username/password verification against the database (no tokens used post-login, relies on `user_id` in requests).
- Manages database sessions (`get_db` dependency).
- Receives chat requests (`ChatRequest` model), including optional Base64 encoded file content and filename.
- **File Processing:** If a file is included, it decodes the Base64 content, uses `pypdf` to extract text from the PDF bytes, and handles potential errors during this process. Includes request size limiting middleware.
- Fetches recent chat history from the database for context.
- Interfaces with the `MultiAgentSystem` (`mas.initiate_task`), passing the user query, chat context, and extracted file text.
- Stores user messages and final swarm responses in the database.
- Returns the final response to the frontend (`ChatResponse` model).

### **Streamlit Frontend (app.py)**  
**Role:** Provides the interactive user interface.

**Functionality:**

- Uses Streamlit widgets (`st.chat_input`, `st.chat_message`, `st.sidebar`, `st.form`, `st.file_uploader`).
- Manages session state (`st.session_state`) to track login status (`user_id`, `username`), current chat messages, and the uploaded file object.
- Handles user registration and login by calling the respective FastAPI endpoints.
- **File Upload:** Allows users to upload PDF files. Stores the uploaded file object. When a chat message is sent:
  - Reads the file bytes.
  - Encodes bytes to Base64.
  - Includes the Base64 string and filename in the `/chat` request payload.
- Displays the chat history for the current session.
- Sends user prompts (and potentially file data) to the FastAPI `/chat` endpoint.
- Displays the response received from the backend, including error messages and response times.

### **Multi-Agent System (swarm_system.py)**  
**Role:** Manages the collection of agents and high-level swarm parameters.

**Functionality:**

- Initializes the `semantic_kernel` instance and adds the `GitHubModelsChatService`.
- Instantiates the individual `Agent` objects (`assistant`, `researcher`, `planner`) with their specific system prompts, capabilities, tools, and swarm parameters (`gamma`, `phi_min`, `link_efficiency`, `comm_timeout`).
- Holds references to all created agents (`self.agents` dictionary).
- Provides `agents_dict_ref` to each agent for peer awareness.
- Initializes the belief system for all agents (`finalize_agent_setup`).
- `initiate_task`: Selects a starting agent (round-robin for simplicity), and calls its `process_task` method, passing the initial query, user context, and file content/name.

### **Agent Logic (agent.py)**  
**Role:** Defines the behavior and capabilities of an individual agent. This is the core intelligence unit.

**Functionality:**

- **Initialization:** Stores its name, capability, tools, system prompt, kernel instance, swarm parameters, and a reference to the global agent dictionary.
- **Belief Management:**
  - `initialize_beliefs`: Sets initial belief in all agents to 1.0.
  - `decay_beliefs`: Applies time-based exponential decay to beliefs about other agents using the `gamma` parameter.
  - `update_belief`: Adjusts belief about another agent based on interaction outcomes (e.g., successful communication sets belief to 1.0, failures penalize belief), clamped between `phi_min` and 1.0.
- `process_task`: The main execution loop for an agent.
  - Determines if analysis is needed (`_should_analyze_for_delegation`).
  - If analysis is needed, calls `analyze_and_plan_task`.
  - If no delegation is planned (or analysis skipped), calls `execute_local_task`.
  - If delegation is planned:
    - Calls `send_message` to communicate with the target agent.
    - Waits (`asyncio.wait_for`) for the response or timeout.
    - Handles communication success, failure, or timeout (updating beliefs accordingly).
    - Calls `_synthesize_delegation_results` to process the peer's response.
- `analyze_and_plan_task`:
  - Uses the LLM to decide whether to handle the task locally or delegate.
  - Constructs a detailed prompt including the agent's identity, task, context (chat history, file presence), its own tools, peer agents' capabilities/tools/beliefs, and recent delegation history.
  - Instructs the LLM to prioritize file content, then its own tools, then delegation.
  - Requires the LLM to respond in a specific JSON format indicating the decision (reasoning, delegation target/sub-task if any).
  - Parses the LLM response and validates the delegation plan (including loopback prevention using `_validate_delegation_plan`).
- `execute_local_task`:
  - Handles the task directly if analysis determined local execution or as a fallback.
  - Includes simplified heuristics to select a tool if appropriate (e.g., calculator for math).
  - Constructs a prompt for the LLM, including system prompt, chat history, file content (if available), the task description, and potentially the results of a tool execution.
  - Instructs the LLM to prioritize file content if relevant.
  - Calls the LLM via the `GitHubModelsChatService`.
  - Returns the LLM's response or an error message.
- **Communication:**
  - `send_message`: Sends a task delegation message probabilistically based on belief (`current_belief * link_efficiency`). Creates an `asyncio.Task` for the receiving agent's `handle_incoming_message`.
  - `handle_incoming_message`: Processes messages received from other agents, extracts task details (including file info), updates belief about the sender to 1.0, and calls `process_task` for the delegated sub-task.
- `_synthesize_delegation_results`: Uses the LLM to combine the result received from a peer agent with the original task context (including file context summary) to generate a final, coherent answer for the agent's original task.

### **API Service (api_service.py)**  
**Role:** Provides a standardized interface to the LLM backend.

**Functionality:**

- `GitHubModelsChatService` class inherits from Semantic Kernel's `ChatCompletionClientBase`.
- `complete_chat_async`:
  - Takes a `ChatHistory` object and settings.
  - Formats the chat history into the JSON structure required by the GitHub Models API.
  - Sets parameters like `model`, `temperature`, `max_tokens`, `response_format`.
  - Sends the request using the `requests` library, including the API key in the headers.
  - Handles potential HTTP errors, timeouts, JSON decoding errors, and unexpected API responses.
  - Parses the successful response to extract the message content.

### **Tools (tools.py)**  
**Role:** Defines reusable functions that agents can leverage.

**Functionality:**

- `MCPTool` class: A simple wrapper to store the tool's name, description (used in agent prompts), and the callable function. Includes a `to_dict` method for potential future integration with OpenAI-style function calling.
- **Individual tool functions:**
  - `web_search_tool`: Uses the `googlesearch-python` library for web searches and `requests/BeautifulSoup` to scrape and summarize content from top results. Includes basic error handling and content extraction logic.
  - `math_calculator`: Uses Python's `eval()` (with safety precautions) to calculate mathematical expressions. Allows basic arithmetic and specific math functions (`abs`, `min`, `max`, etc.). Includes checks for disallowed characters/patterns.
  - `arxiv_search`: Uses the `arxiv` library to search for scientific papers on the ArXiv repository, formatting the results clearly.
- `AVAILABLE_TOOLS` dictionary: Maps tool names to their `MCPTool` instances, allowing agents to easily look up and use available tools.

### **Database (database.py) & Auth (auth.py)**  
**Role:** Data persistence and basic authentication helpers.

**Functionality:**

- **database.py:**
  - Uses SQLAlchemy ORM to define `User` and `ChatHistory` models.
  - Configures the database connection (SQLite default, configurable via `DATABASE_URL`).
  - Provides functions (`create_database_tables`, `get_db`, `get_user_by_username`, `get_user_by_id`, `add_chat_message`, `get_chat_history`) for interacting with the database.
- **auth.py:**
  - Provides `get_password_hash` and `verify_password` functions using `passlib` for secure password handling.

### **Configuration (config.py)**  
**Role:** Centralized application settings management.

**Functionality:**

- Uses Pydantic's `BaseSettings` to load configuration from environment variables or a `.env` file.
- Defines settings like API keys, model IDs, database URL, swarm parameters (`DEFAULT_GAMMA`, `DEFAULT_PHI_MIN`, etc.), logging levels, and timeouts.

---

## **Swarm Intelligence Implementation** 🐝  
---

### **Concept Overview**  
This project leverages concepts from **Swarm Intelligence (SI)** to enable collaboration between specialized agents. SI studies the collective behavior of decentralized, self-organized systems, often inspired by natural systems like ant colonies or bird flocks. Key principles applied here are:

- **Decentralization:** Control is distributed among agents, rather than residing in a single central authority.
- **Specialization:** Agents have distinct capabilities and tools (Assistant, Researcher, Planner).
- **Indirect Communication/Coordination:** While direct messaging exists, the belief system acts as a form of indirect coordination, guiding future interactions based on past performance.
- **Adaptation:** The belief system allows the swarm to adapt, favoring more reliable agents over time.

### **The Belief System: Modeling Trust and Reliability**  
A crucial element of the swarm implementation is the belief system, where each agent maintains a trust score (**belief, Φ**) for every other agent. This allows the system to dynamically adapt its communication and delegation strategies based on perceived reliability.

#### **Parameters**  
- **Gamma (γ):** The belief decay rate. A higher gamma means beliefs decay faster over time if there's no interaction, signifying that past positive interactions become less relevant more quickly.
- **Phi_min (Φ_min):** The minimum belief value an agent can hold about another. This prevents beliefs from dropping to zero, ensuring there's always a small chance of interaction even with agents that have failed previously. It represents a baseline level of potential capability or a chance for redemption.
- **Link Efficiency (L):** A multiplier affecting the probability of successful communication based on belief. `L >= 1.0` forces success (useful for debugging or specific scenarios), while `L < 1.0` makes communication probabilistic.

#### **Belief Decay (Mathematical Basis)**  
When an agent hasn't interacted with another agent, its belief in that agent decays exponentially over time. The `decay_beliefs` method implements this using the formula:

```
new_belief = Φ_min + (current_belief - Φ_min) * exp(-γ * time_delta)
```

Where:

- `new_belief`: The updated belief value after decay.
- `current_belief`: The agent's belief score before decay.
- `Φ_min`: The minimum possible belief value.
- `γ`: The decay rate parameter.
- `time_delta`: The time elapsed since the last belief update for that specific agent.
- `exp()`: The exponential function (e^x).

This formula ensures that the belief decays towards `Φ_min` but never falls below it. The rate of decay is controlled by `γ`.

#### **Belief Updates (Success/Failure)**  
Beliefs are actively updated based on interaction outcomes:

- **Successful Communication:** When an agent successfully receives a message from another agent (in `handle_incoming_message`), it updates its belief in the sender to 1.0. This reinforces trust after a successful interaction.
- **Communication Failure (Probabilistic):** If a message dispatch fails the probabilistic check in `send_message`, the sending agent penalizes its belief in the target agent (e.g., `new_belief = max(Φ_min, current_belief * 0.75)`).
- **Timeout:** If waiting for a delegated task result times out, the requesting agent harshly penalizes its belief in the target agent (e.g., `new_belief = max(Φ_min, current_belief * 0.5)`).
- **Other Errors:** Exceptions during result processing also lead to belief penalties.

### **Probabilistic Communication**  
The success of sending a message between agents isn’t guaranteed (unless `link_efficiency >= 1.0`). It depends on the sender’s current belief in the receiver and the `link_efficiency` parameter:

```
P(Success) = max(0.0, min(current_belief * link_efficiency, 1.0))
```

A random number is compared against this probability. This simulates potential unreliability or communication channel noise inherent in decentralized systems and encourages the system to favor agents it currently trusts more.

### **Intelligent Task Delegation**  
Instead of simple round-robin or fixed routing, task delegation is decided dynamically by the agent currently holding the task (`analyze_and_plan_task`). The agent uses an LLM to:

- Assess if it can handle the task itself (using file content or its tools).
- If not, evaluate available peers based on their advertised capabilities/tools **AND** the current belief (Φ) in them.
- Select the single best peer deemed most suitable and reliable.
- Formulate a specific sub-task for the chosen peer.
- Avoid immediate loopback (delegating back to the agent that just sent the task).

This LLM-driven analysis, informed by the dynamic belief system, allows for more intelligent and context-aware task routing than simple hardcoded rules.

---

## **Agent Workflow Example** 🛠️  
---

Let’s trace a query like: **"Summarize the key findings about reinforcement learning in the uploaded paper and then search for recent news about AlphaFold."**

1. **User Input:** User uploads `rl_paper.pdf` and enters the query in the Streamlit app.
2. **Frontend (app.py):** Encodes the PDF to Base64, sends the query, user ID, Base64 content, and filename (`rl_paper.pdf`) to the FastAPI `/chat` endpoint.
3. **Backend (main.py):**
   - Receives the request, validates the user ID.
   - Decodes the Base64 content.
   - Uses `pypdf` to extract text from `rl_paper.pdf`. Stores the text.
   - Fetches recent chat history.
   - Calls `mas.initiate_task` with the query, history, extracted text, and filename.
4. **Swarm System (swarm_system.py):** Selects a starting agent, e.g., `assistant`. Calls `assistant.process_task`.
5. **Agent (assistant.process_task):**
   - Sees the task is complex ("summarize", "search news") and involves a file. Determines analysis is needed (`_should_analyze_for_delegation`).
   - Calls `assistant.analyze_and_plan_task`.
6. **Agent (assistant.analyze_and_plan_task):**
   - Prepares a prompt for the LLM including: "My task is complex... I have file content from 'rl_paper.pdf'. Peers are 'researcher' [High Belief, Tools: search, arxiv] and 'planner' [High Belief, Tools: None]. My tools: [calculator]. Should I handle this or delegate?"
   - The LLM responds (in JSON): `{"reasoning": "Task requires planning for multi-step query involving file and search. Delegating to Planner.", "delegations": [{"agent_name": "planner", "task": "User wants a summary of the provided RL paper and recent news on AlphaFold. Plan and delegate execution.", "capability_needed": "planner"}]}`
7. **Agent (assistant.process_task cont.):**
   - Sees a delegation plan for `planner`.
   - Calls `assistant.send_message` targeting `planner`, including the sub-task, original query, history, file content, and filename in the message payload.
8. **Agent (assistant.send_message):** Checks belief in `planner`, calculates `P(Success)`. Assuming success, creates a task for `planner.handle_incoming_message`.
9. **Agent (planner.handle_incoming_message):** Receives message, updates belief in `assistant` to 1.0. Calls `planner.process_task` with the sub-task.
10. **Agent (planner.process_task):**
    - Always analyzes. Calls `planner.analyze_and_plan_task`.
11. **Agent (planner.analyze_and_plan_task):**
    - LLM Prompt: "My task is to plan execution for 'Summarize RL paper, then search AlphaFold news'. File provided. Peers: Assistant [Belief: 1.0, Tools: calc], Researcher [Belief: 0.9, Tools: search, arxiv]. How to delegate?"
    - LLM Response: `{"reasoning": "Breaking down task. Delegating file summary to Assistant (best for direct file use), then news search to Researcher.", "delegations": [{"agent_name": "assistant", "task": "Summarize the key findings about reinforcement learning based *only* on the provided file content from 'rl_paper.pdf'.", "capability_needed": "assistant"}]}`
12. **Agent (planner.process_task cont.):** Delegates the summarization task to `assistant`. Sends message (including file content). Waits for response.
13. **Agent (assistant.handle_incoming_message):** Receives summary task. Calls `assistant.process_task`.
14. **Agent (assistant.process_task):** Analyzes/decides to handle locally using the file. Calls `assistant.execute_local_task`.
15. **Agent (assistant.execute_local_task):** Sends prompt to LLM with file content: "Summarize key RL findings from this text...". Gets summary back.
16. **Agent (assistant.process_task cont.):** Returns the summary result to `planner`.
17. **Agent (planner.process_task cont.):** Receives summary. Now delegates the second part. Delegates "Search for recent news about AlphaFold" to `researcher`. Sends message.
18. **Agent (researcher.handle_incoming_message):** Receives search task. Calls `researcher.process_task`.
19. **Agent (researcher.process_task):** Analyzes/decides to handle locally. Calls `researcher.execute_local_task`.
20. **Agent (researcher.execute_local_task):** Uses `web_search_tool` with query "recent news about AlphaFold". Gets results. Sends prompt to LLM: "Based on these search results, provide the final answer...". Gets synthesized search results.
21. **Agent (researcher.process_task cont.):** Returns search results to `planner`.
22. **Agent (planner.process_task cont.):** Receives search results. Calls `planner._synthesize_delegation_results`.
23. **Agent (planner._synthesize_delegation_results):** Sends prompt to LLM: "Original task was X. Assistant provided summary Y. Researcher provided search results Z. Combine these...". Gets final combined answer.
24. **Agent (planner.process_task cont.):** Returns the final combined answer to `assistant`.
25. **Agent (assistant.process_task cont.):** Receives the final answer from `planner`. Returns final answer.
26. **Swarm System (initiate_task):** Receives the final answer from `assistant`.
27. **Backend (main.py):** Gets the final answer, stores it in the database, and sends it back to the frontend.
28. **Frontend (app.py):** Displays the final answer to the user.

> *(Note: This is a simplified flow. Belief updates and potential communication failures/timeouts would add complexity.)*

![Agent Workflow Diagram](images/agent_workflow.png)  
*Illustration of the agent interaction and task delegation process.*

---

## **File Handling Feature** 📂  
---

A key enhancement is the ability to incorporate user-uploaded PDF documents into the conversation:

- **Upload (Frontend):** `app.py` uses `st.file_uploader` to accept a PDF.
- **Encoding (Frontend):** The file content is read as bytes and encoded into a Base64 string.
- **Transmission (API):** The Base64 string and filename are included in the JSON payload sent to the `/chat` endpoint in `main.py`.
- **Decoding & Parsing (Backend):** `main.py` decodes the Base64 string back into bytes. It then uses the `pypdf` library (`PdfReader`) to read the PDF from these bytes and extract text content page by page. Robust error handling is included for invalid encoding or corrupted/unreadable PDFs.
- **Context Injection (Swarm/Agent):** The extracted text and original filename are passed through the `MultiAgentSystem` to the initial `Agent`.
- **Agent Utilization:**
  - The presence and name of the file are included in the analysis prompt (`analyze_and_plan_task`) to help the LLM decide if the task can be handled locally using the file or if delegation/tools are needed.
  - The full extracted text (potentially truncated for context window limits) is included in the prompt for local execution (`execute_local_task`), allowing the agent’s LLM to directly reference and use the file’s content.
  - File content and name are included in delegation messages (`send_message`, `handle_incoming_message`) so receiving agents also have the necessary context.
  - Synthesis prompts (`_synthesize_delegation_results`) also include a summary of the file context for generating the final answer.

This allows users to ask specific questions about documents, request summaries, or use the document content as context for more complex queries involving other tools.

---

## **Advantages Over Standard Chatbots** ✅  
---

This Multi-Agent Swarm approach offers several advantages compared to traditional monolithic chatbots:

- **Specialization & Expertise:** Agents are designed for specific capabilities (planning, research, assistance) and equipped with relevant tools. A researcher using `arxiv_search` will likely outperform a generalist LLM for finding specific scientific papers.
- **Task Decomposition:** Complex queries requiring multiple steps or types of information (e.g., calculation + web search + file summary) can be broken down by the planner and delegated to the appropriate specialized agents.
- **Enhanced Tool Integration:** Tools are assigned to specific agents that are designed to use them effectively. This modular approach makes it easier to add new tools – simply create a new tool function and potentially a new specialized agent or assign it to an existing relevant one.
- **Adaptability & Resilience (via Beliefs):** The belief system allows the swarm to dynamically adapt. If a particular agent consistently fails or times out, other agents’ belief in it will decrease, making them less likely to delegate tasks to it, routing work towards more reliable peers. While not full fault tolerance, this adds a layer of dynamic optimization.
- **Scalability & Modularity:** The system is inherently modular. Adding new capabilities can involve adding new agents or tools without necessarily requiring modifications to unrelated agents, potentially simplifying development and maintenance compared to retraining a massive monolithic model.
- **Focused Context:** While the entire system handles diverse information (chat history, files, tool results), individual LLM calls within agents can be more focused. For example, `execute_local_task` primarily focuses on the specific sub-task, relevant history, and potentially file/tool results, rather than needing the entire global state in every prompt.

---

## **Running the Project** 🏃‍♂️  
---

### **Prerequisites**  
- **Python 3.9+**
- **pip** (Python package installer)
- **Git** (for cloning, optional)

**Install Dependencies:**

```bash
pip install -r requirements.txt
```

### **Environment Variables**  
First, create a virtual environment and activate it:

```bash
python -m venv venv
source venv/bin/activate  # Linux/macOS
# For Windows: venv\Scripts\activate
```

Then, create a `.env` file with the following content:

```bash
# LLM API Configuration
GITHUB_API_KEY="ghp_YOUR_GITHUB_FINE_GRAINED_PAT_WITH_MODELS_ACCESS" # Replace with your key
DEFAULT_MODEL_ID="gpt-4o-mini" # Or another model compatible with the endpoint

# Database setup, Example for PostgreSQL:
# DATABASE_URL="postgresql://user:password@host:port/database_name"

# Swarm Parameters (Optional - Defaults are in config.py)
# DEFAULT_GAMMA=0.05
# DEFAULT_PHI_MIN=0.1
# DEFAULT_LINK_EFFICIENCY=1.0 # 1.0 forces successful communication
# COMM_TIMEOUT_SECONDS=120

# FastAPI Backend URL (for Streamlit)
FASTAPI_BACKEND_URL="http://127.0.0.1:8000"

# Logging (Optional)
MAX_LOG_LEN=200
```

### **Running the Application**  
**Backend:**

```bash
fastapi run main.py
# Or
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

**Frontend:**

```bash
streamlit run app.py
```

---

## **Conclusion** 🎯  
---

This **Decentralized Multi-Agent Swarm Chat System** demonstrates a powerful alternative to traditional chatbots. By leveraging agent specialization, dynamic task delegation, and a trust-based belief system, it can handle complex, multi-faceted queries involving specialized tools and user-provided documents. While more complex than a monolithic approach, the modularity, adaptability, and potential for specialized expertise offered by this swarm intelligence architecture provide a promising direction for building more capable and robust conversational AI systems.

---

## **Visuals** 📸  
---

### **Agent Workflow Diagram**  
![Agent Workflow Diagram](images/agent_workflow.png)  
*Illustration of the agent interaction and task delegation process.*

### **Streamlit Frontend (Placeholder)**  
![Streamlit Frontend](attachment://streamlit_frontend.png)  
*Screenshot of the Streamlit interface (to be updated with actual image).*

---

## **Troubleshooting Image Display Issue** ⚠️  
---

**ERROR NOTICE:** The image isn’t displaying because the original path (`C:\Users\adith\Documents\Agent_101\images\agent_workflow.png`) is a local file path, which GitHub cannot access. Follow these steps to fix it:

1. **Update Path in README.md:** Replace the local path with a relative path:  
   ```markdown
   ![Agent Workflow Diagram](images/agent_workflow.png)
   ```
   Ensure `images/agent_workflow.png` is in your project root’s `images` folder.

2. **Push to GitHub:**  
   ```bash
   git add .
   git commit -m "Update README with correct image path"
   git push origin main  # Replace 'main' with your branch name if different
   ```

3. **Verify Upload:** Check your GitHub repository to confirm the `images` folder and `agent_workflow.png` are uploaded.

4. **Optional Raw URL:** After pushing, use the raw URL (e.g., `https://raw.githubusercontent.com/Adithyan773/test_101/main/images/agent_workflow.png`) if needed.

💡 *After these steps, the image should display correctly on GitHub!*

---

*Built with ❤️ for innovative AI solutions.*