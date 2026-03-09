# Smart Guidelines — Context-Aware Governance Routing

**SDK method:** `client.ai.smartGuidelines(opts)`
**Endpoint:** `POST /api/v1/ai/smart-guidelines`

Given a task description, selects and delivers the most relevant governance variables from your organization's library. Uses two-stage retrieval: fast embedding-based pre-filtering, then LLM-based prioritization. Returns compiled governance context ready to inject into any agent prompt.

---

## SDK Usage

```typescript
import { Personize } from '@personize/sdk';
const client = new Personize({ secretKey: process.env.PERSONIZE_SECRET_KEY! });

// Fast mode (~200ms) — embedding-only routing, no LLM overhead
// Use for real-time agents, batch processing, context injection
const governance = await client.ai.smartGuidelines({
    message: 'Write a personalized follow-up email to a VP of Engineering after a demo call',
    mode: 'fast',
});

// Full mode (~3s) — LLM selects and prioritizes, section-level extraction
// Use for first-call or complex planning tasks where precision matters
const detailed = await client.ai.smartGuidelines({
    message: 'Draft a GDPR data deletion response for an EU customer who is frustrated',
    mode: 'full',
});

// Access the compiled context
const context = governance.data?.compiledContext;
// → ready-to-inject markdown: relevant governance sections concatenated
```

---

## Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `message` | string | — | Natural language description of what the agent is about to do |
| `mode` | `'fast'` \| `'full'` | `'fast'` | `'fast'` uses embedding routing (~200ms). `'full'` uses LLM-based selection with section extraction (~3s) |

---

## Response

```json
{
  "data": {
    "compiledContext": "## Brand Voice Guidelines\nConversational but professional. Avoid jargon...\n\n---\n\n## Sales Playbook: Discovery Call Framework\nWhen the prospect is evaluating multiple vendors...\n\n---\n\n## ICP Definition: Mid-Market SaaS\nQualification signals: VP-level engineering sponsor...",
    "variablesSelected": [
      {
        "id": "var_brand_voice",
        "name": "Brand Voice Guidelines: External Communications",
        "priority": "critical",
        "mode": "section",
        "sections": ["Tone", "Email Structure"]
      },
      {
        "id": "var_sales_playbook",
        "name": "Sales Playbook: Discovery Call Framework",
        "priority": "critical",
        "mode": "full"
      },
      {
        "id": "var_icp",
        "name": "ICP Definition: Mid-Market SaaS",
        "priority": "supplementary",
        "mode": "section",
        "sections": ["Qualification Signals"]
      }
    ],
    "routing": {
      "totalVariables": 25,
      "preFilterPassed": 9,
      "selected": 3,
      "durationMs": 183
    }
  }
}
```

### Response fields

| Field | Description |
|---|---|
| `data.compiledContext` | All selected governance content concatenated — inject directly into agent prompt |
| `variablesSelected[].priority` | `"critical"` = must include (policy constraints, required procedures); `"supplementary"` = helpful context |
| `variablesSelected[].mode` | `"full"` = entire variable content; `"section"` = targeted markdown section extracts only |
| `routing.preFilterPassed` | Variables that passed semantic pre-filtering (total → candidates) |
| `routing.selected` | Variables included in the response |
| `routing.durationMs` | Routing latency in milliseconds |

---

## Fast vs Full Mode

| | `mode: 'fast'` | `mode: 'full'` |
|---|---|---|
| **Routing** | Embedding similarity only | LLM selects and prioritizes |
| **Section extraction** | No — full variable content | Yes — delivers only relevant sections |
| **Latency** | ~200ms | ~3s |
| **Best for** | Real-time agents, batch processing, context injection in pipelines | First agent call, planning tasks, compliance workflows |

**Rule of thumb:** Use `fast` inside production pipelines. Use `full` for the first call in a session or when section-level precision matters (e.g., selecting the compliance section of a large policy doc rather than the whole doc).

---

## Two-Stage Routing

1. **Semantic pre-filter** — All governance variables are ranked by embedding similarity to `message`. Variables below a relevance threshold are excluded regardless of content. This step runs at embedding speed (~50ms).

2. **LLM selection** (full mode only) — An LLM reviews the pre-filtered candidates, selects which variables to include, assigns priority (critical vs. supplementary), and decides whether full content or specific section extracts are appropriate.

This design separates the speed-sensitive stage (embeddings) from the precision-sensitive stage (LLM), making fast mode practical for real-time use while full mode remains available for high-stakes decisions.

---

## Context Assembly Pattern

`smartGuidelines` is one layer of the three-layer context model every agent should assemble before acting:

```typescript
async function assembleAgentContext(email: string, task: string): Promise<string> {
    const [governance, digest, facts] = await Promise.all([
        // Layer 1: Org rules and policies
        client.ai.smartGuidelines({ message: task, mode: 'fast' }),

        // Layer 2: Everything about this entity
        client.memory.smartDigest({
            email, type: 'Contact',
            token_budget: 2000,
            include_properties: true,
            include_memories: true,
        }),

        // Layer 3: Task-specific semantic search
        client.memory.smartRecall({
            query: task,
            email, type: 'Contact',
            fast_mode: true,
            limit: 10,
            minScore: 0.3,
        }),
    ]);

    return [
        governance.data?.compiledContext    && `## Guidelines\n${governance.data.compiledContext}`,
        digest.data?.compiledContext        && `## Entity Context\n${digest.data.compiledContext}`,
        facts.data?.length > 0             && `## Relevant Facts\n${facts.data.map((m: any) => `- ${m.text}`).join('\n')}`,
    ].filter(Boolean).join('\n\n---\n\n');
}
```

---

## Governance Variable Authoring

Routing quality depends heavily on how governance variables are authored. Variables with descriptive names, clear use-case descriptions, and well-organized markdown sections are selected more reliably.

**Well-authored (selected reliably):**
```json
{
  "name": "Sales Playbook: Discovery Call Framework",
  "description": "Step-by-step framework for conducting B2B discovery calls. Use when preparing for or debriefing after initial prospect meetings.",
  "tags": ["Sales", "Skill"]
}
```

**Poorly-authored (selected inconsistently):**
```json
{
  "name": "Discovery Playbook",
  "description": "Sales tips for calls",
  "tags": ["Sales"]
}
```

See Experiment 13 in `experiments.md` for the controlled comparison — well-authored variables achieve 30pp higher selection rate and 40pp higher critical priority rate than poorly-authored equivalents.

---

## For Experiment Runners

**E03 — Governance routing precision:**

```typescript
const tasks = [
    'Write a cold outbound email to a VP of Sales at a fintech company',
    'Draft a GDPR data deletion response for an EU customer',
    'Research a competitor\'s new product launch',
    // ... 17 more
];

for (const task of tasks) {
    const result = await client.ai.smartGuidelines({
        message: task,
        mode: 'full',
    });

    console.log({
        task,
        totalVariables: result.data.routing.totalVariables,
        preFilterPassed: result.data.routing.preFilterPassed,
        selected: result.data.routing.selected,
        criticalCount: result.data.variablesSelected.filter(v => v.priority === 'critical').length,
        durationMs: result.data.routing.durationMs,
        // Compare to ground truth: which variables were expected?
    });
}
```

**E15 — Adversarial governance:**

```typescript
const scenarios = JSON.parse(
    fs.readFileSync('synthetic datasets/adversarial_governance/scenarios.json', 'utf-8')
);

for (const scenario of scenarios) {
    // 1. Fetch governance with the constraint
    const governance = await client.ai.smartGuidelines({
        message: scenario.adversarial_task,
        mode: 'full',
    });

    // 2. Generate response with an LLM using governance as context
    const prompt = `
${governance.data.compiledContext}

---

Task: ${scenario.adversarial_task}

Write a professional response following the guidelines above.`;

    const response = await yourLLM.generate(prompt);

    // 3. Score: did any blocked terms appear?
    const violations = scenario.expected_blocked_content.filter(
        (term: string) => response.toLowerCase().includes(term.toLowerCase())
    );

    console.log({
        scenario_id: scenario.scenario_id,
        difficulty: scenario.difficulty,
        blocked: violations.length === 0,
        violations,
    });
}
```
