# Project 2: SQL Query Generation Agent

## 1. Project Overview

This project builds a **Database-Reflective Agent**. While many LLMs can write SQL, they often write queries that are syntactically correct but semantically useless (e.g., returning an ID number when the user asked for a product name).

**Goal:** Create an agent that generates SQL, executes it against a real SQLite database, "looks" at the returned data rows, and refines the query if the answer doesn't match the user's intent.

---

## 2. The Agentic Workflow

The workflow evolves from a simple "Text-to-SQL" script into a self-correcting agent through three distinct phases.

### Phase 1: Naive Generation (The "Junior Dev")

1. **Input:** User Question ("Which color has the highest sales?") + DB Schema.
2. **Action:** The LLM generates the initial SQL query (V1).
3. **Result:** It might generate a query that sums sales but returns `product_id` instead of the color name.

### Phase 2: Static Reflection (The "Code Reviewer")

1. **Action:** The agent reviews its own SQL code *before* running it.
2. **Logic:** It checks for obvious syntax errors or hallucinations (e.g., using a column `profit` that doesn't exist in the schema).
3. **limitation:** It cannot see the *data*, only the code. It might approve a query that looks correct but returns empty results.

### Phase 3: Dynamic Refinement (The "QA Engineer")

*This is the advanced Agentic step.*

1. **Action:** The agent **executes** the query against `products.db`.
2. **Observation:** It captures the actual returned rows (the Pandas DataFrame).
3. **Critique:** It compares the **User Question** vs. the **Actual Data Output**.
* *User:* "Which color..."
* *Data:* Returns a table with just numbers (IDs).
* *Agent Realization:* "I returned IDs, but the user asked for color names. I need to join with the products table or select the `color` column."


4. **Correction:** It rewrites the SQL to include the missing human-readable columns.

---

## 3. Code Breakdown

### Key Functions

| Function Name | Description |
| --- | --- |
| `generate_sql` | **The Drafter.** Takes the user question and schema string. Returns raw SQL code. |
| `refine_sql` | **The Static Reviewer.** Looks at the *code* only. Useful for catching syntax errors, but misses data-logic errors. |
| `refine_sql_external_feedback` | **The Dynamic Reviewer.** Takes the SQL code **AND** the execution result (DataFrame). This is the most powerful function because it allows the agent to see if its answer actually makes sense to a human. |
| `utils.execute_sql` | **The Tool.** Connects to the SQLite database, runs the query, and returns the results as a Pandas DataFrame for inspection. |

### Technical Details

* **Schema Injection:** The schema is passed as a string context to the LLM. This ensures the model knows exactly which columns (`product_name`, `qty_delta`, etc.) are valid.
* **JSON Structured Output:** The refinement functions request `STRICT JSON` responses. This separates the "Thought" (`feedback`) from the "Action" (`refined_sql`), allowing the system to parse the next step programmatically.
* **Pandas-to-Markdown:** In `refine_sql_external_feedback`, the dataframe is converted to a markdown string (`df.to_markdown()`). This allows the LLM to "read" the database output as text.

---

## 4. Why is this "Agentic"?

A standard LLM script is "fire and forget"—it gives you SQL and hopes for the best.

**This system is Agentic because:**

1. **It Validates Reality:** It doesn't trust its own code until it sees the execution results.
2. **It Bridges the Gap:** It solves the common disconnect where code is "valid" (no error messages) but "wrong" (useless answer).
3. **It Loops:** It uses the output of one step (execution) as the input for the next step (refinement).

---

## 5. How to Run

Ensure your `.env` is set and you have the `utils.py` module (which handles the DB connection).

```python
# 1. Setup the DB
utils.create_transactions_db()
schema = utils.get_schema('products.db')

# 2. Ask a question
question = "Which color of product has the highest total sales?"

# 3. Initial Generation
sql_v1 = generate_sql(question, schema, "openai:gpt-4o")

# 4. Execute V1
df_v1 = utils.execute_sql(sql_v1, 'products.db')

# 5. Agentic Refinement (The magic step)
feedback, sql_v2 = refine_sql_external_feedback(
    question=question,
    sql_query=sql_v1,
    df_feedback=df_v1, # Passing the actual data results!
    schema=schema,
    model="openai:gpt-4o"
)

# 6. Final Result
df_v2 = utils.execute_sql(sql_v2, 'products.db')
print(f"Final Answer: \n{df_v2}")

```
