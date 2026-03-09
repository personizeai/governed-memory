# Recall Query Set — Sarah Chen at TechFlow Inc

**Entity:** Sarah Chen, VP of Engineering at TechFlow Inc

**Total queries:** 10


## query_01 (broad)

**Query:** What do we know about Sarah Chen at TechFlow Inc?

**Expected sources:** source_1, source_2, source_3, source_4, source_5

**Expected topics:** role, company, deal, technology, background

**Notes:** Should retrieve memories from ALL sources. Tests comprehensive recall.


## query_02 (broad)

**Query:** Summarize our relationship with TechFlow Inc

**Expected sources:** source_1, source_2, source_3, source_4

**Expected topics:** meeting history, deal status, next steps

**Notes:** Should retrieve interaction memories. Source 5 (research) less relevant.


## query_03 (specific)

**Query:** What is TechFlow Inc's technology stack?

**Expected sources:** source_1, source_5

**Expected topics:** Java, Spring Boot, AWS, Oracle, PostgreSQL migration

**Notes:** Should retrieve 2-3 memories mentioning technical stack details.


## query_04 (specific)

**Query:** What's the deal value and expected close date?

**Expected sources:** source_2, source_3, source_4

**Expected topics:** $450K, Q2 2026

**Notes:** Tests dedup — this fact appears in 3+ sources. Should NOT return duplicates.


## query_05 (multi_topic)

**Query:** What are their pain points and who makes the buying decision?

**Expected sources:** source_1, source_2, source_4

**Expected topics:** deployment velocity, tooling gaps, Sarah as champion, CFO involvement

**Notes:** Tests retrieval across two distinct topics in a single query.


## query_06 (multi_topic)

**Query:** What competitors did they mention and what's their current solution?

**Expected sources:** source_1, source_2, source_4

**Expected topics:** Gong, Microsoft Copilot, Oracle database, Spring Boot backend

**Notes:** Tests extraction of competitive intelligence across sources.


## query_07 (temporal)

**Query:** What happened in our most recent conversation?

**Expected sources:** source_2

**Expected topics:** vendor evaluation, SOC 2, Snowflake integration, board meeting

**Notes:** Should retrieve memories from source 2 (follow-up) not source 1 (discovery).


## query_08 (temporal)

**Query:** How has Sarah Chen's position evolved across our conversations?

**Expected sources:** source_1, source_2, source_3, source_4

**Expected topics:** initial interest -> active evaluation -> champion status

**Notes:** Tests temporal understanding across chronologically ordered sources.


## query_09 (relational)

**Query:** Who else at TechFlow Inc should we be talking to?

**Expected sources:** source_1, source_3, source_4

**Expected topics:** CEO Mark Thompson, Mike Rivera, CFO, security team

**Notes:** Tests extraction of stakeholder relationships.


## query_10 (relational)

**Query:** What technical requirements have they specified?

**Expected sources:** source_2, source_3

**Expected topics:** SOC 2, API-first, Snowflake, <200ms retrieval, phased rollout

**Notes:** Tests retrieval of technical specifications across sources.
