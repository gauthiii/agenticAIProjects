# Project 5: Research Agentic AI

## 1. Project Overview

This project builds a **Research & Citation Agent**. Unlike a standard LLM that "hallucinates" facts or relies on outdated training data, this agent actively browses the web and academic archives to write accurate, cited reports.

**Goal:** Generate a professional-grade research report by autonomously searching for real-time data (via Tavily) and scientific papers (via ArXiv), then rigorously fact-checking and formatting the output.

---

## 2. The Agentic Workflow

The workflow implements a **ReAct (Reason + Act)** loop combined with a **Reflection** phase.

### Phase 1: Research & Drafting (The "Researcher")

* **Input:** User Prompt (e.g., "What is the state of Agentic AI?").
* **Tools:**
* `tavily_search_tool`: Searches the live web for blogs, news, and definitions.
* `arxiv_search_tool`: Searches the ArXiv database for academic papers and authors.


* **Loop:** The agent enters a `while` loop (max turns):
1. **Reason:** "I need to find the definition of Agentic AI."
2. **Act:** Calls `tavily_search_tool`.
3. **Observe:** Gets search results.
4. **Reason:** "Now I need scientific backing."
5. **Act:** Calls `arxiv_search_tool`.


* **Output:** A raw text draft with citations.

### Phase 2: Reflection & Rewrite (The "Editor")

* **Action:** The system feeds the raw draft into a fresh LLM instance (Acting as an Academic Reviewer).
* **Task:** It produces a JSON object containing:
* **Reflection:** A critique of the draft's strengths and missing citations.
* **Revised Report:** A rewritten version that fixes the identified issues.



### Phase 3: Formatting (The "Publisher")

* **Action:** The final text is sent to a specialized prompt that converts Markdown/Text into clean, styled **HTML**.
* **Result:** A render-ready HTML document with clickable links and proper headers.

---

## 3. Code Breakdown

### Key Functions

| Function Name | Description |
| --- | --- |
| `generate_research_report_with_tools` | **The Core Loop.** Implements the "Tool Use" logic. It allows the LLM to call `arxiv` or `tavily` multiple times before generating the final answer. |
| `reflection_and_rewrite` | **The Quality Control.** Parses the draft and forces the model to output a strict JSON structure containing both a critique and a fixed version. |
| `convert_report_to_html` | **The Formatter.** A utility function that purely focuses on presentation, ensuring the final output is visually structured. |
| `research_tools.*` | **The Toolkit.** Helper module (imported) that contains the actual API logic for connecting to ArXiv and Tavily. |

### Technical Details

* **OpenAI Native Client:** Unlike previous projects that used `aisuite`, this project uses the native `OpenAI()` client to leverage advanced **Tool Calling** features more granularly.
* **`tool_choice="auto"`:** This setting allows the LLM to decide *if* and *when* it needs to search. If the user asks "Hi", it won't search. If they ask "Latest AI news", it will trigger the tools.
* **JSON Enforcement:** The reflection step uses `json.loads()` to parse the LLM's output. This is a crucial technique for integrating LLMs into programmatic pipelines where you need structured data (like a dictionary) rather than just a string of text.

---

## 4. Why is this "Agentic"?

A standard ChatGPT session relies on its internal memory.

**This system is Agentic because:**

1. **It Investigates:** It doesn't just "know" the answer; it goes out and finds it.
2. **It Cites Evidence:** It can provide URLs and Paper Titles because it actually retrieved them.
3. **It Self-Corrects:** The reflection step ensures that if the first draft was too informal or missed a section, the "Editor" agent fixes it before the user ever sees it.

---

## 5. How to Run

You will need a **Tavily API Key** (for web search) in your `.env` file alongside your OpenAI key.

```python
# 1. Setup
from dotenv import load_dotenv
load_dotenv()

# 2. Define your research topic
topic = "The impact of Agentic AI on Software Engineering"

# 3. Step 1: Research & Draft (The Agent browses the web here)
print("--- 🕵️ Researcher Working ---")
raw_report = generate_research_report_with_tools(topic)

# 4. Step 2: Critique & Fix
print("--- ✍️ Editor Working ---")
improved_data = reflection_and_rewrite(raw_report)
final_text = improved_data['revised_report']

# 5. Step 3: Publish
print("--- 🖨️ Publisher Working ---")
final_html = convert_report_to_html(final_text)

# 6. View Result
from IPython.display import display, HTML
display(HTML(final_html))

```
