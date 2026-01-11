# Project 6: Startup Builder (Multi-Agent Chain)

## 1. Project Overview

This project demonstrates a **Vertical Multi-Agent Workflow**. Unlike previous projects that focused on single tasks (coding or searching), this system simulates an entire **Startup Founding Team**. It takes a raw, one-sentence idea and passes it through a chain of 6 specialized agents, progressively refining it into a comprehensive Business Requirement Document (BRD).

**Goal:** Transform a vague idea (e.g., "Uber for dog walking") into a professional PDF report containing market analysis, competitor SWOT, MVP features, and scope definition.

---

## 2. The Agentic Workflow

The architecture follows a **Sequential Context Chain**. Each agent performs a specific role and passes its structured output (JSON) to the next agent in the line.

### The "Virtual Team" Roster:

1. **The Visionary (Agent 1):**
* **Role:** Analyzes the raw topic.
* **Output:** Defines the Problem Statement, Market Size, and Target Audience.


2. **The Market Analyst (Agent 2):**
* **Role:** Takes the Visionary's output and analyzes the landscape.
* **Output:** Identifies Direct/Indirect competitors and performs a Feature SWOT analysis.


3. **The Strategist (Agent 3):**
* **Role:** Synthesizes the market data to find the "Blue Ocean."
* **Output:** Identifies Market Gaps and defines the Product Vision.


4. **The Product Manager (Agent 4):**
* **Role:** Defines boundaries to prevent scope creep.
* **Output:** Lists "In-Scope" vs. "Out-of-Scope" features.


5. **The Tech Lead (Agent 5):**
* **Role:** Translates the scope into actionable requirements.
* **Output:** A list of 10 core MVP (Minimum Viable Product) features.


6. **The Publisher (Agent 6):**
* **Role:** The Compiler.
* **Output:** Takes **all** previous JSON outputs and generates a styled HTML report.



---

## 3. Code Breakdown

### Key Functions

| Function Name | Agent Role | Logic |
| --- | --- | --- |
| `idea_agent` | **Visionary** | Uses `gpt-4o` to expand a short string into a structured JSON business case. |
| `market_research_agent` | **Analyst** | Receives the idea JSON. Uses internal knowledge (or potentially tools) to compare competitors. |
| `gap_vision` | **Strategist** | Compares the Idea vs. Competitors to find the unique value proposition. |
| `scope_analysis` | **Manager** | **Model Switch:** Uses `claude-haiku-4-5`. Defines what *not* to build (crucial for startups). |
| `mvp_research` | **Tech Lead** | **Model Switch:** Uses `claude-haiku-4-5`. Generates the specific feature list. |
| `html_coder` | **Publisher** | Receives the accumulated context from all 5 previous agents and writes code. |

### Technical Details

* **Structured JSON Parsing:** The system relies on "Prompt Engineering for JSON". It strips Markdown code blocks (`re.sub`) and parses the string into a Python dictionary (`json.loads`). This ensures the agents speak a machine-readable language between steps.
* **Model Routing:** The project demonstrates cost/performance optimization by switching models. It uses **GPT-4o** for complex reasoning (Market/Vision) and **Claude Haiku** for structured listing tasks (Scope/MVP).
* **Context Accumulation:** Agent 6 does not just see the previous step; it sees the entire history of the pipeline to generate the final report.
* **WeasyPrint:** The final HTML is programmatically converted to a PDF, creating a tangible artifact for the user.

---

## 4. Why is this "Agentic"?

A standard ChatGPT prompt "Write a business plan for X" gives a generic, hallucinated wall of text.

**This system is Agentic because:**

1. **Decomposition:** It breaks a massive task (Business Planning) into 6 manageable sub-tasks.
2. **Structured Thinking:** It forces the AI to think in steps. It cannot define the MVP (Agent 5) until it has defined the Market Gap (Agent 3).
3. **Role-Playing:** Each prompt assigns a specific persona ("Meticulous research agent"), which drastically changes the tone and quality of the output.

---

## 5. How to Run

Ensure you have `weasyprint` installed (`pip install weasyprint`) and your `.env` set up.

```python
# 1. Define your idea
topic = "An app that helps people find domestic helpers in their area"

# 2. Run the Chain (Agents 1-5)
print("--- 1. Crystallizing Idea ---")
idea_json = idea_agent(topic)

print("--- 2. Analyzing Market ---")
market_json = market_research_agent(idea_json)

print("--- 3. Finding Gaps ---")
gap_json = gap_vision(idea_json, market_json)

print("--- 4. Defining Scope ---")
scope_json = scope_analysis(idea_json, gap_json)

print("--- 5. Listing MVP Features ---")
mvp_json = mvp_research(idea_json, scope_json)

# 3. Generate Report (Agent 6)
print("--- 6. Generating Report ---")
html_content = html_coder(
    idea_json, 
    market_json, 
    gap_json, 
    scope_json, 
    mvp_json
)

# 4. Save to PDF
from weasyprint import HTML
HTML(string=html_content).write_pdf("startup_plan.pdf")
print("✅ Business Plan saved to startup_plan.pdf")

```
