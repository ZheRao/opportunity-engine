# Understanding Grants

Great starting point — you’re essentially mapping the “**data surface area**” of grants before building extraction logic. 
I’ll keep this structured and system-oriented so you can directly translate it into your `opportunity-engine`.

## 1) What is a “grant” (quick mental model)

A grant is **non-repayable funding** (usually from government) given to organizations (businesses, nonprofits, researchers) to **achieve a policy goal**.

**Core invariant (important for your system design)**

A grant is **not just money** — it’s a **contract with constraints + intent**.

Every grant encodes:
- **Who qualifies** (eligibility constraints)
- **What activities are funded** (policy intent)
- **How much & how** (funding structure)
- **How success is measured** (reporting requirements)

👉 Think of grants as:
```
Grant = { constraints + incentives + policy objective }
```
This is critical because:
- Your system is not just retrieving grants
- It is matching **client state → constraint space**

## 2) Where grant information lives (Canada)

We’ll go top-down:

### Federal (Canada-wide)

**Primary sources**

#### 1. Government of Canada – central portals
- Main hub: Grants and Funding
- Other key departments:
    - Innovation, Science and Economic Development Canada (business innovation grants)
    - Natural Resources Canada (energy, sustainability)
    - Agriculture and Agri-Food Canada (agriculture)

👉 Formats:
- Mostly **web pages**
- Sometimes **linked PDFs**
- Occasionally **application portals**

#### 2. National Research Council Canada
- One of the most important business grants (IRAP)
- Semi-standardized structure but still mostly web text

#### 3. Canada Business Network
- Aggregates grants across provinces
- Useful for discovery, not authoritative source

#### Federal format patterns

You’ll see:
- Structured-ish HTML pages
- Long-form descriptive text
- Embedded eligibility rules
- External application systems

### Saskatchewan (SK Provincial)

**Primary sources**

#### 1. Government of Saskatchewan
- Main grants portal (less centralized than federal)
- Departments:
    - Agriculture
    - Trade & Export Development
    - Energy & Resources

#### 2. Saskatchewan Ministry of Agriculture
- Key for ag-related funding (very relevant in SK)

#### 3. Saskatchewan Trade and Export Partnership
- Export-related funding and programs

#### 4. Regional / Crown / Agencies
- Economic development agencies
- Innovation Saskatchewan
- Industry-specific programs

#### SK format patterns

Compared to federal:
- More fragmented
- More PDFs
- Less standardized wording
- Sometimes outdated pages

## 3) Common formats of grant information (VERY important for extraction)

You’ll encounter 3 main types:

### 1. Web pages (most common)

Structure:
- Semi-structured HTML
- Headings + paragraphs
- Lists

👉 Best for:
- LLM + rule-based extraction

### 2. PDFs (VERY common in provinces)

Types:
- Program guides
- Application manuals
- Policy documents

👉 Challenges:
- No consistent structure
- Tables + paragraphs mixed
- OCR sometimes needed

### 3. Application portals / forms
- Often separate systems
- Structured fields exist but not easily scrapable

👉 Insight:
- These are **downstream**, not primary data sources

## 4) What sections are typically included in a grant

This is the most important part for your system.

### Canonical schema (you should base your JSON on this)

1. Basic Info
    - Program name
    - Provider (federal / SK / agency)
    - Description
2. Funding Details
    - Amount (e.g., up to $50,000)
    - Cost-share (e.g., 50%)
    - Eligible expenses
    - Payment structure (reimbursement vs upfront)
3. Eligibility (HARD constraints)  
These are your deterministic filters.
    - Location (Canada / SK / specific regions)
    - Industry (agriculture, tech, etc.)
    - Company size (SME, employees, revenue)
    - Incorporation status
    - Project type
4. Requirements (SOFT constraints)  
These are fuzzy:
    - Innovation level
    - Environmental impact
    - Growth potential
    - Alignment with policy goals
5. Deadlines
    - Application deadlines
    - Intake periods (rolling vs fixed)
6. Application Process
    - Steps
    - Required documents
    - Evaluation criteria (sometimes vague)
7. Contact / Admin
    - Program contact
    - Links to apply

## 5) How consultants think about grants (important for your UX layer)

Typical prioritization:

1. Hard constraints first
    - “Are you even eligible?”
2. Funding size / ROI
    - “Is this worth applying?”
3. Effort vs probability
    - “How hard is this application?”
4. Fit to client narrative
    - “Can we justify this story?”

## 6) Critical insight (Lean in — core growth)

You’re not building a scraper.

You’re building:
> A constraint-extraction + decision system

Most people fail here because they:
- Treat grants as documents
- Instead of treating them as decision objects

## 7) Suggested initial JSON schema (v1)

Start simple:
```
{
  "program_name": "",
  "provider": "",
  "jurisdiction": "federal | SK",
  
  "funding": {
    "max_amount": null,
    "cost_share": null,
    "eligible_expenses": []
  },
  
  "eligibility": {
    "location": [],
    "industries": [],
    "company_size": {},
    "incorporation_required": null
  },
  
  "requirements": {
    "innovation": null,
    "environmental": null,
    "other": []
  },
  
  "dates": {
    "deadline": null,
    "intake_type": ""
  },
  
  "raw_text": ""
}
```