# Agentic AI Starter Guide (Full Stack)

This guide explains the main tools, concepts, and languages you need to work effectively with agentic AI on Ubuntu. It covers why each piece matters, how to use it, and how to connect it to the outside world.  
At the end of each section there is a short **mental model / intuition** to help you “feel” how that piece fits into an agent workflow.

---

## What agentic AI means

Agentic AI is AI that can:
- Take actions, not just answer questions.
- Use tools (Bash, Python, APIs, search, etc.).
- Plan steps, remember context, and retry when things fail.
- Work like a “virtual assistant” that runs on your machine, in the cloud, or in a sandbox.

Your role is to:
- Design the tasks and constraints.
- Choose which tools are available.
- Set up configs and policies.
- Debug and improve how the agent behaves.

**Mental model**  
Think of an agent as a “smart worker” that can read, plan, and act, but only within the tools and rules *you* give it. You are the architect and the safety engineer.

---

## Why Bash matters

Bash is the command language of Ubuntu and Linux.  
It is how you inspect the system, run programs, and automate steps.

### Why learn Bash
- It is the backbone of most server and agent workflows.
- It lets you chain tools together (grep, sed, awk, find, etc.).
- It integrates directly with AI agents (“run this command”).
- It is fast and scriptable for repetition.

### Key things to know
- Basic navigation: `ls`, `cd`, `pwd`, `cat`, `less`.
- File operations: `cp`, `mv`, `rm`, `mkdir`, `chmod`, `./script.sh`.
- Text tools: `grep`, `find`, `sed`, `awk`.
- Shell features: variables, exit codes, `if`, `for`, `while`, functions, and argument handling.
- Scripting: small, reusable scripts that can be called by an agent.

### Use cases
- Running setup scripts for agents or tools.
- Checking logs, configs, and directories.
- Starting services or containers.
- Testing agent outputs by running commands manually.
- Debugging failures by inspecting files and processes.

**Mental model**  
Bash is the “muscle” of the agent. When the agent wants to *do something* on the machine, it will usually call a Bash command or script. You design safe, composable commands that the agent can reuse.

---

## Why Nano matters

Nano is a simple terminal text editor.  
It is used to quickly open and edit config, scripts, and prompt files without leaving the terminal.

### Why learn Nano
- It is easy and beginner‑friendly.
- It has no complex modes.
- It is great for SSH, minimal terminals, and cloud environments.
- It works very well with YAML/JSON/Markdown.

### Key shortcuts
- `nano file` → open.
- `Ctrl + O` → save.
- `Ctrl + X` → exit.
- `Ctrl + W` → search.
- `Ctrl + \` → find and replace.
- `Ctrl + K` → cut line; `Ctrl + U` → paste.

### Use cases
- Editing YAML/JSON config files (e.g., NemoClaw/OpenClaw, API keys, policies).
- Updating Bash scripts quickly.
- Tweaking prompts or rules on the fly.
- Fixing mistakes during debugging without switching to a GUI editor.

**Mental model**  
Nano is the “quick‑edit” channel. When the agent hits a config problem, you jump into Nano, make a small change, and resume. It is low‑friction and safe because you stay in the same terminal environment.

---

## Why Nano and not Vim (yet)

Nano is usually chosen first because it is simpler and safer for beginners.  
Vim is powerful but requires learning modes and a lot of commands.

### When to use Nano
- Quick edits during debugging.
- Occasional file changes.
- Learning phase and experimentation.
- Environments where you just want to “fix and move on”.

### When Vim may be better later
- Heavy code or config editing.
- Advanced navigation and macros.
- Long‑term projects where you want speed and power.

You can start with Nano and move to Vim later only if you are willing to invest time.

**Mental model**  
Nano is the “everyone” editor; Vim is the “power user” editor. Start with Nano so you can focus on the *agent logic*, not the editor learning curve.

---

## YAML, JSON, JSON5, and Markdown

These are the main formats you will see in agentic systems.

### YAML

Great for human‑edited config and policy files.

- Easy to read, uses indentation and key‑value trees.
- Supports comments and nested structures.
- Common in agent configs, manifests, and policy files.

Example:
```yaml
agent:
  name: openclaw
  enabled: true
  tools:
    - bash
    - nano
```

Use cases:
- Policies and sandbox rules (e.g., NemoClaw/OpenClaw).
- App settings and workflows.
- Deployment‑style manifests.

**Mental model**  
YAML is the “designer config” format. You, as the operator, design how the agent is allowed to behave using YAML, and the agent reads those rules at runtime.

---

### JSON

Strict structured data for machines.

- Uses braces, brackets, and quoted strings.
- Very common in APIs and JSON‑based tools.
- No comments allowed in standard JSON.

Example:
```json
{
  "agent": {
    "name": "openclaw",
    "enabled": true,
    "tools": ["bash", "nano"]
  }
}
```

Use cases:
- API requests and responses.
- Database‑like records or tool outputs.
- Config files that must be parsed reliably by many tools.

**Mental model**  
JSON is the “machine‑ready” format. It carries structured data between the agent, APIs, and other tools in a way that is easy to parse and validate.

---

### JSON5

JSON with more “human‑friendly” features.

- Allows comments.
- Often allows unquoted keys and trailing commas.
- Makes it easier to edit config by hand.

Example:
```json5
{
  agent: {
    name: "openclaw",
    enabled: true, // comment
    tools: ["bash", "nano",],
  },
}
```

Use cases:
- Developer config files that are edited often.
- Where you want JSON structure but more comfort.

**Mental model**  
JSON5 is “JSON that developers like to edit by hand.” It is useful when you, the human operator, will tweak the file frequently but still want most tools to treat it like JSON.

---

### Markdown

For writing human‑readable text.

- Headings, lists, code blocks, tables, emphasis.
- Great for docs, notes, and instructions.

Example:
```markdown
# Agent Setup

- Use Bash for scripts.
- Use Nano for edits.
- Store configs in YAML.
```

Use cases:
- READMEs, docs, and internal wikis.
- Agent prompts, instructions, and runbooks.
- Task notes and operational summaries.

**Mental model**  
Markdown is the “storyteller” format. It explains what the agent does, why it does it, and how to run it. It connects the technical stack to the human team.

---

## Why these formats matter together

An agentic workflow often looks like this:
- Bash runs commands and scripts.
- Nano edits YAML/JSON/JSON5/Markdown files.
- YAML/JSON store configs, policies, and state.
- Markdown stores instructions and output summaries.

At the end of the day, the agent can:
- Read YAML/JSON configs to decide what to do.
- Call Bash to act on the system.
- Write results as JSON logs.
- Generate Markdown reports for you.

**Mental model**  
Think of the formats as a “language stack”:
- YAML for your *intent*.
- JSON for *machine communication*.
- JSON5 for *developer comfort*.
- Markdown for *human communication*.

---

## Python: your core agent glue

Python is the language that connects tools, APIs, and logic in most agentic systems.

### Why Python
- Easy to read and write.
- Huge ecosystem for AI, automation, APIs, and data.
- Works well with JSON/YAML and Bash.
- Great for building small agents, tools, and helpers.

### What to learn in Python

#### 1. Basics
- Variables, types, operators.
- Lists, dicts, tuples, strings.
- Loops (`for`, `while`), conditionals (`if/elif/else`).
- Functions and modules.
- Error handling with `try/except`.

#### 2. Files and data
- Read/write files (`open`, `with`).
- Work with JSON and YAML (using `json` and `PyYAML` or similar).
- Handle exceptions and edge cases.

#### 3. Data classes
Use `dataclasses` to represent structured data cleanly.

Example:
```python
from dataclasses import dataclass

@dataclass
class AgentTask:
    name: str
    priority: int
    done: bool = False
```

Data classes are great for:
- Tasks and steps.
- Tool results.
- Messages between components.
- Config‑like objects.

#### 4. APIs and HTTP
- Learn `requests` (or `httpx`).
- Make HTTP requests.
- Handle JSON responses.
- Add authentication (API keys, headers, tokens).

Example:
```python
import requests

resp = requests.get("https://api.example.com/tasks")
data = resp.json()
```

Use cases:
- Talking to AI models.
- Reading external services (weather, search, etc.).
- Sending results back to a dashboard or database.

#### 5. Async and concurrency (basic)
- `async`/`await` and `asyncio`.
- Run multiple tasks at once (e.g., several API calls).
- Not needed for everything, but useful for performance.

You do not need to master async, but you should be comfortable reading and using async‑style code.

**Mental model**  
Python is the “glue” between the agent and everything else. It turns configs, Bash commands, and API calls into a single coherent workflow.

---

## How Python fits agentic AI

A typical agent might:
- Load a YAML config that defines what tools it can use.
- Use Python data classes to model tasks and messages.
- Call Bash scripts or tools.
- Use Python to call APIs (including search).
- Save results as JSON.
- Write a Markdown summary for you.

Python is the orchestrator that ties your local stack (Bash, Nano) to the outside world (APIs, search, databases).

**Mental model**  
Python is the “conductor” of the agent orchestra: it reads the score (config), calls the instruments (Bash, APIs), and outputs the results (JSON, Markdown).

---

## Tools agentic AI can use (connecting to the outside world)

So far the stack is mostly local. Now you also need to understand how the agent can **interact with the outside world**:

- Search the internet.
- Call external APIs.
- Access databases or knowledge stores.
- Perform actions (send messages, create tickets, run code).

These are exposed as **tools** or **functions** that the agent is allowed to call.

### 1. Web search tools

These let the agent pull current info from the internet.

- Search APIs like **Tavily**, **SerpAPI**, **Perplexity Search API**, **Google Programmable Search**, **Bing Search**, etc. [web:59][web:62][web:65]  
- The agent can:
  - Ask “search the web for X”.
  - Get a short, focused result (not raw HTML).
  - Decide next steps based on what it finds.

Example simplified tool call:
```python
web_search(query="latest NVIDIA NemoClaw security features")
```

Use cases:
- Getting current news or prices.
- Researching competitors, specs, or documentation.
- Checking if a library or API has been updated.

**Mental model**  
Web search tools are the agent’s “fast internet lookup” capability. You allow it to search only when you want up‑to‑date information, and you design safety rules so it cannot wander into risky sites.

---

### 2. Web scraping and browser automation

When the agent needs to read pages that a simple search cannot reach.

- Tools like **web_scraper**, **browser_use**, or browser‑automation tools (e.g., Playwright / Puppeteer‑based) [web:65][web:66].  
- These can:
  - Open a URL.
  - Click, wait, fill forms.
  - Extract text or structured data.

Example:
```python
scrape_page(url="https://example.com/pricing")
```

Use cases:
- Extracting prices from a product page.
- Reading tables from a report.
- Logging into a service (if allowed and secure).

**Mental model**  
Web‑scraping tools are the agent’s “eyes on the web.” They let it read specific pages, but you must configure them carefully to avoid breaking robots.txt, rate limits, or security rules.

---

### 3. General APIs

Most real‑world apps expose APIs that agents can call.

- REST APIs for:
  - Weather, finance, travel, CRM, messaging, cloud storage, databases, etc. [web:60][web:64][web:66].  
- In Python, the agent or your helper code usually uses `requests` or similar.

Example pattern:
```python
import requests

resp = requests.get("https://api.example.com/tasks")
tasks = resp.json()
```

Use cases:
- Reading tasks from a project‑management tool.
- Sending notifications (Slack, email, SMS).
- Syncing data between systems.

**Mental model**  
APIs are the agent’s “plugs into existing systems.” Each API is a well‑defined interface you can safely expose as a tool so the agent can talk to the rest of the world.

---

### 4. Databases and knowledge tools

Agents often pull data from data stores or retrieval‑augmented systems.

- SQL databases, vector databases, knowledge bases, internal wikis, CSV/JSON feeds. [web:64][web:67]  
- An agent might:
  - Search a knowledge base.
  - Query a SQL table.
  - Summarize data from a CSV or JSON feed.

Use cases:
- Internal documentation search.
- Customer‑support answers.
- Data‑driven reports.

**Mental model**  
Databases and RAG‑style tools are the agent’s “memory externalized.” Instead of relying only on its training data, it can read current, structured knowledge from your own systems.

---

### 5. “Action” tools

Beyond search and data, agents can perform actions.

- Send emails or Slack messages.
- Create tickets in Jira, GitHub issues, or similar.
- Trigger deployments.
- Run code in a sandbox (Python executor, etc.). [web:65][web:66]  

Example:
```python
send_slack_message(channel="#alerts", text="Agent completed task.")
```

Use cases:
- Notifications and status updates.
- Workflow automation.
- Integrating with existing tools (monitoring, ticketing, CI/CD).

**Mental model**  
Action tools are the agent’s “hands.” They let it change the world (send messages, create records, deploy code) instead of only reading or analyzing data.

---

### Tool‑calling and safety

- Tool calling means the agent **decides which function to call** and what arguments to pass, within a set of functions you define. [web:63][web:66]  
- You can:
  - Define which tools are available.
  - Set which tools are allowed in each environment (e.g., no search in production, no direct web access in sandbox).
  - Log which tools were called and why.

**Mental model**  
Tool calling is the “contract” between the agent and the outside world. You design the vocabulary of tools; the agent chooses words from that vocabulary to solve tasks.

---

## Agent concepts you need to know

Now that you know the stack and the tools, you also need to understand how agents think and work.

### 1. Planning and tool use
- The agent takes a goal and breaks it into steps.
- Each step can be:
  - A Bash command.
  - A Python function.
  - A web search.
  - A database query.
- The agent remembers which step it is on and why it took previous steps.

**Mental model**  
Planning is the agent’s “internal task list.” You can influence it by giving clear instructions, but the agent decides how to break things down and which tools to use.

### 2. Memory and context
- The agent stores:
  - Conversation history.
  - Past actions and tool calls.
  - Task state and intermediate results.
- This is usually stored in JSON/YAML, a database, or a vector store.

**Mental model**  
Memory is the agent’s “notebook.” It remembers what it did so it does not repeat mistakes and can resume work after interruptions.

### 3. Retrieval and knowledge
- Retrieval‑augmented generation (RAG) lets the agent search internal documents, wikis, or notes and answer from them. [web:64][web:67]  
- You can store docs and notes in Markdown or structured formats.

**Mental model**  
Retrieval is the agent’s “library.” Instead of guessing from vague training data, it can read your own documents and base answers on your internal knowledge.

### 4. Guardrails and safety
- You limit what the agent can do:
  - No sensitive commands.
  - No certain APIs or sites.
  - No dangerous file writes.
- Policies are stored in YAML‑style rules and enforced at runtime. [web:52][web:64]

**Mental model**  
Guardrails are the “traffic rules” for the agent. You let it operate in the safe lanes of your system, not on the open road.

### 5. Evaluation and debugging
- You need to:
  - Check if the agent completed the task correctly.
  - Read logs of what tools it called.
  - Verify outputs and side effects.
- You can use JSON logs and Markdown reports to help.

**Mental model**  
Evaluation is your “quality‑control checklist.” You review what the agent did, not just what it said, and you refine its tools and rules over time.

---

## What you need to do next

To work effectively with agentic systems, focus on:

1. **Bash & Nano fluency**
   - Edit YAML/JSON/Markdown files.
   - Write small Bash scripts.
   - Practice debugging from the terminal.

2. **Python layer**
   - Learn Python basics and data classes.
   - Learn reading/writing JSON/YAML.
   - Learn how to call APIs and web‑search‑style tools.
   - Learn basic async if you plan o run multiple tools or API calls at the same time, so your agent can be faster and more efficient.