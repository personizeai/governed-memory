# E01 Results: Extraction Quality Across Content Types

**Run date:** 2026-02-20
**Duration:** 31 minutes 36 seconds
**Samples:** 90 (50 transcripts, 10 emails, 10 chats, 10 documents, 10 call notes)
**API calls:** 91 (90 memorize + 1 identity resolve)

---

## Summary Table

| Content Type | Samples | Avg Facts Extracted | Avg Props Extracted | Fact Recall | Prop Recall |
|---|---|---|---|---|---|
| Call Notes | 10 | 6.9 | 12.5 | **96%** | **99%** |
| Chats | 10 | 6.5 | 10.4 | 85% | 94% |
| Documents | 10 | 7.7 | 12.7 | 79% | 99% |
| Emails | 10 | 7.0 | 7.0 | 77% | 71% |
| Transcripts | 50 | 8.2 | 8.8 | 74% | 78% |

---

## Key Findings

**1. Terse content outperforms verbose content on fact recall.**
Call notes (avg ~500 words) achieve 96% fact recall vs. 74% for transcripts (avg ~2,500 words). Shorter, denser content gives the extractor a cleaner signal. Verbose conversational content introduces noise — filler, repetition, and tangents — that dilutes the useful signal without proportionally increasing extracted facts.

**2. Property recall is consistently high for structured content types.**
Documents and call notes both achieve ~99% property recall. These content types tend to state facts directly ("Budget: $400K", "Call with: Sarah Chen, VP Engineering") rather than embedding them in dialogue. Chat logs and transcripts are more implicit.

**3. Emails show the largest fact-to-property recall gap (77% vs. 71%).**
Email threads present a unique structural challenge: forwarded messages, signature blocks, and quoted reply chains create ambiguity about which speaker/entity facts belong to. Property recall suffers more than fact recall because typed extraction requires confident entity attribution.

**4. Transcripts have the highest absolute fact and property counts.**
Even at lower recall rates, transcripts extract more total facts (8.2 avg) than call notes (6.9 avg) because they contain more ground truth facts (8-14 per sample vs. 5-7 for call notes). More content = more information, just noisier.

---

## Quality Gate Metrics

Quality gate scores (coreference, self-containment, temporal anchoring) were instrumented in the API response schema but were not captured per-sample in this run. These fields are `null` in the raw JSON. A follow-up run will collect these scores across the same 90 samples.

---

## Per-Sample Results

See `e01_extraction_quality_20260220_215510.json` for the full per-sample breakdown (90 records).

Each record contains:
- `sample_id`, `content_type`
- `memories_extracted`, `properties_extracted`
- `expected_fact_count`, `expected_property_count`
- `fact_recall`, `property_recall`
- `planted_pronoun_issues`, `planted_temporal_issues`
- `duration_ms`, `wall_clock_ms`
- `duplicates_skipped`

---

## Comparison to Production Benchmarks

| Metric | Production Benchmark | E01 Result | Within Range? |
|---|---|---|---|
| Facts per record | 8.2 avg | 8.2 (transcripts) | Yes |
| Properties per record | 6.7 avg | 8.8 (transcripts) | Above (schema has 14 props) |
| Dedup rate | ~12% | ~0% (E01 used skipStorage) | N/A — dedup not tested here |

The higher property count in E01 (8.8 vs 6.7 production avg) reflects the 14-property schema used in experiments vs. the ~8-property average schema in production.
