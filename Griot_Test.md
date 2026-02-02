# AI‑Driven Persistent World ("Griot") – n8n Workflow Documentation

## Overview

This workflow implements an AI‑driven, persistent game world using n8n as the orchestration layer. Conceptually, it behaves like a lightweight MUD where the world advances on a fixed schedule and reacts to player input asynchronously.

The **Griot workflow** is the authoritative world engine. It runs on a schedule, reads the current world state from persistent storage, asks an LLM to advance the simulation according to defined rules, publishes narrative updates to Slack, and writes the updated state back to storage.

This document explains *what happens step‑by‑step* in the Griot workflow, using accessible language while remaining technically accurate.

---

## High‑Level Architecture

**Core idea:**
Everything about the world lives in a single JSON document. There is no traditional database or game server.

* **n8n** – Orchestration, scheduling, branching logic, API glue
* **JSONBin** – Persistent world state (REST‑based JSON storage)
* **Groq (LLM)** – Acts as the game master / physics engine
* **Slack** – Player‑facing interface and narrative output

The LLM does *not* store state. It only receives the current world snapshot and produces a structured update, which n8n then merges and persists.

---

## Griot Workflow Purpose

* Advance the world once every **5 minutes** (≈ one in‑game day)
* Detect whether players have acted recently
* Generate new events, consequences, or narrative beats
* Enforce world rules via prompt engineering
* Publish updates to Slack
* Persist the updated world state

---

## Step‑by‑Step Workflow Breakdown

### 1. Schedule Trigger

**Node:** Schedule Trigger

* Runs automatically every **5 minutes**
* Represents the game’s global time tick
* No input required

This guarantees the world continues to evolve even if no players act.

---

### 2. Read World State

**Node:** HTTP Request – `Read World State`

* Performs a `GET` request to JSONBin
* Retrieves the entire world state as a single JSON object

**World state includes:**

* Grid / coordinates
* Entities (players, NPCs, locations)
* Active events
* Pending dice rolls
* Temporal metadata (last update, recent actions)

This JSON is the *source of truth* for the entire system.

---

### 3. Check for Recent Player Actions

**Node:** Code – `Check for Recent Actions`

* Inspects the world state
* Determines whether any player actions occurred since the last tick

**Why this matters:**

* Prevents redundant narration
* Allows the LLM to react differently when players have intervened
* Enables escalation rules (e.g., buildup before major events)

The result is a simple flag or summary passed forward in the workflow.

---

### 4. Generate Griot Response (LLM Call)

**Node:** Code – `Generate Griot Response`

* Constructs a prompt for the Groq LLM
* Injects:

  * Current world state
  * Recent actions summary
  * Rule constraints (the “physics engine”)

**Key rule examples enforced by the prompt:**

* Escalation requires narrative buildup
* Contested actions require dice rolls
* Large changes must justify themselves in‑world
* Rich narrative detail is rewarded

The LLM returns a **structured response**, typically including:

* Narrative text for Slack
* World mutations (state changes)
* Optional dice‑roll requests

The LLM never writes data directly — it only proposes changes.

---

### 5. Merge World Updates

**Node:** Code – `Merge World Updates`

* Takes the LLM’s proposed changes
* Applies them deterministically to the existing world JSON

This step is critical for safety and debuggability:

* Prevents accidental state overwrites
* Ensures only allowed fields change
* Keeps control in n8n, not the model

The output is a fully updated world state object.

---

### 6. Post Narrative Update to Slack

**Node:** HTTP Request – `Post to Slack`

* Sends the generated narrative text to Slack via webhook
* This is the primary player‑facing output

Slack functions as:

* Game log
* Narrative feed
* Interaction surface

---

### 7. Check If a Dice Roll Is Requested

**Node:** IF – `If Dice Roll Requested`

* Examines the merged state / LLM output
* Determines whether a dice roll is required to resolve an action

**If false:**

* Skip directly to saving the world state

**If true:**

* Trigger a dice roll request flow

This enforces uncertainty and fairness in contested situations.

---

### 8. Post Dice Roll Request (Conditional)

**Node:** HTTP Request – `Post Dice Requested`

* Notifies Slack that a dice roll is needed
* May tag players or describe the stakes

Dice are treated as first‑class events, not hidden mechanics.

---

### 9. Prepare World for Write‑Back

**Node:** Code – `Prepare World Write`

* Final cleanup before persistence
* Ensures the JSON matches JSONBin’s expected schema
* Removes transient fields if needed

This step prevents malformed writes and state corruption.

---

### 10. Write World State Back to JSONBin

**Node:** HTTP Request – `Write World Back`

* Performs a `PUT` request to JSONBin
* Overwrites the previous world state with the updated version

At this point:

* The tick is complete
* The world is consistent
* The next workflow run starts from this new state

---

## Player Workflow (Planned / Parallel)

A separate workflow handles player input:

* Slack message → Webhook
* Interpret player intent
* Update the same JSON world state
* Immediately respond in Slack

Both workflows operate on the same persistent JSON, making the system fully stateful without a database.

---

## Key Design Decisions

* **Single JSON State:** Simplifies persistence and debugging
* **LLM as Referee, Not Authority:** n8n controls writes
* **Prompt‑Level Rule Enforcement:** Game logic lives in language, not code
* **Time‑Based World Advancement:** World progresses even without players

---

## Current Status

* Griot workflow runs end‑to‑end
* Slack posting works reliably
* Actively debugging a state persistence edge case
* Need to update day increment, refine narrative voice, implement player flow

---

## What This Demonstrates

* Designing multi‑workflow systems
* Orchestrating LLMs as deterministic components
* Managing persistent state without a database
* Debugging complex, stateful automation pipelines

---

*This system intentionally blurs the line between game engine, narrative machine, and automation workflow — and treats n8n as all three.*
