
# Knowledge Layer

## Purpose

The Knowledge Layer is a **truth-acquisition system** that transforms unstructured web sources into **auditable, structured grant data** for downstream decision systems.

It does not assume clean inputs.
It operates over noisy, heterogeneous, and evolving sources.


## Responsibilities

1. Discover candidate sources  
   → `discovery`

2. Traverse source structures and identify candidate terminal nodes  
   → `navigator`

3. Extract raw content from identified terminals (without interpretation)  
   → `parser`

4. Interpret raw content into structured semantic fields  
   → `interpreter`

5. Produce auditable, schema-aligned outputs with full provenance  
   → `interpreter`

## System Architecture

```text
source discovery
  -> source navigation (graph traversal)
    -> node classification (structure detection)
      -> terminal detection (candidate extraction points)
        -> raw content extraction
          -> semantic interpretation
            -> schema normalization
              -> database
```


This system can be understood as a **graph-walker with perception and interpretation layers**.

## Subsystems

### 1. Discovery System

Goal:  
Identify high-level entry points (roots) for grant exploration.

Output:
- source URLs
- domain scope
- initial traversal seeds

Characteristics:
- wide and shallow
- low precision, high recall
- not responsible for correctness of grant detection

### 2. Navigator System (Phase-1 Focus)

Goal:  
Traverse a source and identify **candidate terminal nodes** that may contain grant information.

Core functions:
- node classification (container vs terminal vs irrelevant)
- link discovery and filtering
- traversal decision-making

Key constraint:
- does not interpret grant semantics
- operates only on structure and weak signals

Output:
- candidate terminal nodes
- traversal trace (path, decisions, reasoning)

### 3. Parser System

Goal:  
Extract **raw content faithfully** from candidate terminal nodes.

Handles:
- HTML pages
- PDFs
- (future) structured forms

Output:
- normalized raw representation:
    - text
    - headings
    - metadata
    - source reference

Key constraint:
- no semantic interpretation
- no field extraction

### 4. Interpreter System

Goal:  
Transform raw content into **structured, schema-aligned grant data**

Core functions:
- classify whether content is a valid grant
- extract structured fields:
    - eligibility
    - funding
    - deadlines
    - requirements
- handle ambiguity and missing data
- attach evidence to every extracted field

Output:
- structured grant objects with provenance

## Core Invariants

### 1. Raw truth is preserved before interpretation

All parsing outputs must retain:
- full raw content
- source reference
- extraction metadata

### 2. Structure and meaning are strictly separated
| Layer       | Responsibility         |
| ----------- | ---------------------- |
| Navigator   | structure detection    |
| Parser      | raw content extraction |
| Interpreter | semantic meaning       |

No cross-contamination allowed.

### 3. Terminal detection is the key decision point

The system’s correctness depends on identifying:
> where a meaningful grant unit begins and ends

### 4. All outputs must be auditable

Every structured field must trace back to:
- source document
- text span or evidence

### 5. The system must tolerate ambiguity and incompleteness

Missing, conflicting, or vague information must be explicitly represented.

### 6. Failures must be visible

Uncertainty in:
- classification
- extraction
- interpretation

must be recorded, not hidden.

## Mental Model

This system is not a scraper.

It is:
> a perception system that converts messy external signals into structured, trustworthy knowledge

More specifically:
- Discovery = coarse attention
- Navigator = structural perception
- Parser = sensory capture
- Interpreter = semantic reasoning

## Notes
- This is an internet-scale extension of a tree-walker architecture
- The system operates over a **graph**, not a clean tree
- Nodes may contain:
    - partial information
    - duplicated information
    - misleading signals

Robustness comes from:
- separation of concerns
- preservation of raw evidence
- explicit uncertainty handling