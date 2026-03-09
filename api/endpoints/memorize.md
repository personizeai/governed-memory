# Memorize — Store Content with AI Extraction

**SDK method:** `client.memory.memorize(opts)`
**Endpoint:** `POST /api/v1/memorize`

Extracts both open-set atomic facts (memories) and schema-enforced property values from unstructured content in a single pass. Use for meeting transcripts, emails, call notes, documents, chat logs, and any other free-form text.

---

## SDK Usage

```typescript
import { Personize } from '@personize/sdk';
const client = new Personize({ secretKey: process.env.PERSONIZE_SECRET_KEY! });

// Single item — AI extraction enabled
const result = await client.memory.memorize({
    content: 'Call with Sarah Chen (VP Engineering, TechFlow Inc). They are migrating from Oracle to PostgreSQL and evaluating vendors. Budget approved at $450K for Q2. Main pain point: deployment velocity — releases currently take 3 weeks.',
    speaker: 'Sales Team',
    email: 'sarah.chen@techflow.io',
    enhanced: true,
    tags: ['call-notes', 'sales', 'source:manual'],
    actionId: 'your-collection-id',  // schema collection for property extraction
});

console.log(result.data.memories);     // extracted open-set facts
console.log(result.data.properties);   // extracted schema properties
console.log(result.data.qualityGates); // coreference, self-containment, temporal scores
```

---

## Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `content` | string | Yes | Raw text to extract from (transcript, email, document, etc.) |
| `email` | string | No | Contact email — scopes memory to this entity |
| `website_url` | string | No | Company website — scopes memory to this company entity |
| `record_id` | string | No | Explicit record ID — overrides email/website_url resolution |
| `enhanced` | boolean | No | Enable AI extraction of facts and properties (default: `false`) |
| `speaker` | string | No | Who produced this content — appears in extracted facts for attribution |
| `timestamp` | string | No | ISO 8601 timestamp — enables temporal ordering across sources |
| `tags` | string[] | No | Categorization tags for filtering and attribution (recommended: always set) |
| `actionId` | string | No | Schema collection ID to extract properties against |
| `max_properties` | number | No | Max properties to extract per chunk |

> **`enhanced: true` is required for AI extraction.** Without it, content is stored as a structured property but no facts or vectors are created — `smartRecall()` will not find it.

> **Always set `tags`.** Tags enable filtering by source, team, or content type. Untagged memories cannot be filtered by category.

---

## Response

```json
{
  "data": {
    "memories": [
      {
        "id": "mem_abc123",
        "text": "Sarah Chen (VP Engineering at TechFlow Inc) confirmed budget of $450K approved for Q2 migration project.",
        "type": "memory",
        "keywords": ["budget", "Q2", "migration"],
        "persons": ["Sarah Chen"],
        "entities": ["TechFlow Inc"],
        "timestamp": "2026-Q2"
      }
    ],
    "properties": [
      {
        "id": "prop_xyz789",
        "propertyName": "Deal Value",
        "propertyValue": 450000,
        "confidence": 0.94,
        "type": "property_value",
        "collectionId": "your-collection-id"
      }
    ],
    "qualityGates": {
      "coreferenceScore": 0.95,
      "selfContainmentScore": 0.88,
      "temporalAnchoringScore": 0.91
    },
    "stats": {
      "chunksProcessed": 1,
      "factsExtracted": 6,
      "propertiesExtracted": 4,
      "duplicatesSkipped": 0
    }
  }
}
```

### Response fields

| Field | Description |
|---|---|
| `memories[]` | Open-set atomic facts with entity annotations and vector embeddings |
| `properties[]` | Schema-enforced typed values extracted against your collection |
| `qualityGates.coreferenceScore` | Proportion of pronouns resolved to named entities (0-1) |
| `qualityGates.selfContainmentScore` | Proportion of facts understandable without the source document (0-1) |
| `qualityGates.temporalAnchoringScore` | Proportion of relative time references converted to absolute dates (0-1) |
| `stats.duplicatesSkipped` | Facts skipped because a semantically equivalent memory already exists for this entity |

---

## Chunking Behavior

Long content is automatically split into overlapping chunks:

| Content Type | Max Words per Chunk | Overlap |
|---|---|---|
| Dialogue / Chat | 2,000 | 300 words |
| Transcript | 3,500 | 250 words |
| Document / Email | 3,000 | 200 words |

Cross-chunk deduplication merges results: schema properties keep the highest-confidence extraction, and facts are deduplicated by normalized text.

---

## Extraction Hints for Identity Fields

When identity fields (name, company, job title) may be absent for a record, prepend extraction hints so the property selector prioritizes them:

```typescript
await client.memory.memorize({
    content: 'Also extract First Name, Last Name, Company Name, and Job Title if mentioned.\n\nCall with Sarah Chen (VP Engineering, TechFlow Inc)...',
    email: 'sarah.chen@techflow.io',
    enhanced: true,
    tags: ['call-notes', 'sales'],
});
```

---

## Batch Memorization

For syncing multiple records (CRM export, CSV, database rows), use `memorizeBatch()` instead:

```typescript
await client.memory.memorizeBatch({
    source: 'Hubspot',
    mapping: {
        entityType: 'contact',
        email: 'email',
        runName: 'hubspot-contact-sync',
        properties: {
            full_name:  { sourceField: 'firstname', collectionId: 'col_xxx', collectionName: 'Contacts', extractMemories: false },
            job_title:  { sourceField: 'jobtitle',  collectionId: 'col_xxx', collectionName: 'Contacts', extractMemories: false },
            last_notes: { sourceField: 'notes',     collectionId: 'col_xxx', collectionName: 'Contacts', extractMemories: true },
        },
    },
    rows: crmContacts,
});
// Note: memorizeBatch() is async — records land in ~1-2 minutes via background processing.
```

Set `extractMemories: true` on any field with free-form text. Set `extractMemories: false` (default) for structured fields like email, plan tier, or login count.

---

## For Experiment Runners

Use `memorize()` for E01, E05, E06, E09, E11, E12. For each sample:

```typescript
const content = fs.readFileSync('transcript_001.txt', 'utf-8');
const groundTruth = JSON.parse(fs.readFileSync('transcript_001.ground_truth.json', 'utf-8'));

const result = await client.memory.memorize({
    content,
    email: groundTruth.entity_email,
    enhanced: true,
    tags: ['experiment:e01', 'content-type:transcript'],
    actionId: 'your-collection-a-id',
});

// Record for scoring
const factsExtracted = result.data.memories.length;
const propsExtracted = result.data.properties.length;
const factRecall = Math.min(factsExtracted, groundTruth.expected_facts.length) / groundTruth.expected_facts.length;
```
