# Phase-1 Objective

Goal:
Transform a known source root into a set of **candidate grant-bearing terminal nodes**, 
with full traversal trace and raw evidence preserved for downstream extraction.

## Core Responsibilities

### 1. Node Classification (state classification)

Given a page (node), determine its structural and semantic role.

Questions:
- What type of node is this?
    - irrelevant
    - structural container (portal / category)
    - listing page (contains multiple programs)
    - candidate terminal (single grant/program)
    - supporting document (PDF / guide)
    - ambiguous
- Does this node contain grant-related signals?
- Should this node be explored further?

Output:
- `node_type`
- `relevance_score` (optional)
- `confidence`
- `classification_evidence` (text snippets / signals)

### 2. Sub-content Discovery (link extraction + filtering)

Given a node, identify candidate links for traversal.

Questions:
- What links exist on this page?
- Which links are likely relevant to grant discovery?

Output:
- `candidate_links`: [{url, anchor_text, context}]
- `link_priority` (optional ranking)
- `link_filter_reasoning` (why included/excluded)

### 3. Traversal Decision (control flow)

Given node classification + discovered links, determine next action.

Questions:
- Should this node be traversed further?
- Should this node be treated as a terminal?
- Should traversal terminate at this branch?

Output:
- `action` ∈ {EXPAND, EXTRACT, TERMINATE}
- `next_links` (subset of candidate_links if EXPAND)
- `termination_reason` (if TERMINATE)

## Key Invariant

Traversal decisions are made based on **node type**, not ad hoc page-specific rules.

# Phase-1 Architecture

## System Model

A **recursive graph-walker** operating over web pages.

At each node:

1. Fetch page
2. Apply `state_classifier`
3. Apply `link_extractor`
4. Apply `decision_policy`
5. Execute traversal accordingly

## Core Components

### 1. `state_classifier(node)`

Input:
- page content (HTML / text)
- metadata (URL, title, headings)

Output:
```json
{
  "node_type": "...",
  "confidence": 0.0,
  "signals": [...]
}
```
### 2. `link_extractor(node)`

Input:
- raw HTML

Output:
```
[
  {
    "url": "...",
    "anchor_text": "...",
    "context": "..."
  }
]
```

### 3. `decision_policy(classification, links)`

Output:
```
{
  "action": "EXPAND | EXTRACT | TERMINATE",
  "next_links": [...],
  "reason": "..."
}
```
## Action Semantics
- `EXPAND`  
→ recursively traverse selected links
- `EXTRACT`  
→ emit this node as a candidate terminal for downstream parsing
→ (no parsing happens in Phase-1)
- `TERMINATE`  
→ stop traversal on this branch

## Critical Constraints
1. No interpretation in Phase-1
    - Do not extract structured fields
    - Only identify candidate terminals
2. Raw evidence must be preserved
    - full text
    - URL
    - traversal path
3. Traversal must be bounded
    - depth limits
    - domain constraints
    - deduplication
4. All decisions must be explainable
    - classification signals recorded
    - traversal reasons logged
