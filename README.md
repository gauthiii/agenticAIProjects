# Agentic AI Projects

This repository contains a collection of **Agentic AI workflows** built using structured multi-agent architectures. These projects explore how **LLM planning, tool usage, reflection loops, evaluation feedback, and continuous improvement** can be designed in real-world automated systems.

---

## Project Philosophy

Agentic AI goes beyond simple prompt-based responses. These projects follow a **systematic agent design workflow**:

Stage | Purpose
----- | --------
Initial Planning & Task Decomposition | Break down a user problem into logical steps
Reflective Design | Think before acting – evaluate steps and plan improvements
Tool Use (Functions & External APIs) | Use tools such as databases, file systems, web search, and compute functions
Evals & Error Analysis | Use feedback from LLM self-evaluation and/or external evaluators
Iterative Refinement | Improve responses using refinement loops
Impact Measurement | Show how structured planning improves output quality and reliability

---

## Project Index

S.No | Project Name | Description | Key Concepts
---- | -------------|-------------|--------------
1 | Interactive Chart Generator | Generates visual insights from CSV files and refines based on feedback | File Parsing, Visualization, Reflection
2 | SQL Query Generation Agent | Validates SQL queries from DB schemas with reflection loops | DB Tools, Planning
3 | Tool Toggle Agent | Chooses and toggles tools based on task goals | Decision Routing
4 | Content Writing Agentic AI | Generates and refines content using constructive feedback | Reflection, Evals
5 | Research Agentic AI | Researches topics using external tools and citation refinement | Web Tools, Self-Correction

---

## Tech Stack

- Python
- LangChain / LangGraph
- LLM APIs
- Plotly / Matplotlib
- SQLite/Postgres
- Docker (optional)

---

## Repo Layout

agenticAIProjects/
- charts_agent
- sql_agent
- tool_toggle_agent
- content_writer_agent
- research_agent

---

## Contributions
Pull requests are welcome.

---