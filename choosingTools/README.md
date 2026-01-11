# Project 3: Tool Toggle Agent

## 1. Project Overview

This project demonstrates a **Tool-Use Agent (Router)**. Unlike the previous projects that focused on single-domain tasks (Charts or SQL), this agent acts as a general-purpose assistant that can "route" a user's request to the correct utility.

**Goal:** Create an agent that understands the *intent* of a request (e.g., "What time is it?" vs. "Make me a QR code") and autonomously selects and executes the appropriate Python function from a toolkit.

---

## 2. The Agentic Workflow

The workflow relies on **Function Calling (Tool Use)** capabilities in modern LLMs.

### Phase 1: Tool Definition (The "Toolkit")

1. **Input:** We define standard Python functions (`get_current_time`, `get_weather_from_ip`, etc.).
2. **Action:** We pass these functions directly to the `aisuite` client.
3. **Concept:** The LLM receives not just the prompt, but also a "menu" of available tools and their descriptions.

### Phase 2: Decision Making (The "Router")

1. **Input:** User Prompt (e.g., "Generate me a QR code...").
2. **Reasoning:** The LLM analyzes the prompt against the tool definitions.
* *Prompt:* "Generate QR code..."
* *Match:* Matches `generate_qr_code` function description.


3. **Action:** The LLM decides *which* tool to call and *what arguments* to pass (e.g., extracting the URL "gauthiii.github.io" from the prompt).

### Phase 3: Execution & Response (The "Worker")

1. **Action:** The system executes the chosen Python function.
2. **Output:** The function returns the result (e.g., saves a PNG file or returns a weather string).
3. **Final Response:** The LLM takes the tool's output and formulates a natural language response to the user.

---

## 3. Code Breakdown

### Key Functions

| Function Name | Description |
| --- | --- |
| `get_current_time` | **Utility.** Returns the system time. Simple zero-dependency tool. |
| `get_weather_from_ip` | **API Integration.** Fetches external data using `requests` (IP geolocation + Open-Meteo API). Shows how agents can access real-time internet data. |
| `write_txt_file` | **File I/O.** Allows the agent to persistently save text to the local disk. |
| `generate_qr_code` | **Media Generation.** Uses the `qrcode` library to generate an image file. Shows how agents can create non-text artifacts. |

### Technical Details

* **`aisuite` Integration:** The `client.chat.completions.create` call includes a `tools=[...]` parameter. This automatically handles the complex JSON schema conversion required for OpenAI/Anthropic function calling.
* **`max_turns`:** This parameter prevents infinite loops. If the agent tries to call tools repeatedly without success, it will stop after 5 turns.
* **Type Hinting:** The Python functions use type hints (e.g., `file_path: str`). The LLM uses these hints to understand exactly what kind of data to provide (string vs integer).

---

## 4. Why is this "Agentic"?

A standard script requires you to manually call `get_weather()` if you want the weather.

**This system is Agentic because:**

1. **It Decides:** You don't tell it *which* function to run; you just state your problem. The agent figures out the solution.
2. **It Routes:** It can handle mixed workflows. You could theoretically ask it to "Get the weather and write it to a text file," and it would chain the tools together (calling weather first, then file write).
3. **It Abstracts Complexity:** The user doesn't need to know the API endpoints or library syntax; they just ask in plain English.

---

## 5. How to Run

Ensure you have `requests` and `qrcode` installed (`pip install requests qrcode[pil]`).

```python
# 1. Initialize the client
client = ai.Client()

# 2. Define the tools (as shown in the code block above)
tools_list = [get_current_time, get_weather_from_ip, write_txt_file, generate_qr_code]

# 3. Run a query requiring an external API
response = client.chat.completions.create(
    model="openai:gpt-4o",
    messages=[{"role": "user", "content": "What is the weather right now?"}],
    tools=tools_list
)
display_functions.pretty_print_chat_completion(response)

# 4. Run a query requiring local file generation
response = client.chat.completions.create(
    model="openai:gpt-4o",
    messages=[{"role": "user", "content": "Make a QR code for https://google.com"}],
    tools=tools_list
)
display_functions.pretty_print_chat_completion(response)

```
