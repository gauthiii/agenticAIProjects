# Project 1: Interactive Chart Generator

## 1. Project Overview

This project is a **Vision-Reflective Agent**. Unlike standard code generators that blindly output Python scripts, this agent generates a chart, "looks" at the result (using a Vision-Language Model), critiques its own work, and fixes visual errors or data misrepresentations automatically.

**Goal:** Create high-quality, bug-free data visualizations from raw CSV data without manual intervention.

---

## 2. The Agentic Workflow

The workflow follows a **Generation  Execution  Reflection  Refinement** loop.

### Phase 1: Generation (The "Drafter")

1. **Input:** The agent receives the dataset (`coffee_sales.csv`) and a user instruction (e.g., "Compare Q1 sales...").
2. **Action:** The LLM acts as a **Data Visualization Expert**. It writes Python code using `matplotlib` to visualize the data.
3. **Constraint:** The code is wrapped in strict `<execute_python>` tags for safe extraction.

### Phase 2: Execution (The "Environment")

1. **Action:** The system extracts the Python code from the LLM's response.
2. **Runtime:** It executes the code in a local Python environment where the dataframe `df` is pre-loaded.
3. **Output:** A chart image file (`chart_v1.png`) is saved to disk.

### Phase 3: Reflection (The "Critic")

*This is the core "Agentic" step.*

1. **Input:** The agent is fed the **actual image** (`chart_v1.png`) generated in Phase 2, along with the original code.
2. **Action:** A Vision-Language Model (like GPT-4o or Claude 3.5 Sonnet) analyzes the image. It checks for:
* **Data Accuracy:** Did we actually plot Q1 data?
* **Aesthetics:** Are labels overlapping? Is the title clear? Is the color scheme readable?


3. **Output:** It generates a JSON object containing **Feedback** (what went wrong) and **Refined Code** (how to fix it).

### Phase 4: Refinement (The "Fixer")

1. **Action:** The system takes the new, refined code from the reflection phase.
2. **Runtime:** It executes the new code to generate `chart_v2.png`.
3. **Result:** A polished, error-checked chart.

---

## 3. Code Breakdown

### Key Functions

| Function Name | Description |
| --- | --- |
| `generate_chart_code` | **The Drafter.** Takes user instructions and the dataframe schema. Returns a raw string containing the Python code wrapped in XML tags. |
| `reflect_on_image_and_regenerate` | **The Critic.** Takes the generated image path (`chart_v1.png`) and the original code. Uses a Vision model to critique the visual output and returns a JSON object with feedback and fixed code. |
| `run_workflow` | **The Orchestrator.** Connects the dots. It loads data, calls the generator, runs the code `exec()`, calls the reflector, and runs the fixed code. |

### Technical Details

* **Tag-Based Extraction:** The prompts strictly enforce `<execute_python>` tags. This allows the system to use Regex (`re.search`) to reliably separate the "thought process" from the "executable code."
* **Dynamic Execution:** The `exec(initial_code, exec_globals)` function allows the agent to run code generated on the fly. The `exec_globals = {"df": df}` dictionary ensures the agent has access to the data without needing to reload the CSV every time.
* **Vision Integration:** The `utils.encode_image_b64` function converts the chart into a format the LLM can "see," enabling visual debugging.

---

## 4. Why is this "Agentic"?

If this were a simple script, you would run it once and hope the output is correct. If the chart had overlapping labels or the wrong data, you (the human) would have to rewrite the prompt.

**This system is Agentic because:**

1. **It Self-Corrects:** It identifies its own mistakes (e.g., "The legend is unclear") without human input.
2. **It Uses Tools:** It actively uses the Python runtime as a tool to verify its logic.
3. **It Has "Sight":** It doesn't just read code; it interprets the visual result of that code.

---

## 5. How to Run

Ensure you have your `.env` file set up with API keys and the `coffee_sales.csv` in the root directory.

```python
# Define your request
user_instructions = "Create a plot comparing Q1 coffee sales in 2024 and 2025."

# Run the full pipeline
results = run_workflow(
    dataset_path="coffee_sales.csv",
    user_instructions=user_instructions,
    generation_model="gpt-4o-mini", # Fast model for initial draft
    reflection_model="gpt-4o",      # Smart vision model for critique
    image_basename="coffee_analysis"
)

# The final chart will be saved as 'coffee_analysis_v2.png'

```

---
