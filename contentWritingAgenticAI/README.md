# Project 4: Content Writing Agentic AI

## 1. Project Overview

This project implements a **Reflection-Based Writing Agent**. Writing high-quality content rarely happens in a single pass; it requires drafting, reviewing, and editing. This system mimics that human process by using three distinct "agents" (roles) to iteratively improve the output.

**Goal:** Produce high-quality, well-structured essays by separating the **creation** phase from the **critique** phase.

---

## 2. The Agentic Workflow

The workflow is a linear pipeline designed to simulate a professional editorial team.

### Phase 1: Drafting (The "Writer")

* **Role:** Meticulous Research Agent.
* **Action:** Takes a high-level topic (e.g., "Is AI the new electricity?") and generates a comprehensive first draft.
* **Structure:** It is strictly instructed to include an Introduction, Main Content, Advantages, Disadvantages, and Conclusion.

### Phase 2: Reflection (The "Critic")

* **Role:** Critical Reviewer.
* **Action:** It reads the draft *without* trying to rewrite it. Its sole job is to identify weaknesses.
* **Criteria:** It analyzes Structure, Clarity, Strength of Argument, and Writing Style.
* **Output:** It produces a "Feedback" document, not a new essay.

### Phase 3: Revision (The "Editor")

* **Role:** Expert Editor.
* **Action:** It takes two inputs: the **Original Draft** and the **Feedback**.
* **Logic:** It synthesizes the creativity of the Writer with the logic of the Critic to rewrite the essay.
* **Result:** A polished final piece that addresses gaps identified in Phase 2.

---

## 3. Code Breakdown

### Key Functions

| Function Name | Description | Role |
| --- | --- | --- |
| `generate_draft` | **Creation.** Generates the initial text based on the user prompt. Uses a high-creativity model (e.g., GPT-4o). | **Writer** |
| `reflect_on_draft` | **Analysis.** Critiques the draft. It uses a reasoning-heavy model (e.g., o4-mini) to spot logical inconsistencies or structural issues. | **Reviewer** |
| `revise_draft` | **Synthesis.** Rewrites the content by applying the specific feedback points to the original text. | **Editor** |

### Technical Details

* **Model Selection:** You'll notice the code allows different models for different steps.
* *Drafting:* Uses standard GPT-4o for fluency.
* *Reflection:* Uses `o4-mini` (likely a reasoning model), which is often better at critical thinking and spotting logic errors than pure text generation models.


* **Prompt Engineering:** The prompts are "Role-Based." Each function starts by defining who the agent is ("You are a meticulous research agent", "You are going to provide constructive criticism"). This "Persona Pattern" helps anchor the LLM's behavior.

---

## 4. Why is this "Agentic"?

If you ask ChatGPT, "Write me an essay," you get a decent result. But if you ask it to "Write, then critique yourself, then rewrite," the quality jumps significantly.

**This system is Agentic because:**

1. **Separation of Concerns:** It doesn't try to be creative and critical at the same time. It separates these conflicting goals into different steps.
2. **Iterative Improvement:** It acknowledges that the first output is rarely the best.
3. **Context Passing:** The output of Agent 1 becomes the input for Agent 2, creating a chain of thought that is deeper than a single prompt could achieve.

---

## 5. How to Run

Ensure `aisuite` is installed and your API keys are in `.env`.

```python
# 1. Define your topic
topic = "Is AI the new electricity that is gonna change all industries?"

# 2. Step 1: Generate
print("--- Generating Draft ---")
draft = generate_draft(topic)
print(draft[:200] + "...") # Preview

# 3. Step 2: Critique
print("\n--- Reflecting ---")
feedback = reflect_on_draft(draft)
print(feedback)

# 4. Step 3: Refine
print("\n--- Revising ---")
final_essay = revise_draft(original_draft=draft, reflection=feedback)
print(final_essay)

```
