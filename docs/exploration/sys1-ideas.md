# initial architecture ideas - 2026-03-29

Yes — and your repo framing is already pointing at the right decomposition: **knowledge extraction → deterministic narrowing → LLM reasoning**. 
The part you’re exploring now is the hardest one: turning messy, changing grant sources into a stable JSON knowledge layer.

Your invariant is good, but I’d tighten it slightly:

**Invariant:**  
**Every grant program must be represented as:**
- raw source evidence,
- normalized structured fields, and
- traceable provenance linking the structure back to the evidence.

That third part matters a lot. Without provenance, your JSON becomes hallucination-prone data.

## The real design question

Not “how do I use an LLM to get grants?”

It is:

**What pipeline best converts unstable public information into auditable structured opportunity records?**

That opens a few possible extraction strategies.

### Option 1 — LLM-first discovery, then structure later

This is closest to your current 30-minute plan.

Flow:
1. Search broadly for grant-related pages
2. Pull as much text as possible
3. Ask LLM to summarize / identify candidate grant opportunities
4. Store raw summaries
5. Later run a second pass to extract JSON fields

Why it’s attractive:
- fastest prototype
- low engineering overhead
- useful for learning source diversity quickly

Why it’s dangerous:
- summary becomes an information bottleneck
- if the first pass compresses badly, later structure extraction is corrupted
- hard to audit whether a field came from source text or model inference
- duplicate grants may be rediscovered in inconsistent forms

Verdict:
**Good for rapid exploration, weak as the long-term ingestion backbone**.

Use this only as a **discovery bootstrap**, not as your final method.

---
### Option 2 — Deterministic text extraction first, LLM only for normalization

This is the strongest default architecture.

Flow:
1. Crawl / collect source URLs
2. Deterministically extract raw text from HTML/PDF
3. Store raw document snapshots
4. Chunk text into logical sections
5. Use LLM to map text into schema fields
6. Store structured JSON plus source spans / evidence

Why it’s strong:
- preserves raw truth
- lets you re-run normalization later with better prompts/models
- separates extraction failure from interpretation failure
- easy to audit and debug

This matches your repo philosophy best:
**deterministic before probabilistic**.

Verdict:
**Best foundational method**.

If I were shaping `opportunity-engine`, this would be the backbone.

---
### Option 3 — Schema-guided extraction page by page

Instead of “summarize first,” you immediately ask:

“Given this source text, fill this schema and mark unknown fields as null.”

Flow:
1. Extract source text
2. Send source text + strict schema to LLM
3. Require:
    - field value
    - confidence
    - evidence quote/snippet
    - unknown/null when unsupported
4. Merge outputs across chunks/pages

Why it’s good:
- forces structured thinking early
- avoids fuzzy summaries
- easier to load into DB immediately

Why it’s hard:
- long documents spread one field across many pages
- chunk-level extraction can miss cross-page dependencies
- merging partial outputs is nontrivial

Verdict:  
**Very good, but only if you also store raw text and evidence**.  
Do not let the JSON be the only artifact.

---
### Option 4 — Two-stage extraction: candidate detection → deep normalization

This may be the best compromise for messy web ecosystems.

#### Stage A: candidate discovery

Goal: identify pages/documents likely to describe an actual grant.

Output:
```
{
  "source_url": "...",
  "document_type": "grant_page | program_guide | news | aggregator | irrelevant",
  "program_name_guess": "...",
  "is_likely_grant": true,
  "reason": "..."
}
```
#### Stage B: structured normalization

Only if likely relevant:
- parse full page/document
- extract full schema
- attach evidence spans

Why this is strong:
- avoids wasting expensive normalization on junk
- lets you separate “find possible opportunities” from “understand the opportunity”
- scales better later

Verdict:  
**Probably the most practical architecture for real deployment**.

---
### Option 5 — Human-assisted curation pipeline

Very underrated for internal tools.

Flow:
1. System proposes candidate grants + extracted JSON
2. Human reviews in lightweight UI
3. Human approves / edits / rejects
4. Approved records enter canonical database

Why this matters:
- grant language is ambiguous
- deadlines and eligibility change
- hidden constraints are common
- internal consultant trust matters more than full automation

Verdict:  
**This should almost certainly exist, even if later**.  
For internal use, this may be more valuable than chasing full autonomy.

## Recommended mental model

Do not think of the extraction layer as “LLM searches the web and gives me grant info.”

Think of it as 4 subproblems:

### 1. Discovery

Where are candidate sources?

Examples:
- official program pages
- ministry / agency portals
- intake announcements
- PDF guidelines
- application manuals
- archived program pages
- RSS/news release pages

### 2. Acquisition

How do I fetch and preserve the source?

Artifacts to store:
- raw HTML
- extracted text
- PDF text
- snapshot timestamp
- source URL
- title
- last seen / last changed

### 3. Interpretation

How do I convert source content into stable fields?

This is where LLM belongs.


### 4. Canonicalization

How do I decide whether this is:
- a new grant,
- an update to existing grant,
- duplicate mention,
- expired / archived version?

This part is easy to underestimate, but it becomes a major problem fast.

## The key architecture choice: what is your unit of storage?

You need at least three storage layers.

### Layer 1 — Raw source objects

Store source truth.

Example:
```
{
  "source_id": "src_001",
  "url": "...",
  "retrieved_at": "...",
  "content_type": "html",
  "raw_text": "...",
  "title": "...",
  "hash": "..."
}
```
### Layer 2 — Parsed evidence objects

Intermediate representation.

Example:
```
{
  "source_id": "src_001",
  "sections": [
    {
      "section_id": "sec_01",
      "heading": "Eligibility",
      "text": "...",
      "char_start": 1200,
      "char_end": 1890
    }
  ]
}
```
### Layer 3 — Canonical grant records

Your normalized grant database.

Example:
```
{
  "grant_id": "grant_abc",
  "grant_name": "...",
  "status": "open",
  "deadline": "...",
  "province": ["SK"],
  "eligibility": {...},
  "source_evidence": [
    {
      "source_id": "src_001",
      "field": "deadline",
      "evidence_text": "...",
      "section_id": "sec_03"
    }
  ]
}
```
That structure protects you from future reprocessing pain.

## Important brainstorm directions

### Direction A — Fixed schema vs evolving schema

At the start, a rigid schema is tempting, but grants are messy.

You likely want:

**stable core fields**
- program name
- source URL
- jurisdiction
- deadline
- amount
- status
- applicant type
- project type

**flexible extension fields**
- cost-share rules
- matching funds
- partnerships required
- indigenous eligibility
- geographic restrictions
- sector-specific subrules

Best pattern:
```
{
  "core": {...},
  "hard_constraints": {...},
  "soft_constraints": {...},
  "extra_fields": {...}
}
```
Not everything deserves first-class columns immediately.

### Direction B — Hard constraints vs descriptive facts

Do not ask the LLM to decide too early what is “hard” vs “soft.”

Better:
1. extract all relevant facts
2. classify them later into:
    - deterministic filterable constraints
    - advisory / soft considerations
    - ambiguous human-review items

Because many grant pages phrase hard constraints vaguely.

### Direction C — One-shot extraction vs iterative extraction

A one-pass prompt may miss buried requirements.

Better approach:
- pass 1: extract program overview
- pass 2: extract eligibility
- pass 3: extract funding mechanics
- pass 4: extract deadlines and application details
- pass 5: extract hidden constraints / caveats

This mirrors how humans read dense policies.

### Direction D — Grant-level record vs versioned record

Grants change over time.

So instead of only:
```
grant_id -> current state
```
You may eventually want:
```
grant_id -> versions[]
```
Because:
- deadline may change
- intake can reopen
- amount can change
- criteria can tighten

Even if you do not fully implement versioning now, design so you can add it later.

## A strong prototype architecture

Here is the version I would recommend for your current stage.

### Prototype v1

**Step 1 — Source discovery**

Collect candidate URLs manually + search-assisted.

Store:
- URL
- source type
- jurisdiction
- notes

Important idea: 
- auto-discovery + manual double checking sources
- grant sources (general source, not specific grant)

**Step 2 — Raw extraction**

For each URL:
- fetch page or document
- extract raw text deterministically
- save snapshot

Important note:
- need to explore methods for deterministic raw text extraction

**Step 3 — LLM structured extraction**

Prompt LLM to return:
- strict JSON schema
- unknown as null
- confidence per field
- supporting evidence snippets

Important idea:
- instead of squashing full JSON file into LLM, use an iteration, one item (one grant id, source URL,...) per LLM query
- each iteration, LLM just need to refer facts from one grant and structure JSON file!

**Step 4 — Canonical grant registry**

Normalize into one table/document per grant:
- deduplicate by name + issuer + jurisdiction + URL patterns

**Step 5 — Review queue**

Flag records for human review when:
- confidence low
- critical fields missing
- contradictory evidence
- duplicate suspicion

This is enough to learn a lot without overcommitting.

## Prompting pattern that will likely work well

Instead of:  
“Summarize this grant page.”

Use:  
“Extract supported facts only.”

Prompt objective:
- fill schema
- do not infer unsupported values
- return null if absent
- include evidence text for each populated field
- distinguish exact statements from interpretation

This is huge.

Because your enemy is not missing data.
Your enemy is **false certainty**.

## Failure modes you should expect early

### 1. Duplicate grants across pages

Same program appears on:
- overview page
- PDF guide
- ministry listing
- news release

### 2. Ambiguous program status

Open? closed? rolling? upcoming?

### 3. Incomplete source coverage

Some pages describe benefits but not full requirements.

### 4. Buried constraints

Matching funds, partnership requirements, location limits.

### 5. LLM over-normalization

Model turns vague language into crisp-but-false structure.

### 6. Schema mismatch

Not all programs have max amount, fixed deadline, or clean industry tags.

That is why raw evidence preservation is non-negotiable.

## Concrete methodology options, ranked

### Best long-term backbone

Deterministic extraction → raw storage → LLM normalization → evidence-backed JSON

### Best fastest prototype

Search + manual source collection → raw text extraction → schema extraction

### Best scalable production direction

Candidate discovery classifier → normalization pipeline → review queue → canonical versioned DB

### Weakest but fastest experimental shortcut

LLM summarizes the web directly and you store the summary

That last one is okay for a weekend spike, but not as the system’s foundation.

## My blunt recommendation

Do **not** harden around “LLM does extensive web-search and outputs summarized text.”

That is fine for ideation, but architecturally it is too lossy.

Harden instead around this invariant:
> Every structured grant record must be reproducible from stored source artifacts.

That gives you freedom later to swap:
- model
- prompt
- schema
- ranking logic
- DB structure

without losing trust.

## A clean way to phrase your current exploration

You are not yet choosing an implementation.

You are exploring this design space:
1. How to discover relevant grant sources
2. How to preserve source truth
3. How to normalize messy text into JSON
4. How to maintain canonical, deduplicated grant records over time

That is the correct abstraction boundary.

Lean in — core growth: the strongest insight here is that the system’s real moat is not “LLM search,” but 
**auditable normalization of changing external reality into stable internal structure.**