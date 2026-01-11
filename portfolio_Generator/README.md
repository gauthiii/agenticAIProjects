# Project 7: Portfolio Generator Agent

## 1. Project Overview

This project implements a **Resume-to-Website Pipeline**. It uses a multi-agent system to parse a standard PDF resume, extract the unstructured data into a structured format, and then iteratively build a responsive, dark-mode-enabled portfolio website using Tailwind CSS.

**Goal:** Automate the conversion of a static document (Resume PDF) into a dynamic digital asset (Personal Website) without manual coding.

---

## 2. The Agentic Workflow

The system operates like a **Software Development Assembly Line**. Instead of asking an LLM to "write the whole website at once" (which often leads to cutoff text or hallucinations), this project breaks the site construction into modular steps.

### The Assembly Line:

1. **The Parser (Agent 1):**
* **Input:** Raw text extracted from `resume.pdf` via `PyPDF2`.
* **Role:** Data Engineer.
* **Action:** Cleans the messy text and organizes it into a strict JSON dictionary containing keys like `educationDetails`, `workExperience`, and `skills`.


2. **The Architect (Agent 2):**
* **Input:** The structured JSON.
* **Role:** Frontend Tech Lead.
* **Action:** Generates the **HTML Skeleton**. It sets up the `<head>`, Tailwind CDN, FontAwesome, Navigation Bar, and empty sections (`<section id="about">`). It handles the "plumbing" (dark mode logic, smooth scrolling).


3. **The Content Injectors (Agents 3 & 4...):**
* **Input:** The current HTML Skeleton + The specific JSON data point.
* **Role:** Full Stack Developer.
* **Action:** These agents do not just output HTML. They output **Python code** wrapped in `<execute_python>` tags. This Python code uses string manipulation to *inject* the specific content (e.g., the About section or Skills cards) into the correct placeholder in the HTML skeleton.



---

## 3. Code Breakdown

### Key Functions

| Function Name | Agent Role | Description |
| --- | --- | --- |
| `parse_basic_details` | **Data Parser** | Uses `gpt-4o` to extract entities (Name, Email, Skills) from the raw PDF text and return a sanitized JSON object. |
| `web_skeleton_generator` | **Architect** | Uses `claude-haiku` to build the core HTML structure. It defines the layout but leaves the content sections empty. |
| `about_generator` | **Integrator** | targeted specifically at the "About" section. It writes a Python script to locate the "About" section in the HTML and insert the bio text. |
| `skills_generator` | **Integrator** | targeted at the "Skills" section. It formats the skills list into responsive Tailwind cards and injects them. |
| `save_fenced_html` | **Utility** | A helper function to strip Markdown fences and save the actual HTML code to a file. |

### Technical Highlights

* **Metaprogramming (Code Writing Code):** Agents 3 and 4 are unique because they don't just return text; they return an *executable Python script* that modifies the `skeleton` variable. This allows for precise programmatic editing of the HTML file.
* **Model Routing:** The project uses **GPT-4o** for the complex task of understanding a messy PDF resume, but switches to **Claude Haiku** (a faster, cheaper model) for generating the standard HTML/CSS boilerplate.
* **State Management:** The variable `skeleton` is passed down the chain. `skeleton`  `skeleton2`  `skeleton3`. Each agent adds its layer to the evolving file.

---

## 4. Why is this "Agentic"?

A standard script might use a template and just fill in the blanks using f-strings (`{{ name }}`).

**This system is Agentic because:**

1. **It Handles Unstructured Data:** It doesn't need a specific resume format. The agent "reads" the PDF regardless of layout.
2. **It Codes its Own Solutions:** When inserting the "About" section, the agent writes the Python logic to find the insertion point. If the insertion point changed, the agent would adapt its code.
3. **Modular Construction:** It understands the concept of a "Skeleton" vs. "Components," mimicking how modern frontend frameworks (like React) work, but doing it with vanilla HTML/Python.

---

## 5. How to Run

**Prerequisites:**

* `pip install PyPDF2 aisuite python-dotenv`
* Place a file named `resume.pdf` in the root directory.

```python
# 1. Parse the Resume
print("--- 📄 Parsing PDF ---")
# Reads resume.pdf and converts to text
resume_json = parse_basic_details(text)

# 2. Build the Skeleton
print("--- 🏗️ Building Skeleton ---")
skeleton_html = web_skeleton_generator(resume_json)
save_fenced_html(skeleton_html, "skeleton.html")

# 3. Inject "About" Section
print("--- 👤 Injecting Bio ---")
# The agent writes python code to merge the bio into the skeleton
about_code = about_generator(skeleton_html, resume_json)
# We execute that code to get 'skeleton2'
# ... (Execution logic handled in script)

# 4. Inject "Skills" Section
print("--- 🛠️ Injecting Skills ---")
# The agent adds skill cards
skills_code = skills_generator(skeleton2, resume_json)
# ... (Execution logic handled in script)

# 5. Final Output
print("✅ Website ready at skills.html (final version)")

```
