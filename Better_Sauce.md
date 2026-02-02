# Better Sauce – Wikipedia Synthesis Agent (n8n Workflow Documentation)

## Overview

**Better Sauce** is an autonomous research-and-synthesis agent built in n8n. Its job is simple but non-trivial: read **eight random Wikipedia pages** and produce a **single, cohesive journal-style entry** that connects ideas, themes, or tensions across otherwise unrelated topics.

This workflow demonstrates controlled looping, state accumulation, external data ingestion, and LLM-based synthesis — all without treating the model as a black box.

---

## Conceptual Model

* Wikipedia pages = raw sensory input
* n8n = memory, control flow, and sequencing
* LLM (Groq) = synthesis engine
* Output = reflective journal entry, not a summary list

The agent is intentionally *interpretive*, not encyclopedic.

---

## Step-by-Step Workflow Breakdown

### 1. Schedule Trigger

**Node:** Schedule Trigger

* Starts the workflow on a defined schedule (or manual run)
* Represents the moment the agent “wakes up” to write

---

### 2. Set – Start Here

**Node:** Set – Start Here

* Initializes the workflow run
* Acts as a visual and logical anchor point
* Useful for debugging and re-running partial flows

No data transformation occurs here.

---

### 3. Set – Init Storage

**Node:** Set – Init Storage

* Creates an empty data structure to hold:

  * Collected page summaries
  * Page titles / metadata
  * Loop counters

This is the agent’s **working memory** for the session.

---

### 4. HTTP Request – Fetch Wikipedia

**Node:** HTTP Request – Fetch Wikipedia

* Calls the Wikipedia API
* Requests a **random article**
* Retrieves raw page content and metadata

Each execution fetches exactly one page.

---

### 5. Code – Process Page

**Node:** Code – Process Page

* Cleans and normalizes the Wikipedia response
* Extracts:

  * Title
  * Introductory text or relevant excerpt
* Appends this data to the session storage

This ensures consistency across pages before synthesis.

---

### 6. If – Loop Control

**Node:** If

* Checks how many pages have been collected so far

**If fewer than 8 pages:**

* Continue looping

**If 8 pages collected:**

* Exit loop and move to synthesis

This prevents infinite loops and enforces scope discipline.

---

### 7. Set – Loop Control (True Branch)

**Node:** Set – Loop Control

* Increments loop counters
* Routes execution back to the Wikipedia fetch step

This creates a controlled, deterministic loop within n8n.

---

### 8. Code – Prepare for AI (False Branch)

**Node:** Code – Prepare for AI

* Consolidates all collected page excerpts
* Formats them into a single structured prompt input
* Emphasizes:

  * Reflection over summarization
  * Thematic linkage
  * Journal-like tone

No API calls occur yet — this is prompt assembly.

---

### 9. Code – Build Groq Request

**Node:** Code – Build Groq Request

* Wraps the prepared prompt into Groq’s API schema
* Defines:

  * Model parameters
  * Token limits
  * System vs user instructions

This step separates **prompt logic** from **transport logic**.

---

### 10. HTTP Request – Groq API

**Node:** HTTP Request – Groq API

* Sends the assembled request to Groq
* Receives the LLM-generated journal entry

The model’s task:

> “Given these eight unrelated Wikipedia pages, write a cohesive journal entry that finds meaning, resonance, or contrast between them.”

---

### 11. Code – Extract Report

**Node:** Code – Extract Report

* Parses the Groq API response
* Extracts only the generated text
* Removes metadata and token usage details

This isolates the final artifact.

---

### 12. Display

**Node:** Display

* Outputs the journal entry
* Used for inspection, logging, or downstream publishing

This is the end of the agent’s cognitive loop.

---

## Design Intent

* **Randomness with bounds:** Exactly eight pages, no more, no less
* **Interpretation over accuracy:** The agent is allowed to speculate
* **LLM as synthesizer, not scraper:** All data collection is deterministic
* **Explicit memory:** State is accumulated manually, not implicitly

---

## What This Workflow Demonstrates

* Controlled looping in n8n
* Stateful aggregation across iterations
* Clean separation of:

  * Data fetching
  * Data normalization
  * Prompt construction
  * Model invocation
* Designing agents that *think with material*, not just react to prompts

---

*Better Sauce is less about knowledge retrieval and more about meaning-making under constraint.*
