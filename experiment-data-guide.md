# Experiment Data Preparation Guide

**Purpose:** Exactly what to create in the UI, what to generate synthetically, and what criteria matter.

---

## Part 1: Schema Collections to Create

You need 3 collections. Create them via the UI.

---

### Collection A: Sales Contacts (Well-Defined) --- Primary Experiment Schema

**Based on your existing `sales-contact-schema.json` but expanded to 14 properties.** This is the "production-quality" schema used in most experiments. Every description is specific, with examples and edge case guidance.

**Why 14 properties:** Having 14 means the system must actively choose which properties are relevant to each content chunk --- too few and it extracts everything without selection signal, too many and most get filtered with little interesting data. 14 is a practical sweet spot for sales contact schemas.

**Property design criteria:**

| Criterion | Good (Include) | Bad (Exclude) |
|---|---|---|
| **Type diversity** | Mix of text, number, date, options, array, boolean | All text or all options |
| **Extraction difficulty** | Range from easy (job title) to hard (sentiment, implied budget) | All trivially obvious |
| **Overlap potential** | Some properties overlap with open-set facts (pain points appear in both) | Fully disjoint from likely facts |
| **Append vs replace** | 2-3 append properties (pain_points, competitors, next_steps) | All replace |
| **Presence frequency** | Some appear in every transcript, some rarely | All always present |

**The 14 properties:**

```
ALWAYS PRESENT (appear in nearly every sales conversation):
1.  job_title           (text)     - Easy extraction
2.  company_name        (text)     - Easy extraction
3.  company_size        (options)  - Sometimes explicit, sometimes inferred
4.  technology_stack    (text)     - Medium difficulty, requires distinguishing tech vs products
5.  deal_stage          (options)  - Must be inferred from context, not stated directly

FREQUENTLY PRESENT (appear in ~60-80% of conversations):
6.  pain_points         (array, append) - Medium difficulty, some explicit, some implied
7.  deal_value          (number)   - Sometimes stated, sometimes a range, sometimes missing
8.  decision_timeline   (date)     - Tests temporal anchoring (relative → absolute)
9.  competitors_mentioned (array, append) - Must distinguish actual competitors from passing mentions
10. decision_maker      (boolean)  - Must infer from authority signals, not direct statements

SOMETIMES PRESENT (appear in ~30-50% of conversations):
11. next_steps          (text)     - Tests extraction of action items
12. buying_process      (text)     - Complex extraction: procurement steps, approvals needed
13. current_solution    (text)     - What they use today (the thing being replaced)
14. urgency_level       (options: Low/Medium/High/Critical) - Must infer from language intensity
```

**Description quality rules for each property:**

1. **State what TO extract** --- not just "the company's technology" but "programming languages, frameworks, cloud platforms, and databases"
2. **State what NOT to extract** --- "Focus on technical stack decisions rather than product or SaaS tool usage"
3. **Give 2-3 examples** --- "e.g., 'Python, React, AWS, PostgreSQL'"
4. **Handle ambiguity** --- "If a range is given, use the midpoint"
5. **Handle absence** --- implicitly, by allowing null (don't force extraction of things not mentioned)

---

### Collection B: Sales Contacts (Intentionally Vague) --- For Experiment 5 Only

**Same 14 properties but with deliberately poor descriptions.** This is the "before" in the schema lifecycle experiment.

Make 6 properties vague, keep 8 well-defined (control group):

```
VAGUE (these should underperform):
1.  technology_stack    → "The company's technology"
2.  pain_points         → "Problems they have"
3.  deal_value          → "How much money"
4.  decision_timeline   → "When they decide"
5.  buying_process      → "How they buy things"
6.  urgency_level       → "How urgent it is"

WELL-DEFINED (control group, same descriptions as Collection A):
7.  job_title           → (same rich description)
8.  company_name        → (same)
9.  company_size        → (same)
10. deal_stage          → (same)
11. competitors_mentioned → (same)
12. decision_maker      → (same)
13. next_steps          → (same)
14. current_solution    → (same)
```

**What this tests:** The evaluate pipeline should flag properties 1-6 as underperforming and generate targeted refinements. Properties 7-14 should show minimal change. This proves the pipeline identifies WHICH properties need work --- not a blanket "improve everything."

---

### Collection C: Support Tickets --- For Content Type Diversity

**Use your existing `support-ticket-schema.json` as-is (8 properties).** This gets used in Experiment 1 (content type comparison) with support-related content. Shows the system works beyond just sales.

---

## Part 2: Governance Variables to Create

You need 25 variables for Experiment 3 (routing precision). Create them via the UI using AI-assisted authoring.

**Organization principle:** Each variable should be **focused on one concern** (not a monolithic policy doc). This is how the routing system is designed to work --- small, composable documents.

### Must-Have Variables (12) --- Used Across Multiple Experiments

```
BRAND & MESSAGING (3 variables):
1.  Brand Voice Guidelines
    - Tags: Brand, Guideline
    - Content: Tone, email structure, audience-specific formality
    - Based on your existing template, expanded with Personize-specific voice
    - ~800 words

2.  Product Positioning & Messaging
    - Tags: Brand, Marketing
    - Content: Value propositions, competitive framing, words to use/avoid
    - ~600 words

3.  Social Media Content Policy
    - Tags: Brand, Policy
    - Content: Platform-specific rules, attribution, approval process
    - ~400 words

SALES (3 variables):
4.  Sales Playbook: Discovery Call
    - Tags: Sales, Skill
    - Content: Questions to ask, qualification criteria, red flags
    - ~700 words

5.  ICP Definition: Mid-Market SaaS
    - Tags: Sales, Guideline
    - Based on your existing template
    - ~600 words

6.  Pricing & Negotiation Guidelines
    - Tags: Sales, Policy
    - Content: Discount authority, deal terms, competitive pricing responses
    - ~500 words

COMPLIANCE (3 variables):
7.  Data Handling & Privacy Policy
    - Tags: Policy, Guideline
    - Content: PII rules, GDPR requirements, data retention, consent
    - ~600 words

8.  Regulatory Compliance: Financial Services
    - Tags: Policy, Guideline
    - Content: Industry-specific rules for fintech/banking customers
    - ~500 words

9.  Regulatory Compliance: Healthcare
    - Tags: Policy, Guideline
    - Content: HIPAA-adjacent rules for health-related customers
    - ~500 words

PRODUCT (3 variables):
10. Product Knowledge: Core Platform
    - Tags: Product, Skill
    - Content: Feature descriptions, technical specs, architecture overview
    - ~800 words

11. Product Knowledge: MCP Integration
    - Tags: Product, Tool
    - Content: MCP setup, supported platforms, configuration patterns
    - ~600 words

12. Product Knowledge: Evaluation Framework
    - Tags: Product, Skill
    - Content: How evaluate works, rubric presets, schema refinement
    - ~500 words
```

### Additional Variables (13) --- Needed for Routing Experiment Coverage

```
SUPPORT (3 variables):
13. Support Escalation Policy        - Tags: Support, Policy
14. Support SLA by Account Tier      - Tags: Support, Guideline
15. Technical Troubleshooting Guide  - Tags: Support, Skill

RESEARCH (3 variables):
16. Prospect Research Protocol       - Tags: Sales, Skill
17. Competitive Analysis Framework   - Tags: Marketing, Skill
18. Market Research Guidelines       - Tags: Marketing, Guideline

OPERATIONS (2 variables):
19. Onboarding Workflow: Builder Track     - Tags: Customer, Guideline
20. Onboarding Workflow: Design Partner    - Tags: Customer, Guideline

INTERNAL (3 variables):
21. Engineering Style Guide          - Tags: Product, Guideline
22. HR: Interview Process            - Tags: Policy
23. Meeting Notes Template           - Tags: Guideline

NICHE / DISTRACTOR (2 variables):
24. Event Planning: Conference Booth  - Tags: Marketing, Guideline
25. Office Supplies Policy            - Tags: Policy
```

**Why 25 specifically:** The routing system's two-stage design first semantically pre-filters candidates, then uses an LLM to make the final selection. With 25 variables:
- Pre-filter should reduce to roughly 6-12 candidates (50-75% reduction)
- LLM stage selects 1-3 from the reduced set
- Variables 24-25 are intentional distractors --- they should NEVER be selected for any task. If they are, the routing has a false positive.

**Tags are critical.** The system does hard tag filtering first. If your task context includes a `Sales` tag, only variables with `Sales` in their tags pass the first gate. Design your tags so:
- Every variable has 1-3 tags
- No two variables have the exact same tag combination
- Distractors have tags that don't overlap with test tasks

### Variable Length Guidelines

| Length | Use For | Example |
|---|---|---|
| ~300-500 words | Focused policies (one topic, clear rules) | Escalation policy |
| ~500-800 words | Skill guides (procedures, steps, examples) | Sales playbook |
| ~800-1200 words | Comprehensive guidelines (multiple sections) | Brand voice |

**Don't exceed 1,200 words per variable.** Longer variables get section-level extraction (the router picks specific markdown headings instead of the full doc). That's a feature, but it adds a variable to the experiment. Keep variables focused so you're testing routing precision, not section extraction.

---

## Part 3: Synthetic Datasets to Generate

### Dataset 1: Meeting Transcripts (50 samples)

**Used in:** Experiments 1, 2, 5, 6, 8

**Generate with an LLM using this prompt structure:**

```
Generate a realistic {duration}-minute B2B sales meeting transcript.

PARTICIPANTS:
- Alex Morgan (Account Executive at TechNova)
- {prospect_name} ({prospect_title} at {company_name})
{optional_third_participant}

COMPANY CONTEXT:
- Industry: {industry}
- Size: {size}
- Current situation: {situation_brief}

FACTS TO EMBED (mention naturally in dialogue, do NOT label them):
1. {fact_1}
2. {fact_2}
... (8-12 facts)

PROPERTY VALUES TO MENTION:
- Job Title: {title}
- Technology Stack: {stack}
- Company Size: {size}
- Deal Value: {value}
- Pain Points: {pain_points}
- Decision Timeline: {timeline}
... (7-10 properties)

QUALITY TEST PATTERNS (embed naturally):
- {n} sentences where someone uses a pronoun without prior resolution
  Example: Start a paragraph with "He mentioned that..." without
  establishing who "he" is in that paragraph
- {n} relative time references that should be anchored
  Example: "next quarter", "a few weeks ago", "by end of month"
- {n} facts that repeat the same information in slightly different words
  Example: "Budget is around 400K" and later "We've allocated roughly
  four hundred thousand for this"

STYLE:
- Natural conversation with topic changes, interruptions, and filler words
- Both speakers should contribute substantially
- Include at least one digression or small talk moment
- {word_count} words approximately

OUTPUT: The transcript only. No metadata, labels, or annotations.
```

**Vary across the 50 samples:**

| Parameter | Variation Range |
|---|---|
| Duration | 20, 30, 45, 60 minutes |
| Word count | 1,500 - 4,000 words |
| Industry | SaaS, Fintech, Healthcare, E-commerce, Manufacturing, Logistics, Media, Education |
| Company size | 20-50, 50-200, 200-1000, 1000+ |
| Deal value | $50K - $2M |
| Pronoun issues | 2-5 per sample |
| Temporal issues | 1-4 per sample |
| Duplicate facts | 1-3 per sample |
| Fact count | 8-14 per sample |
| Property coverage | 60-100% of schema properties mentioned |
| Third participant | 30% of samples include a third person (makes coreference harder) |

**Ground truth file:** For each transcript, save a JSON sidecar:

```json
{
  "sample_id": "transcript_001",
  "company": "TechFlow Inc",
  "word_count": 2847,
  "expected_facts": [
    {"text": "TechFlow is migrating from on-prem to AWS in Q2 2026", "type": "explicit"},
    {"text": "Sarah is concerned about data gravity with their current Oracle setup", "type": "implied"}
  ],
  "expected_properties": {
    "job_title": "VP of Engineering",
    "technology_stack": "Oracle, Java, migrating to AWS",
    "deal_value": 450000,
    "decision_timeline": "2026-06-30"
  },
  "planted_issues": {
    "pronoun_issues": 3,
    "temporal_issues": 2,
    "duplicate_facts": 2
  }
}
```

---

### Dataset 2: Email Threads (10 samples)

**Used in:** Experiment 1

**Structure:** 4-8 emails per thread, ~1,200-2,000 words total.

```
Generate a realistic email thread between a sales team and a prospect.

THREAD STRUCTURE:
- Email 1: Initial outreach from sales rep
- Email 2: Prospect's reply (asks a question)
- Email 3: Sales rep's answer with proposal
- Email 4: Prospect forwards to colleague (introduces new person)
- Email 5: Colleague asks technical question
- Email 6: Sales rep's technical response
{optional: Email 7-8 for longer threads}

INCLUDE:
- Email headers (From, To, Subject, Date) for each email
- Forwarded email markers ("---------- Forwarded message ----------")
- Reply chains ("> " quoted text from previous emails)
- Signature blocks with title, phone, company

FACTS TO EMBED: {8-10 facts across the thread}
PROPERTY VALUES: {6-8 properties}
QUALITY PATTERNS: {2-3 pronoun issues, 1-2 temporal issues}
```

**Why emails are interesting:** They have a unique structure (headers, signatures, quoted replies) that tests chunking differently from transcripts. Forwarded emails introduce new entities mid-thread (coreference challenge). Signature blocks contain property values (title, company) that should be extracted even though they're metadata, not conversation.

---

### Dataset 3: Chat Logs (10 samples)

**Used in:** Experiment 1

**Structure:** Slack/Teams style, 30-60 messages, ~800-1,200 words.

```
Generate a realistic Slack channel conversation about a customer deal.

FORMAT:
[timestamp] username: message

CHARACTERISTICS:
- Short messages (1-3 sentences each)
- Abbreviations and informal language ("tbh", "imo", "lmk")
- Multiple participants (3-4 people)
- Thread replies mixed with main channel messages
- Emoji reactions described as text: "[thumbs up from @sarah]"
- Shared links: "[shared link: Acme Corp pricing page]"
- At least one "@channel" or "@here" ping

FACTS TO EMBED: {6-8 facts across messages}
PROPERTY VALUES: {4-6 properties}
QUALITY PATTERNS:
- {3-5 pronoun issues} (chat naturally has MORE pronoun issues)
- {2-3 temporal issues} (chat naturally has MORE relative times)
- {1-2 duplicate facts} (people repeat things in chat)
```

**Why chat logs matter:** This is the content type where quality gates SHOULD score lowest. Chat is informal, fragmented, full of pronouns and relative times. If your quality gates correctly flag chat as lower quality than transcripts, that's a meaningful finding. It proves the gates are sensitive to content type, not just producing a fixed score.

---

### Dataset 4: Documents/Wiki Pages (10 samples)

**Used in:** Experiment 1

**Structure:** Structured pages with headings, bullet points, tables, ~1,500-2,500 words.

```
Generate a realistic internal wiki page about a customer account.

STRUCTURE:
# {Company Name} - Account Overview

## Company Background
(paragraph with company description)

## Key Contacts
| Name | Title | Role in Deal |
...

## Meeting History
### Meeting 1: Discovery Call (Date)
...

## Technical Requirements
(bullet points)

## Deal Status
(paragraph with current status, timeline, blockers)

FACTS TO EMBED: {8-10 facts}
PROPERTY VALUES: {7-9 properties}
QUALITY PATTERNS:
- {1-2 pronoun issues} (documents have FEWER --- more formal)
- {1-2 temporal issues}
- {1 duplicate fact}
```

**Why documents matter:** They're the best-case content type. Structured, formal, few pronoun issues. Quality gates should score highest here. This provides the upper bound in your content type comparison.

---

### Dataset 5: Call Notes (10 samples)

**Used in:** Experiment 1

**Structure:** Terse summary format, ~400-700 words.

```
Generate realistic sales call notes written by a rep after a call.

FORMAT:
Call with: {name}, {title} at {company}
Date: {date}
Duration: {duration}

Key Points:
- {bullet point}
- {bullet point}
...

Action Items:
- [ ] {action item with owner}
...

Notes:
{1-2 paragraphs of additional context}

CHARACTERISTICS:
- Abbreviations and shorthand ("mtg", "f/u", "re:", "w/")
- Incomplete sentences
- Mixed first and third person
- Some bullet points are single words or phrases

FACTS TO EMBED: {5-7 facts}
PROPERTY VALUES: {5-6 properties}
QUALITY PATTERNS:
- {2-3 pronoun issues} (notes are terse, context-dependent)
- {1-2 temporal issues}
- {0-1 duplicate facts}
```

**Why call notes matter:** They're the "worst written" content type. Fragments, abbreviations, assumed context. Tests whether extraction can handle low-quality input gracefully.

---

### Dataset 6: Multi-Source Entity Profile (1 detailed entity, 5 sources)

**Used in:** Experiment 6 (dedup) and Experiment 7 (recall at scale)

This is NOT generated by prompt. You manually design it to have precise overlap:

```
Entity: Sarah Chen, VP Engineering at TechFlow Inc

Source 1 - Discovery Call Transcript (12 unique facts, 0 duplicates)
  All facts are new. This is the first memorization.

Source 2 - Follow-up Call Transcript (10 unique facts, 3 overlap with Source 1)
  Overlapping facts:
  - "TechFlow uses AWS" (same as Source 1: "Their infrastructure is on AWS")
  - "Budget is around $400K" (same as Source 1: "Allocated roughly four hundred thousand")
  - "Q2 timeline" (same as Source 1: "Targeting Q2 for migration")

Source 3 - Email Thread (8 unique facts, 2 overlap with Sources 1-2)
  Overlapping facts:
  - References AWS migration (same as Source 1+2)
  - Mentions $400K budget in writing (same as Source 1+2)

Source 4 - CRM Notes by Sales Rep (6 unique facts, 4 overlap with Sources 1-3)
  Heavy overlap --- rep summarized what was already in transcripts
  Overlapping: AWS, budget, Q2 timeline, team size

Source 5 - LinkedIn + Company Research (7 unique facts, 1 overlap)
  Mostly new info (company background, funding, public tech stack)
  Overlap: company size matches CRM notes

TOTALS:
  40 unique facts across all sources
  10 facts that appear in 2+ sources (should be deduped)
  4 facts that are SIMILAR but semantically distinct (should NOT be deduped):
    - "AWS migration planned for Q2" vs "AWS migration facing procurement delays"
    - "Team of 50 engineers" vs "Engineering team grew to 65 after recent hiring"
```

**Why you design this manually:** You need to KNOW the exact dedup ground truth. If you let an LLM generate it, you can't precisely control similarity levels. Design the overlapping facts yourself, then use an LLM to flesh them out into full transcripts/emails/notes.

---

### Dataset 7: Recall Query Set (10 queries with expected answers)

**Used in:** Experiment 7

Design queries at varying specificity. For each, note the expected relevant memories.

```
BROAD QUERIES:
1. "What do we know about Sarah Chen at TechFlow?"
   → Expected: all memories (tests comprehensive recall)

2. "Summarize our relationship with TechFlow"
   → Expected: meeting history + deal status memories

SPECIFIC QUERIES:
3. "What is TechFlow's technology stack?"
   → Expected: 2-3 memories mentioning AWS, Java, PostgreSQL

4. "What's the deal value and expected close date?"
   → Expected: 1-2 memories with $400K and Q2 2026

MULTI-TOPIC QUERIES:
5. "What are their pain points and who makes the buying decision?"
   → Expected: pain point memories + decision maker memories

6. "What competitors did they mention and what's their current solution?"
   → Expected: competitor memories + current solution memories

TEMPORAL QUERIES:
7. "What happened in our most recent conversation?"
   → Expected: memories from the latest source only

8. "How has their position evolved across our conversations?"
   → Expected: memories from multiple sources showing progression

RELATIONAL QUERIES:
9. "Who else at TechFlow should we be talking to?"
   → Expected: memories mentioning other stakeholders

10. "What technical requirements have they specified?"
    → Expected: memories about architecture, integrations, specs
```

---

## Part 4: What NOT to Include

### In Schemas

**Don't include properties that:**
- Can't be extracted from conversational content (e.g., "CRM Record ID", "Salesforce Opportunity ID")
- Are always the same across all records (e.g., "Product Being Sold" if you only sell one product)
- Require real-time data (e.g., "Current Stock Price", "LinkedIn Follower Count")
- Are pure metadata (e.g., "Date of Last Contact" --- this comes from the system, not extraction)

### In Governance Variables

**Don't include variables that:**
- Are longer than 1,200 words (forces section extraction, adds complexity)
- Have identical tags to another variable (makes routing indistinguishable)
- Are too generic to ever be "critical" ("General Business Principles")
- Overlap significantly with another variable's content (the router can't distinguish them)

### In Synthetic Data

**Don't generate content that:**
- Is unrealistically clean (no filler words, perfect grammar, every sentence contains a fact)
- Contains labeled facts ("FACT: The budget is $400K" --- embed it naturally)
- Has inconsistent entity names (pick one spelling and stick with it within a source)
- Is too short for meaningful extraction (<300 words produces too few facts to measure)
- Contains information that contradicts other sources about the same entity (unless testing conflict resolution, which is a separate experiment)

---

## Part 5: Generation Order and Effort

| Step | What | Method | Time |
|---|---|---|---|
| 1 | Create Collection A (14 props, well-defined) | UI + AI Assist | 1 hour |
| 2 | Create Collection B (14 props, 6 vague) | Clone A, degrade 6 descriptions | 30 min |
| 3 | Verify Collection C exists (support tickets) | Check UI | 5 min |
| 4 | Create 12 must-have governance variables | UI + AI Assist authoring | 2-3 hours |
| 5 | Create 13 additional governance variables | UI + AI Assist authoring | 2-3 hours |
| 6 | Generate 50 meeting transcripts + ground truth | LLM API batch | 3-4 hours |
| 7 | Generate 10 email threads + ground truth | LLM API batch | 1 hour |
| 8 | Generate 10 chat logs + ground truth | LLM API batch | 1 hour |
| 9 | Generate 10 documents + ground truth | LLM API batch | 1 hour |
| 10 | Generate 10 call notes + ground truth | LLM API batch | 1 hour |
| 11 | Design multi-source entity (manual overlap design) | Manual + LLM flesh-out | 2 hours |
| 12 | Design recall query set | Manual | 30 min |

**Total: ~2-3 days of focused work.**

Store everything in `Docs/experiments/data/` with subfolders per dataset.

---

## Part 6: Validation Against Production

After running experiments, compare your synthetic data results against known production metrics:

| Metric | Production Benchmark | Synthetic Should Be Within |
|---|---|---|
| Facts per record | 8.2 avg | 6-11 |
| Properties per record | 6.7 avg | 5-9 |
| Dedup rate | ~12% | 8-18% |
| Coreference score | >0.90 | 0.85-0.95 |
| Self-containment score | >0.85 | 0.80-0.92 |
| Temporal anchoring | >0.90 | 0.82-0.95 |

If synthetic results are wildly outside these ranges, either the synthetic data isn't realistic enough or the benchmarks need updating. Either finding is valuable for the paper.
