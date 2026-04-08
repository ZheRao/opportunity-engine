# Exploring node classifier architecture - [2026-04-07]

You’re rejecting two bad extremes:
- **LLM runtime classifier** → flexible, but feels too soft and non-deterministic
- **bag-of-words heuristic soup** → deterministic, but brittle and dumb

And you’re right to dislike both.

The better framing is:
> runtime classification should be policy-driven and inspectable, not “AI magic” and not “keyword garbage.”

So the real question is not:
> LLM or bag-of-words?

It is:
> what is the lowest-complexity deterministic decision system that still respects the structure of the problem?

## My blunt view

For Phase 1, I would not use either as the core architecture.

Instead, I would build a **deterministic evidence-scoring classifier** with explicit features.

That is different from crude bag-of-words.

### What that means

Not this:
```
if "grant" in text -> relevant
```
But more like:
```
node_type_score = weighted combination of:
- URL pattern signals
- title signals
- heading signals
- anchor-text context
- page structure signals
- file type
- domain/location in site hierarchy
- presence of deadline/funding/eligibility phrases
- number/type of outgoing links
- whether page looks like a listing vs a terminal
```
Then the policy says:
- high container score + many relevant links → `EXPAND`
- high terminal score + funding/eligibility signals → `EXTRACT`
- low relevance / nav junk / duplicate → `TERMINATE`

That is still deterministic.  
But it is not dumb keyword matching.

## The middle path you probably want

### 1. Deterministic feature extraction

Extract features from the page:

**Structural features**
- URL path tokens
- content type (html, pdf)
- depth from source root
- number of outgoing internal links
- whether links cluster under program-like anchors
- title/headings length and shape

**Semantic-lite features**
Not “deep intelligence,” just targeted signals:
- funding-like phrases
- deadline-like phrases
- eligibility-like phrases
- “apply,” “program guide,” “intake,” “eligible applicants”
- ministry/program naming patterns

**Graph-context features**
- parent node type
- anchor text that led here
- sibling page patterns

This is much better than plain bag-of-words because it uses **context + structure**, not just word counts.

### 2. Explicit scoring policy

Then define something like:
```
container_score
terminal_score
irrelevant_score
```
And make the decision from those scores.

That gives you:
- determinism
- debuggability
- tunability
- visible failure modes

### 3. “Unknown” must be a real class

This is very important.

Do **not** force every page into:
- container
- terminal
- irrelevant

You need:

`AMBIGUOUS`

Because that is where your future insights come from.

If you force certainty too early, the system lies.

## Why this is better than bag-of-words

Because bag-of-words is usually:
- flat
- contextless
- unable to distinguish listing vs terminal
- unable to use site structure
- unable to explain subtle decisions

What you want is closer to a **rule-based evidence model**.

That is a much more respectable first system.

## Why this is better than an LLM right now

Because it gives you:
- same input → same output
- explicit weights
- explicit reasons
- easy regression testing
- easier trust

This matches your taste much more closely.

And honestly, for navigator-phase work, that taste is probably correct.

## The real tradeoff

You are not choosing between:
- deterministic and good
- non-deterministic and bad

You are choosing between:

**Option A — deterministic but incomplete**

Good for architecture learning, trust, debugging

**Option B — flexible but fuzzy**

Good for rapid semantic coverage, worse for trust/control

For **Phase 1**, Option A is probably the right bias.

## What I would recommend

Use a layered classifier:

### Layer 1 — hard filters

Cheap, deterministic:
- domain allowlist
- duplicate detection
- file type
- depth limit
- obvious non-content pages

### Layer 2 — feature extraction

Extract explicit evidence:
- title
- headings
- URL tokens
- funding/deadline/eligibility markers
- outgoing link shape

### Layer 3 — scoring policy

Compute:
- `container_score`
- `terminal_score`
- `irrelevant_score`

### Layer 4 — decision policy

Emit:
- `EXPAND`
- `EXTRACT`
- `TERMINATE`
- `AMBIGUOUS`

That is clean.

## Important subtle point

Do not try to solve “is this definitely a grant page?”

That is too semantic for navigator.

Navigator should answer a weaker question:
> is this page structurally likely to be a grant-bearing terminal worth extracting?

That weaker question is much more compatible with deterministic logic.

## A strong hybrid you may actually like later

Not now, but later:
- runtime = deterministic feature/scoring system
- offline analysis = LLM used only to review ambiguous cases and help refine rules
- future = your own trained classifier once enough labeled data exists

That keeps runtime deterministic while still letting you use LLMs as a research assistant, not as production truth.

**Park — high future value**

## My recommendation in one sentence

Do not choose between LLM classification and bag-of-words.

Build a **deterministic, feature-based evidence scorer** with an explicit `AMBIGUOUS` state.

That is the most “you” solution here:
- principled
- inspectable
- evolvable
- not fake-smart

And honestly, it fits your architecture philosophy much better than either extreme.