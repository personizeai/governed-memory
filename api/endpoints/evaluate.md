# Evaluate — Schema Quality and Extraction Accuracy

**SDK method:** `client.memory.evaluate(opts)`
**Endpoint:** `POST /api/v1/evaluate`

Runs a three-phase pipeline — extraction replay, per-property analysis, and schema optimization — to measure how well a schema collection captures information from a given content sample. Returns per-property status, confidence scores, and targeted improvement instructions.

Used in experiments that need output quality scoring (E02, E08, E09) and schema lifecycle measurement (E05).

---

## SDK Setup

```typescript
import { Personize } from '@personize/sdk';
const client = new Personize({ secretKey: process.env.PERSONIZE_SECRET_KEY! });
```

---

## SDK Usage

```typescript
// Evaluate extraction quality for a content sample against a collection
const result = await client.memory.evaluate({
    content: 'Meeting transcript text...',
    collectionId: 'col_sales_contacts',
    recordId: 'rec_abc123',      // optional — scope to a specific record
    preset: 'sales',             // rubric preset: 'default' | 'sales' | 'support' | 'research'
});

// Per-property results
for (const prop of result.data.extractionAnalysis.propertyResults) {
    console.log({
        property: prop.propertyName,
        status: prop.status,          // 'extracted' | 'missed' | 'low_confidence' | 'inaccurate' | 'unavailable'
        confidence: prop.confidence,
        extractedValue: prop.extractedValue,
    });
}

// Schema refinements for underperforming properties
for (const refinement of result.data.schemaRefinements) {
    console.log({
        property: refinement.propertyName,
        reason: refinement.changeReason,
        before: refinement.before.description,
        after: refinement.after.description,
    });
}

// Overall quality score
console.log(result.data.evaluation.totalScore, '/', result.data.evaluation.maxScore);
```

---

## Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `content` | string | Yes | Source content to evaluate extraction against |
| `collectionId` | string | Yes | Schema collection ID to evaluate |
| `recordId` | string | No | Scope evaluation to a specific record |
| `preset` | string | No | Rubric preset — `'default'`, `'sales'`, `'support'`, `'research'` |
| `criteria` | object[] | No | Custom rubric criteria (overrides preset) |
| `stream` | boolean | No | Enable SSE streaming for real-time progress (default: `false`) |

### Rubric presets

| Preset | Criteria | Use for |
|---|---|---|
| `default` | General extraction quality | Any content type |
| `sales` | Personalization (30), Value Prop (25), CTA (20), Tone (25) | Sales emails and outbound |
| `support` | Issue resolution, clarity, empathy | Support ticket responses |
| `research` | Completeness, accuracy, sourcing | Research and analysis outputs |

---

## Response

```json
{
  "data": {
    "evaluation": {
      "totalScore": 82,
      "maxScore": 100,
      "criteria": [
        {
          "name": "Personalization",
          "score": 26,
          "maxScore": 30,
          "reason": "Strong use of entity-specific details from recalled memories."
        },
        {
          "name": "Value Proposition",
          "score": 19,
          "maxScore": 25,
          "reason": "Value prop present but not tied to prospect's stated pain points."
        }
      ]
    },
    "extractionAnalysis": {
      "propertyResults": [
        {
          "propertyId": "prop_tech_stack",
          "propertyName": "Technology Stack",
          "status": "extracted",
          "extractedValue": "AWS, React, PostgreSQL",
          "confidence": 0.92,
          "sourceSpan": "...they use AWS for infrastructure, React on the frontend..."
        },
        {
          "propertyId": "prop_budget",
          "propertyName": "Deal Value",
          "status": "missed",
          "improvementInstruction": "Update description to clarify that budget references include both explicit amounts and inferred ranges from deal context."
        },
        {
          "propertyId": "prop_urgency",
          "propertyName": "Urgency Level",
          "status": "low_confidence",
          "extractedValue": "High",
          "confidence": 0.58
        }
      ]
    },
    "schemaRefinements": [
      {
        "propertyId": "prop_budget",
        "propertyName": "Deal Value",
        "changeReason": "Added extraction hints for indirect budget references and range indicators.",
        "before": {
          "description": "How much money"
        },
        "after": {
          "description": "The estimated or stated monetary value of the deal in USD. Extract explicit amounts ('$450K', '450,000 dollars') and convert shorthand: K = thousands, M = millions. If a range is given, use the midpoint. Also capture indirect references ('mid-six-figure range', 'budget approved for Q3')."
        }
      }
    ]
  }
}
```

### Property status values

| Status | Meaning |
|---|---|
| `extracted` | Property found and extracted with sufficient confidence |
| `missed` | Content contained the information but it was not extracted |
| `low_confidence` | Extracted but confidence score is below threshold |
| `inaccurate` | Extracted value does not match the source content |
| `unavailable` | Property not mentioned in the content (expected) |

---

## Three-Phase Pipeline

1. **Extraction Replay** — Re-runs the dual extraction pipeline on `content` against the collection schema, producing property values with confidence scores.

2. **Per-Property Analysis** — Evaluates each property for coverage (was data present but missed?), accuracy (does the extraction match the source?), and gaps (what information has no matching property?).

3. **Schema Optimization** — For properties classified as `missed`, `low_confidence`, or `inaccurate`, generates targeted improvement instructions and revised property descriptions in parallel.

---

## Streaming (SSE)

When `stream: true`, progress events are emitted as Server-Sent Events:

```
event: phase
data: {"phase": "extraction", "status": "started"}

event: progress
data: {"phase": "extraction", "propertiesProcessed": 5, "total": 14}

event: phase
data: {"phase": "analysis", "status": "started"}

event: result
data: {"evaluation": {...}, "extractionAnalysis": {...}, "schemaRefinements": [...]}
```

Useful for showing live progress in a UI while processing long transcripts.

---

## For Experiment Runners

### E05 — Schema lifecycle (before/after refinement)

```typescript
const transcripts = fs.readdirSync('synthetic datasets/transcripts')
    .filter(f => f.endsWith('.txt'))
    .slice(0, 10);  // use 10 samples for before/after comparison

const beforeResults = [];
const afterResults = [];

// BEFORE: run with Collection B (6 vague properties)
for (const file of transcripts) {
    const content = fs.readFileSync(`synthetic datasets/transcripts/${file}`, 'utf-8');
    const result = await client.memory.evaluate({
        content,
        collectionId: COLLECTION_B_ID,  // vague schema
        preset: 'default',
    });

    for (const prop of result.data.extractionAnalysis.propertyResults) {
        beforeResults.push({
            file,
            property: prop.propertyName,
            status: prop.status,
            confidence: prop.confidence ?? null,
        });
    }
}

// Apply refinement suggestions from the evaluate response, then:

// AFTER: re-run the same 10 transcripts with the refined collection
for (const file of transcripts) {
    const content = fs.readFileSync(`synthetic datasets/transcripts/${file}`, 'utf-8');
    const result = await client.memory.evaluate({
        content,
        collectionId: COLLECTION_B_REFINED_ID,  // same collection after applying schemaRefinements
        preset: 'default',
    });

    for (const prop of result.data.extractionAnalysis.propertyResults) {
        afterResults.push({
            file,
            property: prop.propertyName,
            status: prop.status,
            confidence: prop.confidence ?? null,
        });
    }
}

// Compare per-property confidence before vs after
```

### E08 — End-to-end (4-condition ablation)

```typescript
const conditions = ['no_memory', 'raw_memory', 'open_set_governance', 'full_governed'];

for (const prospect of prospects) {  // 10 entities, each memorized with 15-20 memories
    for (const condition of conditions) {
        // Assemble context per condition (see experiments.md E08 protocol)
        const context = await assembleContext(prospect.email, condition);

        // Generate the follow-up email
        const email = await llm.generate(`${context}\n\nWrite a follow-up email.`);

        // Evaluate the output using the sales rubric
        const evaluation = await client.memory.evaluate({
            content: email,
            collectionId: COLLECTION_A_ID,
            preset: 'sales',
        });

        console.log({
            prospect: prospect.email,
            condition,
            totalScore: evaluation.data.evaluation.totalScore,
            criteria: evaluation.data.evaluation.criteria.map(c => ({
                name: c.name,
                score: c.score,
                max: c.maxScore,
            })),
        });
    }
}
```
