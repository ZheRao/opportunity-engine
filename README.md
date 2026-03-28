# opportunity-engine

## Overview

**opportunity-engine** is a prototype system for mapping real-world conditions to actionable opportunities.

This repository explores how to transform messy, unstructured external information (e.g., government grants) into structured, decision-ready intelligence.

The initial use case focuses on **grant eligibility and recommendation**, but the underlying architecture is designed to generalize to broader decision systems.



## Core Idea

> Most systems answer: *"What exists?"*
>
> This system answers: *"What is worth pursuing?"*

The engine bridges three layers:

* **Reality (external signals)** → messy, ambiguous, unstructured
* **Structure (internal representation)** → normalized, queryable
* **Decision (actionable output)** → ranked, contextualized recommendations



## Prototype System Architecture

The system is composed of three core components with clear boundaries.

### 1. Grant Knowledge Layer

Responsible for collecting and structuring grant information from official sources.

**Input:**

* Government websites
* PDFs / policy documents

**Process:**

* Extract raw text (deterministic where possible)
* Use LLM-assisted normalization to convert into semi-structured schema
* Preserve original source text for auditability

**Output:**

* Structured grant records



### 2. Deterministic Eligibility Engine

Responsible for narrowing the opportunity space using hard constraints.

**Input:**

* Client profile (structured)
* Grant database

**Process:**

* Apply rule-based filtering (e.g., province, industry, size, project type)

**Output:**

* Defensible subset of candidate grants

**Key Property:**

* Fully deterministic, explainable, and reproducible



### 3. Interactive Recommendation Engine (LLM)

Responsible for reasoning over candidates and guiding decision-making.

**Input:**

* Filtered candidate grants
* Client profile

**Process:**

* Identify missing information (soft constraints)
* Generate follow-up questions
* Evaluate trade-offs across candidates
* Rank and group opportunities

**Output:**

* Follow-up questions
* Ranked recommendations
* Explicit reasoning and assumptions



## End-to-End Flow

```
External sources
    ↓
Structured grant database
    ↓
Client hard constraints
    ↓
Deterministic filtering
    ↓
Candidate subset
    ↓
LLM-guided questioning + ranking
    ↓
Actionable recommendations
    ↓
Human validation
```



## Design Principles

### 1. Separation of Concerns

* Knowledge extraction, filtering, and reasoning are independent layers

### 2. Deterministic Before Probabilistic

* Reduce search space with rules before applying LLM reasoning

### 3. Auditability

* Every recommendation must trace back to:

  * Source documents
  * Applied filters
  * Reasoning steps

### 4. Human-in-the-Loop

* The system supports consultants, not replaces them

### 5. Progressive Narrowing

```
Many opportunities
    ↓
Filtered subset
    ↓
Ranked shortlist
    ↓
Actionable decisions
```



## Why This Exists

In many domains (grants, finance, compliance, etc.), the bottleneck is not access to information — it is:

* interpreting requirements
* matching context
* deciding what is worth pursuing

This project explores a generalizable pattern:

> **Opportunity Intelligence Systems**



## Current Scope (Prototype)

This repository is intentionally scoped as a **prototype / exploration**.

It focuses on:

* Validating system boundaries
* Testing component interactions
* Building small, working end-to-end flows

It does NOT aim to:

* Be production-ready
* Handle full data coverage
* Optimize for scale or performance (yet)



## Future Directions

* Approval likelihood estimation
* Effort / complexity scoring
* Historical outcome learning
* Knowledge graph representation of grants
* Generalization beyond grants (other opportunity domains)



## Repository Philosophy

This repository serves as:

* A **working prototype**
* A **thinking artifact**
* A **record of evolving system design**

The focus is on:

* clarity of architecture
* explicit reasoning
* incremental iteration



## Summary

**opportunity-engine** is not a grant search tool.

It is an early exploration of a broader idea:

> Systems that convert unstructured reality into structured decisions.
