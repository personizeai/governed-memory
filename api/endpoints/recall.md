# Recall — Retrieve Entity Memory

Two methods depending on what you need:

| Method | SDK | Endpoint | Use when |
|---|---|---|---|
| **`smartRecall`** | `client.memory.smartRecall(opts)` | `POST /api/v1/smart-recall` | Semantic search with optional reflection and answer generation — recommended for most use cases |
| **`recall`** | `client.memory.recall(opts)` | `POST /api/v1/recall` | Direct lookup — simpler, requires `type`, no reflection |
| **`smartDigest`** | `client.memory.smartDigest(opts)` | `POST /api/v1/smart-memory-digest` | Full entity context — all properties and memories compiled into a single token-budgeted block |

---

## SDK Usage

```typescript
import { Personize } from '@personize/sdk';
const client = new Personize({ secretKey: process.env.PERSONIZE_SECRET_KEY! });

// --- smartRecall: semantic search (recommended) ---
const results = await client.memory.smartRecall({
    query: 'What pain points did this contact mention and what is their technology stack?',
    email: 'sarah.chen@techflow.io',
    type: 'Contact',
    limit: 10,
    minScore: 0.4,
    include_property_values: true,
});
// results.data → array of memory entries with relevance scores

// --- Fast mode: skip reflection, ~500ms ---
const fast = await client.memory.smartRecall({
    query: 'What do we know about this contact?',
    email: 'sarah.chen@techflow.io',
    type: 'Contact',
    fast_mode: true,   // embedding-only, no LLM reflection overhead
    limit: 10,
    minScore: 0.3,
});

// --- smartDigest: full entity context block ---
const digest = await client.memory.smartDigest({
    email: 'sarah.chen@techflow.io',
    type: 'Contact',
    token_budget: 2000,        // max tokens for the compiled output
    include_properties: true,
    include_memories: true,
});
// digest.data.compiledContext → ready-to-inject markdown string
```

---

## Parameters

### `smartRecall()` Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `query` | string | — | Natural language question or topic |
| `email` | string | — | Scope search to one contact by email |
| `website_url` | string | — | Scope search to one company by website |
| `record_id` | string | — | Scope search to one record by ID |
| `type` | string | — | Entity type filter (`'Contact'`, `'Company'`) — optional, inferred from email/website_url |
| `limit` | number | 10 | Max results to return |
| `minScore` | number | — | Minimum relevance score (0-1). Recommended: 0.3 for broad, 0.5+ for precision |
| `include_property_values` | boolean | false | Include structured schema properties alongside semantic results |
| `enable_reflection` | boolean | true | AI checks completeness and generates follow-up queries (disabled in fast_mode) |
| `generate_answer` | boolean | false | AI synthesizes a direct answer from retrieved results |
| `fast_mode` | boolean | false | Skip reflection and answer gen — ~500ms latency. Recommended for real-time and batch |
| `min_score` | number | 0.3 | Server-side score filter applied in fast_mode |

### `smartDigest()` Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `email` | string | — | Contact email |
| `website_url` | string | — | Company website |
| `record_id` | string | — | Record ID |
| `type` | string | — | Entity type (`'Contact'`, `'Company'`) |
| `token_budget` | number | 1000 | Max tokens for compiled output — always set explicitly |
| `max_memories` | number | 20 | Max memories to include |
| `include_properties` | boolean | true | Include structured schema properties |
| `include_memories` | boolean | true | Include open-set semantic memories |

---

## Response: `smartRecall()`

```json
{
  "data": [
    {
      "id": "mem_abc123",
      "text": "Sarah Chen (VP Engineering at TechFlow Inc) confirmed $450K budget approved for Q2 migration project.",
      "type": "memory",
      "score": 0.94,
      "timestamp": "2026-01-15",
      "source": "call-notes"
    },
    {
      "id": "prop_xyz789",
      "text": "Deal Stage: Proposal Sent",
      "type": "property_value",
      "propertyName": "Deal Stage",
      "propertyValue": "Proposal Sent",
      "confidence": 0.95,
      "score": 0.88
    }
  ],
  "answer": {
    "text": "TechFlow's main pain points are deployment velocity (releases take 3 weeks) and Oracle database management costs. Their stack is Java/Spring Boot backend with Oracle DB, planning to migrate to PostgreSQL and AWS in Q2 2026.",
    "confidence": "high",
    "sourceMemoryIds": ["mem_abc123", "mem_def456"]
  },
  "reflectionStats": {
    "roundsUsed": 1,
    "maxRounds": 2,
    "additionalQueriesGenerated": 1
  }
}
```

## Response: `smartDigest()`

```json
{
  "data": {
    "compiledContext": "## Sarah Chen — TechFlow Inc\n\n**Properties**\n- Job Title: VP of Engineering\n- Technology Stack: Java, Spring Boot, Oracle (migrating to AWS)\n- Deal Value: $450,000\n- Deal Stage: Proposal Sent\n\n**Recent Memories**\n- Confirmed $450K budget for Q2 migration (Jan 15)\n- Evaluating 3 vendors; SOC 2 compliance required (Jan 22)\n- Mike Rivera (Director of Data) is technical evaluator (email thread)...",
    "tokenCount": 487,
    "memoriesIncluded": 12,
    "propertiesIncluded": 8
  }
}
```

---

## Reflection Behavior

When `enable_reflection` is true (default in non-fast-mode):

1. Performs entity-scoped vector search on the query
2. An LLM checks: "Is this evidence complete for the question?"
3. If incomplete, generates 1-2 targeted follow-up queries
4. Searches again with follow-up queries and merges results (deduplicated by ID)
5. Repeats up to the configured round limit

Each reflection round adds one LLM call plus 1-2 additional vector searches. Production average: ~1.3 rounds per query.

| Mode | Latency | When to use |
|---|---|---|
| `fast_mode: true` | ~500ms | Real-time agents, batch processing, context injection |
| Default (reflection on) | ~2-10s | Exploratory queries, completeness-critical workflows |
| `generate_answer: true` | +1-3s | When you need a synthesized answer, not just a list |

---

## Context Assembly Pattern

Most generation pipelines combine all three recall methods:

```typescript
async function assembleContext(email: string, task: string): Promise<string> {
    const sections: string[] = [];

    // 1. Governance — org rules and policies
    const governance = await client.ai.smartGuidelines({
        message: `${task} — guidelines, tone, constraints`,
        mode: 'fast',
    });
    if (governance.data?.compiledContext) {
        sections.push('## Guidelines\n' + governance.data.compiledContext);
    }

    // 2. Entity context — everything about this person
    const digest = await client.memory.smartDigest({
        email,
        type: 'Contact',
        token_budget: 2000,
        include_properties: true,
        include_memories: true,
    });
    if (digest.data?.compiledContext) {
        sections.push('## Recipient Context\n' + digest.data.compiledContext);
    }

    // 3. Task-specific facts — semantic search
    const recalled = await client.memory.smartRecall({
        query: task,
        email,
        type: 'Contact',
        fast_mode: true,
        limit: 10,
        minScore: 0.3,
    });
    if (recalled.data?.length > 0) {
        sections.push('## Relevant Facts\n' + recalled.data.map((m: any) => `- ${m.text}`).join('\n'));
    }

    return sections.join('\n\n---\n\n');
}
```

---

## For Experiment Runners

**E07 — Recall latency by store density:**

```typescript
const queries = JSON.parse(fs.readFileSync('synthetic datasets/recall_queries/recall_queries.json', 'utf-8'));

for (const query of queries) {
    const start = Date.now();
    const result = await client.memory.smartRecall({
        query: query.query,
        email: query.entity_email,
        type: 'Contact',
        fast_mode: false,   // measure reflection overhead
        limit: 15,
        minScore: 0.3,
    });
    const latencyMs = Date.now() - start;

    console.log({
        queryId: query.id,
        category: query.category,
        resultsReturned: result.data.length,
        latencyMs,
        roundsUsed: result.reflectionStats?.roundsUsed,
    });
}
```

**E10 — Reflection rounds ablation:**

```typescript
for (const rounds of [0, 1, 2]) {
    const result = await client.memory.smartRecall({
        query: 'What are TechFlow\'s pain points, migration timeline, and competitive landscape?',
        email: 'sarah.chen@techflow.io',
        type: 'Contact',
        fast_mode: rounds === 0,        // fast_mode disables reflection
        enable_reflection: rounds > 0,
        limit: 15,
    });

    console.log({
        rounds,
        resultsReturned: result.data.length,
        roundsActuallyUsed: result.reflectionStats?.roundsUsed ?? 0,
    });
}
```

**E11 — Entity isolation:**

```typescript
// Scoped query — should return ONLY this entity's memories
const scoped = await client.memory.smartRecall({
    query: 'What is their technology stack?',
    email: 'sarah.chen@techflow-e11.test',   // entity_email from entity_isolation metadata
    type: 'Contact',
    fast_mode: true,
    limit: 20,
});

// Check every result — should all belong to this entity
const leaks = scoped.data.filter(m => !m.text.includes('TechFlow'));
console.log({ totalResults: scoped.data.length, crossEntityLeaks: leaks.length });
```
