# Governed Memory: Experiment Datasets

Synthetic datasets and experiment protocols for the paper **"Governed Memory: A Shared Layer for Accuracy and Compliance Across Agentic Workflows"**.

All datasets are fully synthetic. No real customer data, PII, or proprietary information is included.

---

## What Is This?

This repository contains:

1. **Synthetic datasets** — business content generated specifically to stress-test governed memory capabilities, each with paired ground truth files for objective evaluation
2. **Experiment protocols** — reproducible measurement procedures for each capability claim in the paper
3. **Schema collections** — the property extraction schemas used across experiments
4. **API reference** — the endpoint documentation needed to reproduce experiments with the Governed Memory API

---

## Experiment Status

| # | Experiment | Status | Dataset |
|---|---|---|---|
| E01 | Extraction Quality Across Content Types | **Results included** | `synthetic datasets/{transcripts,emails,chats,documents,call_notes}/` |
| E02 | Memory Density vs. Output Quality | **Results included** | — |
| E03 | Governance Routing Precision | **Results included** | `synthetic datasets/governance_pairs/` |
| E04 | Progressive Delivery Token Savings | **Results included** | — |
| E05 | Schema Lifecycle — Before and After Refinement | **Results included** | `experiment-collections-import.json` |
| E06 | Deduplication Effectiveness at Scale | **Results included** | `synthetic datasets/multi_source/` |
| E07 | Recall Speed, Relevance, and Stage Breakdown | **Results included** | `synthetic datasets/recall_queries/` |
| E08 | End-to-End Workflow Quality (4-Condition Ablation) | **Results included** | — |
| E09 | Quality Gates Ablation | **Results included** | Reuses E01 data |
| E10 | Reflection Rounds Ablation | **Results included** | `synthetic datasets/multi_source/` + `recall_queries/` |
| E11 | Entity Isolation Validation | **Results included** | `synthetic datasets/entity_isolation/` |
| E12 | Dual Memory Complementarity | **Results included** | Reuses E01 data |
| E13 | Governance Variable Authoring Quality Impact | **Results included** | `synthetic datasets/governance_pairs/` |
| E14 | Temporal Conflict Resolution | **Results included** | `synthetic datasets/conflict_pairs/` |
| E15 | Governance Constraint Enforcement Under Adversarial Pressure | **Results included** | `synthetic datasets/adversarial_governance/` |

---

## Folder Structure

```
experiments datasets/
├── README.md                              — This file
├── experiments.md                         — Full experiment protocols (E01–E15)
├── experiment-data-guide.md               — How the synthetic datasets were designed
├── experiment-collections-import.json     — Schema collections used in experiments
│
├── api/                                   — API reference for reproducing experiments
│   ├── authentication.md
│   └── endpoints/
│       ├── memorize.md                    — POST /api/v1/memorize
│       ├── recall.md                      — POST /api/v1/smart-recall
│       ├── evaluate.md                    — POST /api/v1/evaluate
│       └── smart_guidelines.md            — POST /api/v1/ai/smart-guidelines
│
├── results/
│   ├── e01_extraction_quality/            — Per-sample extraction results (250 records)
│   ├── e02_memory_density/                — Output quality by memory density level
│   ├── e03_routing_precision/             — Governance routing precision measurements
│   ├── e04_progressive_delivery/          — Token savings per step
│   ├── e05_schema_lifecycle/              — Before/after schema refinement comparison
│   ├── e06_dedup_effectiveness/           — Deduplication rate and false positive counts
│   ├── e07_recall_speed/                  — Latency and relevance by reflection round
│   ├── e08_end_to_end/                    — 4-condition ablation scores
│   ├── e09_quality_gates_ablation/        — Quality gate impact on output
│   ├── e10_reflection_ablation/           — Recall recall and latency by round count
│   ├── e11_entity_isolation/              — Cross-entity leak detection results
│   ├── e12_dual_memory_complementarity/   — Open-set vs. schema-only vs. combined
│   ├── e13_authoring_quality/             — Well-authored vs. poorly-authored variable routing
│   ├── e14_conflict_resolution/           — Temporal conflict resolution accuracy
│   └── e15_adversarial_governance/        — Constraint enforcement under adversarial input
│
└── synthetic datasets/
    ├── transcripts/        — 50 meeting transcripts (+ ground truth sidecars)
    ├── emails/             — 10 email threads (+ ground truth)
    ├── chats/              — 10 Slack-style chat logs (+ ground truth)
    ├── documents/          — 10 account wiki pages (+ ground truth)
    ├── call_notes/         — 10 post-call notes (+ ground truth)
    ├── multi_source/       — 1 entity (Sarah Chen, TechFlow Inc) across 5 sources
    ├── recall_queries/     — 10 recall queries with expected answers
    ├── entity_isolation/   — 3 semantically overlapping entities (for E11)
    ├── governance_pairs/   — Well-authored vs. poorly-authored variable pairs (for E13)
    ├── conflict_pairs/     — 30 stale/fresh fact pairs (for E14)
    └── adversarial_governance/ — 50 adversarial scenarios (for E15)
```

---

## Dataset Summary

### Primary dataset: 250 labeled content samples (E01)

| Content Type | Samples | Avg Words | Ground Truth |
|---|---|---|---|
| Meeting transcripts | 50 | ~2,500 | 8–14 facts, 7–10 properties per sample |
| Email threads | 50 | ~1,600 | 8–10 facts, 6–8 properties per sample |
| Chat logs | 50 | ~1,000 | 6–8 facts, 4–6 properties per sample |
| Account documents | 50 | ~2,000 | 8–10 facts, 7–9 properties per sample |
| Call notes | 50 | ~500 | 5–7 facts, 5–6 properties per sample |

Each sample has a `.ground_truth.json` sidecar with expected facts, expected property values, and a count of planted defects (unresolved pronouns, relative time references, near-duplicate facts).

### Ground truth schema

```json
{
  "sample_id": "transcript_001",
  "company": "TechFlow Inc",
  "industry": "B2B SaaS",
  "word_count": 2847,
  "expected_facts": [
    {"text": "TechFlow is migrating from on-prem Oracle to AWS in Q2 2026", "type": "explicit"},
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

### Multi-source entity (E06, E10)

`synthetic datasets/multi_source/` contains 5 sources about a single entity — Sarah Chen, VP Engineering at TechFlow Inc — spanning a discovery call, follow-up call, email thread, CRM notes, and LinkedIn research. Designed with known overlap:
- 36 unique facts total across sources
- 10 facts that appear in 2+ sources (dedup targets)
- 4 facts that are similar but semantically distinct (should NOT be deduped)

### Conflict pairs (E14)

30 pairs, each with a `stale.txt` and `fresh.txt` about the same entity — the fresh version supersedes the stale on a specific factual claim. Categories: database migration, cloud provider, headcount, leadership, deal stage. `ground_truth.json` specifies the expected query and expected answer for each pair.

### Adversarial governance scenarios (E15)

50 scenarios, each with a governance constraint (in `scenarios.json`) and an adversarial task designed to trigger a policy violation. Categories: competitor pricing, roadmap confidentiality, customer data privacy, discount authorization, legal claims.

---

## How to Reproduce Experiments

### Requirements

- Access to the Governed Memory API (see `api/authentication.md`)
- An API key with read/write access to a workspace

### Basic workflow for E01 (extraction quality)

Install the SDK first:

```bash
npm install @personize/sdk
```

```typescript
import { Personize } from '@personize/sdk';
import * as fs from 'fs';
import * as path from 'path';

const client = new Personize({ secretKey: process.env.PERSONIZE_SECRET_KEY! });

const results = [];
const transcriptDir = 'synthetic datasets/transcripts';

for (const file of fs.readdirSync(transcriptDir).filter(f => f.endsWith('.txt'))) {
    const sampleId = file.replace('.txt', '');
    const content = fs.readFileSync(path.join(transcriptDir, file), 'utf-8');
    const groundTruth = JSON.parse(
        fs.readFileSync(path.join(transcriptDir, `${sampleId}.ground_truth.json`), 'utf-8')
    );

    const result = await client.memory.memorize({
        content,
        email: groundTruth.entity_email,
        enhanced: true,
        tags: ['experiment:e01', 'content-type:transcript'],
        actionId: 'your-collection-id',
    });

    const factsExtracted = result.data.memories.length;
    const propsExtracted = result.data.properties.length;
    const factsExpected = groundTruth.expected_facts.length;
    const propsExpected = Object.keys(groundTruth.expected_properties).length;

    results.push({
        sample_id: groundTruth.sample_id,
        content_type: 'transcript',
        memories_extracted: factsExtracted,
        properties_extracted: propsExtracted,
        expected_fact_count: factsExpected,
        expected_property_count: propsExpected,
        fact_recall: Math.min(factsExtracted, factsExpected) / factsExpected,
        property_recall: Math.min(propsExtracted, propsExpected) / propsExpected,
        quality_gates: result.data.qualityGates ?? {},
        duration_ms: result.data.stats?.durationMs,
    });
}

fs.writeFileSync('results/e01_my_run.json', JSON.stringify(results, null, 2));
console.log(`Done. ${results.length} samples processed.`);
```

See `experiments.md` for the full protocol for each experiment, including scoring methodology, expected API fields, and how to interpret results.

### SDK methods used

| Experiment(s) | SDK Method | Endpoint |
|---|---|---|
| E01, E05, E06, E09, E11, E12 | `client.memory.memorize()` | `POST /api/v1/memorize` |
| E07, E10, E11 | `client.memory.smartRecall()` | `POST /api/v1/smart-recall` |
| E06, E12 | `client.memory.smartDigest()` | `POST /api/v1/smart-memory-digest` |
| E02, E08, E09 | `client.memory.evaluate()` | `POST /api/v1/evaluate` |
| E03, E04, E08, E13, E15 | `client.ai.smartGuidelines()` | `POST /api/v1/ai/smart-guidelines` |

---

## Schema Collections

`experiment-collections-import.json` contains three schema collections used across experiments:

- **Collection A** — Sales Contacts (14 properties, well-defined descriptions) — primary schema for E01, E05, E06, E07, E08, E09, E10, E12
- **Collection B** — Sales Contacts (same 14 properties, 6 with deliberately vague descriptions) — used in E05 as the "before" condition
- **Collection C** — Support Tickets (8 properties) — used in E01 for content type diversity

Import via your workspace UI or API to use the same schemas as the paper.

---

## Synthetic Data Generation

All content was generated using the methodology in `experiment-data-guide.md`. The key principle: each sample has a known ground truth so extraction recall can be measured objectively. Content is realistic B2B business conversation across 8 industries (SaaS, Fintech, Healthcare, E-commerce, Manufacturing, Logistics, Media, Education).

To generate additional samples using the same methodology, see the generation prompts in `experiment-data-guide.md`.

---

## Citation

[Paper citation will be added upon publication]

---

## License

The synthetic datasets (content + ground truth files) are released for research use. The experiment protocols and API documentation are copyright the authors.
