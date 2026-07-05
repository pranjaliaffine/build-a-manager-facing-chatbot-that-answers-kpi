# Build a manager-facing chatbot that answers KPI and business questions a…

Agentic workflow architecture exported from **[Agentic LaunchPad](https://github.com)** by Affine Analytics.

> This repository is a scaffold generated from an interactive architecture interview and visual workflow builder. Implement each agent step per the plan below.

## At a glance

- **Session:** `session-1783076414314-khkzgw`
- **Steps:** 10
- **Connections:** 11
- **Exported:** 2026-07-05 18:52 UTC

## Problem statement

Build a manager-facing chatbot that answers KPI and business questions across structured databases and enterprise documents using a shared enterprise knowledge base.

## Requirements

### Interview summary

Provide internal managers with fast, trustworthy KPI summaries and business answers by combining curated document retrieval with live database lookups, while showing evidence on demand and handling uncertainty safely.

### Architecture blueprint

## Provide internal managers with fast, trustworthy KPI summaries and business answers by combining curated document retrieval with live database lookups, while showing evidence on demand and handling uncertainty safely.

Build a manager-facing chatbot that answers KPI and business questions across structured databases and enterprise documents using a shared enterprise knowledge base.

### Integrations
Shared enterprise knowledge base, Enterprise document repository, Structured databases for live KPI queries

### HITL
No mandatory human review in the normal flow. Managers can inspect sources on demand to verify answers or investigate flagged conflicts.

## Architecture summary

This architecture implements a manager-facing chatbot that answers KPI and business questions by combining retrieval from the shared enterprise knowledge base and enterprise document repository with live queries against structured databases. The orchestration model is a routed mixed-mode flow: the system first classifies each question as document, database, or mixed, then executes the relevant retrieval paths, compares outputs, and returns a grounded answer with optional evidence and explicit conflict signaling.

The main flow starts with manager question intake, then an intent-routing gateway sends requests to the document path, the database path, or both. On the document side, the GraphRAG Index & Query Agent is reused for enterprise knowledge-base retrieval, followed by an adapted Eryl Semantic RAG Agent Chain to produce document-grounded findings. In parallel when needed, a custom live SQL node retrieves fresh KPI values from structured databases. A custom conflict-and-confidence gateway then checks for disagreement between live data and knowledge-base content and enforces the requirement to return "I do not know" when evidence is insufficient. The adapted Final Answer Consolidation Agent composes the manager-friendly response, and an adapted GraphRAG evidence node supports source snippets and document names on demand.

Reuse is catalog-first: 4 of 10 nodes are direct reuse/adapt processing agents, and the core processing chain is anchored on catalog assets for retrieval, semantic answering, consolidation, and citation delivery. Custom build is reserved for capabilities not covered by the catalog: conversational intake, intent routing, live SQL execution, and conflict/confidence governance. The optional human-in-the-loop behavior is represented as manager source review after citation delivery, matching the requirement that there is no mandatory human review in the normal flow but managers can inspect sources or flagged conflicts on demand.

Key risks are safe SQL generation against live systems, maintaining grounding across mixed document and database answers, and calibrating the confidence threshold so the system neither fabricates nor over-defers. Additional implementation care is needed to preserve source traceability across both integrations and to ensure conflict flags are surfaced clearly rather than silently resolved.

## Workflow overview

This architecture has **10** step(s) and **11** connection(s).

### Execution flow

- **Manager Question Intake** → *question text and user context* → **Intent Routing Gateway**
- **Intent Routing Gateway** → *document or mixed route* → **Enterprise Document Retrieval**
- **Intent Routing Gateway** → *database or mixed route* → **Live KPI SQL Query**
- **Enterprise Document Retrieval** → *retrieved passages and KB evidence* → **Semantic RAG Answering**
- **Semantic RAG Answering** → *document-grounded findings* → **Conflict Detection and Confidence Gate**
- **Live KPI SQL Query** → *live KPI query results* → **Conflict Detection and Confidence Gate**
- **Conflict Detection and Confidence Gate** → *validated evidence, conflict flags, confidence status* → **Manager Answer Consolidation**
- **Manager Answer Consolidation** → *answer plus evidence references* → **Citation and Evidence Delivery**
- **Citation and Evidence Delivery** → *on-demand source inspection* → **Manager Source Review**
- **Citation and Evidence Delivery** → *default answer delivery* → **Final Response Outcome**
- **Manager Source Review** → *reviewed answer acceptance* → **Final Response Outcome**

## Agents & steps

### Manager Question Intake

*custom* · **build**

Accepts the manager's question and session context for downstream routing.

*Rationale:* Auto-applied: catalog alternative identified during validation.

**Purpose:** Capture the manager's natural-language KPI or business question and pass it into the workflow with session context.

**Role:** This is the entry point for every interaction. It receives the manager's question from the chatbot UI, preserves any relevant conversational context, and normalizes the request so downstream routing can decide whether to use documents, live databases, or both.

**Execution:** Accepts the manager's question and session context for downstream routing.

**Consumes from:**
- Manager chatbot UI input
- session context

**Feeds into:**
- Intent Routing Gateway
- question text and user context

**Inputs:**
- Workflow entry (user request or trigger)

**Outputs:**
- Processed result from: Workflow entry (user request or trigger)

### Intent Routing Gateway

*custom* · **build**

Determines whether the question should use document retrieval, live SQL, or both.

*Rationale:* Auto-applied: catalog alternative identified during validation.

**Purpose:** Classify each question as document-focused, database-focused, or mixed and send it to the right retrieval path.

**Role:** After intake, this node decides how the question should be answered. It routes document-heavy questions to the enterprise knowledge-base path, KPI freshness questions to the live SQL path, and mixed questions to both so the system can produce one combined answer.

**Execution:** Determines whether the question should use document retrieval, live SQL, or both.

**Consumes from:**
- Manager Question Intake
- question text and user context

**Feeds into:**
- Enterprise Document Retrieval
- Live KPI SQL Query
- document or mixed route
- database or mixed route

**Inputs:**
- Processed result from: Workflow entry (user request or trigger)

**Outputs:**
- Processed result from: Processed result from: Workflow entry (user request or trigger)

### Enterprise Document Retrieval

*agent* · catalog `graphrag_index_query_agent` · **reuse** · GraphRAG Index & Query Agent

Searches the enterprise knowledge base and document repository for relevant grounded evidence.

*Rationale:* This agent strongly aligns to indexing and querying enterprise knowledge sources for grounded retrieval.

**Purpose:** Retrieve grounded enterprise content from the shared knowledge base and document repository for document or mixed questions.

**Role:** When the router selects the document path, this node searches curated enterprise sources for relevant passages, KPI snapshots, and supporting business context. Its output is evidence, not the final answer, and it prepares the material needed for document-grounded reasoning in the next step.

**Execution:** Searches the enterprise knowledge base and document repository for relevant grounded evidence.

**Consumes from:**
- Intent Routing Gateway
- document or mixed route
- shared enterprise knowledge base
- enterprise document repository

**Feeds into:**
- Semantic RAG Answering
- retrieved passages and KB evidence

**Inputs:**
- project_id
- TXT documents in projects/{id}/input/
- natural language question
- search method (local/global/drift/basic)

**Outputs:**
- entities.parquet
- community_reports.parquet
- LanceDB embeddings
- natural language answer text

### Semantic RAG Answering

*agent* · catalog `eryl_semantic_rag_agent_chain` · **adapt** · Eryl Semantic RAG Agent Chain

Generates document-grounded findings and a draft answer from retrieved enterprise content.

*Rationale:* The agent already performs semantic retrieval and answering over unstructured documents and needs only light adaptation to enterprise content domains.

**Purpose:** Turn retrieved enterprise document evidence into a draft, document-grounded answer for document-heavy or mixed questions.

**Role:** This node takes the passages found by document retrieval and performs semantic reasoning over them to extract the relevant business answer. It produces structured findings and a draft answer that can later be compared against live KPI results if the SQL path also ran.

**Execution:** Generates document-grounded findings and a draft answer from retrieved enterprise content.

**Consumes from:**
- Enterprise Document Retrieval
- retrieved passages and KB evidence

**Feeds into:**
- Conflict Detection and Confidence Gate
- document-grounded findings

**Inputs:**
- semantic_question
- Vector_Data routing hints

**Outputs:**
- retrieval-grounded answer
- Semantic-based analysis_type

### Live KPI SQL Query

*agent* · catalog `quin_sql_agent_chain` · **reuse** · Quin SQL Agent Chain

Runs live SQL queries to retrieve current KPI values from structured systems.

*Rationale:* Reuses catalog agent Quin SQL Agent Chain for structured SQL query generation and execution.

**Purpose:** Fetch fresh KPI values from structured databases when the question requires live operational data.

**Role:** If the router identifies a database or mixed question, this node generates and executes parameterized SQL queries against approved structured systems. It returns current KPI values and structured results that can be compared with knowledge-base content and merged into the final answer.

**Execution:** Runs live SQL queries to retrieve current KPI values from structured systems.

**Consumes from:**
- Intent Routing Gateway
- database or mixed route
- structured databases for live KPI queries

**Feeds into:**
- Conflict Detection and Confidence Gate
- live KPI query results

**Inputs:**
- sql_question
- SQL_Meta_Data schema
- optional vision_context block

**Outputs:**
- sql_query results
- SQL-based narrative answer
- analysis_type SQL-based

### Conflict Detection and Confidence Gate

*custom* · **build**

Checks evidence sufficiency and flags conflicts between knowledge-base content and live KPI results.

*Rationale:* Auto-applied: catalog alternative identified during validation.

**Purpose:** Compare document-derived findings and live KPI results, flag disagreements, and decide whether evidence is strong enough to answer.

**Role:** This node sits after both evidence-producing branches and acts as the trust-control layer. It checks for mismatches between live database outputs and knowledge-base content, preserves both when they disagree, and determines whether the system should proceed with an answer or fall back to 'I do not know' due to insufficient confidence.

**Execution:** Checks evidence sufficiency and flags conflicts between knowledge-base content and live KPI results.

**Consumes from:**
- Semantic RAG Answering
- Live KPI SQL Query
- document-grounded findings
- live KPI query results

**Feeds into:**
- Manager Answer Consolidation
- validated evidence, conflict flags, confidence status

**Inputs:**
- retrieval-grounded answer
- Semantic-based analysis_type

**Outputs:**
- Processed result from: retrieval-grounded answer
- Processed result from: Semantic-based analysis_type

### Manager Answer Consolidation

*agent* · catalog `final_answer_consolidation_agent` · **adapt** · Final Answer Consolidation Agent

Builds a manager-friendly final answer from validated evidence, confidence status, and conflict flags.

*Rationale:* The consolidation pattern fits well and can be adapted from its current domain to merge enterprise document and KPI outputs into a final response.

**Purpose:** Merge validated evidence, live KPI outputs, and any conflict or confidence signals into one manager-friendly summary.

**Role:** Once the system knows what evidence is available and whether conflicts exist, this node composes the actual response text. It turns raw retrieval outputs and SQL results into a concise manager-facing answer that can include both perspectives, note any mismatch, and respect low-confidence fallback decisions.

**Execution:** Builds a manager-friendly final answer from validated evidence, confidence status, and conflict flags.

**Consumes from:**
- Conflict Detection and Confidence Gate
- validated evidence, conflict flags, confidence status

**Feeds into:**
- Citation and Evidence Delivery
- answer plus evidence references

**Inputs:**
- User query
- intermediate JSON with count_analysis and generic_analysis

**Outputs:**
- final_answer
- reasoning
- user_query

### Citation and Evidence Delivery

*agent* · catalog `graphrag_index_query_agent` · **adapt** · GraphRAG Index & Query Agent

Delivers the answer with optional citations and source evidence available on demand.

*Rationale:* The same GraphRAG capability can be adapted to expose supporting snippets and document references for evidence-on-demand delivery.

**Purpose:** Attach and serve source snippets, document names, and supporting references for optional evidence inspection.

**Role:** After the answer is composed, this node prepares the evidence package that supports it. It enables the default response to be delivered immediately while also making citations and snippets available if the manager chooses to inspect the basis for the answer or investigate a flagged conflict.

**Execution:** Delivers the answer with optional citations and source evidence available on demand.

**Consumes from:**
- Manager Answer Consolidation
- answer plus evidence references

**Feeds into:**
- Manager Source Review
- Final Response Outcome
- on-demand source inspection
- default answer delivery

**Inputs:**
- project_id
- TXT documents in projects/{id}/input/
- natural language question
- search method (local/global/drift/basic)

**Outputs:**
- entities.parquet
- community_reports.parquet
- LanceDB embeddings
- natural language answer text

### Manager Source Review

*human* · **build**

Provides an optional manager review step for inspecting citations and flagged conflicts.

*Rationale:* This is an optional human inspection step defined by the HITL requirement rather than an automatable catalog agent.

**Purpose:** Let the manager optionally inspect cited sources or flagged conflicts before accepting the answer.

**Role:** This is an optional human review branch, not a mandatory approval gate. Managers can enter this step when they want to verify evidence, compare conflicting sources, or investigate why the system produced a particular KPI summary.

**Execution:** Provides an optional manager review step for inspecting citations and flagged conflicts.

**Consumes from:**
- Citation and Evidence Delivery
- on-demand source inspection

**Feeds into:**
- Final Response Outcome
- reviewed answer acceptance

**Inputs:**
- entities.parquet
- community_reports.parquet
- LanceDB embeddings
- natural language answer text

**Outputs:**
- Human review decision
- Approved / rejected outcome for next step

### Final Response Outcome

*custom* · **build**

Returns the final manager-facing answer, conflict notice, citations option, or 'I do not know' fallback.

*Rationale:* Final channel delivery and fallback rendering are application-specific presentation concerns outside the catalog.

**Purpose:** Return the final chatbot result as either a grounded answer with optional evidence and conflict flags or an 'I do not know' fallback.

**Role:** This is the terminal delivery step for the workflow. It presents the default answer path directly from citation delivery, or the reviewed result after optional source inspection, and ensures the final output shown in the chatbot matches the confidence and conflict decisions made upstream.

**Execution:** Returns the final manager-facing answer, conflict notice, citations option, or 'I do not know' fallback.

**Consumes from:**
- Citation and Evidence Delivery
- Manager Source Review
- default answer delivery
- reviewed answer acceptance

**Feeds into:**
- Manager chatbot response

**Inputs:**
- Human review decision
- Approved / rejected outcome for next step

**Outputs:**
- Processed result from: Human review decision
- Processed result from: Approved / rejected outcome for next step

## Reuse decisions

- **Manager Question Intake** — `build` → custom
  - Auto-applied: catalog alternative identified during validation.
- **Intent Routing Gateway** — `build` → custom
  - Auto-applied: catalog alternative identified during validation.
- **Enterprise Document Retrieval** — `reuse` → GraphRAG Index & Query Agent
  - This agent strongly aligns to indexing and querying enterprise knowledge sources for grounded retrieval.
- **Semantic RAG Answering** — `adapt` → Eryl Semantic RAG Agent Chain
  - The agent already performs semantic retrieval and answering over unstructured documents and needs only light adaptation to enterprise content domains.
- **Live KPI SQL Query** — `reuse` → Quin SQL Agent Chain
  - Reuses catalog agent Quin SQL Agent Chain for structured SQL query generation and execution.
- **Conflict Detection and Confidence Gate** — `build` → custom
  - Auto-applied: catalog alternative identified during validation.
- **Manager Answer Consolidation** — `adapt` → Final Answer Consolidation Agent
  - The consolidation pattern fits well and can be adapted from its current domain to merge enterprise document and KPI outputs into a final response.
- **Citation and Evidence Delivery** — `adapt` → GraphRAG Index & Query Agent
  - The same GraphRAG capability can be adapted to expose supporting snippets and document references for evidence-on-demand delivery.
- **Manager Source Review** — `build` → custom
  - This is an optional human inspection step defined by the HITL requirement rather than an automatable catalog agent.
- **Final Response Outcome** — `build` → custom
  - Final channel delivery and fallback rendering are application-specific presentation concerns outside the catalog.

## Catalog matches

- **GraphRAG Index & Query Agent** (`graphrag_index_query_agent`) — score 0.77
  - Matched for: Build a manager-facing chatbot that answers KPI and business questions across st
  - Indexes per-project KYC documents into a knowledge graph with entity/community extraction and embeddings; answers analyst questions via local, global, drift, or basic search methods.
- **KYC Risk Report Generator** (`kyc_risk_report_generator`) — score 0.77
  - Matched for: Provide internal managers with fast, trustworthy KPI summaries and business answ
  - Generates downloadable Word (.docx) KYC risk assessment reports with KPI summary table, section scores, UBO details, override status, and AI reasoning narratives.
- **Eryl Semantic RAG Agent Chain** (`eryl_semantic_rag_agent_chain`) — score 0.65
  - Matched for: Build a manager-facing chatbot that answers KPI and business questions across st
  - Retrieves and answers from indexed retail policy and unstructured documents (Chocolate_Confectionery_Retail_Policy.docx, emails, guidelines) using Azure AI Search vector + semantic retrieval.
- **Planogram Vision LLM Suite** (`planogram_vision_llm_suite`) — score 0.55
  - Matched for: Provide internal managers with fast, trustworthy KPI summaries and business answ
  - Azure OpenAI vision functions for shelf-level product extraction, row counting, generic row description, daily KPI JSON, and final natural-language shelf answers after Roboflow cropping.
- **Final Answer Consolidation Agent** (`final_answer_consolidation_agent`) — score 0.55
  - Matched for: Manager submits natural language question → Intent router classifies the questio
  - Consolidates count and generic row-level analyses into a single natural-language answer based on task_type routing.
- **Quin SQL Agent Chain** (`quin_sql_agent_chain`) — score 0.85
  - Matched for: sql-query-step
  - AutoGen multi-agent chain that generates, executes, and critiques SQL Server queries on mars schema tables (Mars_Sales_Data, shelf_visit, retail_planogram_stocks, etc.) and returns structured sales/inventory insights.

## Open questions

- Which structured database platforms must the live KPI query node support, and is SQL generation allowed or should it use only predefined semantic query templates?
- Should the shared enterprise knowledge base already exist as an indexed corpus, or must the solution also include ingestion and refresh pipelines for enterprise documents?
- What confidence threshold and conflict severity rules should trigger the 'I do not know' fallback versus a flagged answer with both sources shown?
- Are managers authenticated with role-based access controls that restrict which documents and KPI domains can be retrieved per user?

## Validation notes

Overall status: **warn**

- [warn] Catalog coverage for 'Manager Question Intake' is weak overall (top raw search score 0.0137).
- [warn] Catalog coverage for 'Intent Routing Gateway' is weak overall (top raw search score 0.0310).
- [warn] 'Intent Routing Gateway' may not match catalog agent capabilities (review fit).
- [warn] Catalog coverage for 'Enterprise Document Retrieval' is weak overall (top raw search score 0.0310).
- [warn] Catalog coverage for 'Semantic RAG Answering' is weak overall (top raw search score 0.0308).
- [warn] Catalog coverage for 'Conflict Detection and Confidence Gate' is weak overall (top raw search score 0.0310).
- [warn] Catalog coverage for 'Manager Answer Consolidation' is weak overall (top raw search score 0.0111).
- [warn] Catalog coverage for 'Citation and Evidence Delivery' is weak overall (top raw search score 0.0310).
- [warn] 'Citation and Evidence Delivery' may not match catalog agent capabilities (review fit).
- [warn] Which structured database platforms must the live KPI query node support, and is SQL generation allowed or should it use only predefined semantic query templates?

## Repository contents

| Path | Description |
|------|-------------|
| `README.md` | This overview |
| `workflow.json` | Full architecture graph, reuse decisions, and layout |
| `session.json` | Interview spec and session metadata (when available) |
| `agents/*.json` | Per-step scaffold files for implementation |

## Next steps

1. Review the architecture summary and agent steps above
2. Open `workflow.json` for the complete graph and reuse decisions
3. Implement each step under `agents/` using your runtime of choice
4. Wire integrations and HITL paths described in the requirements

---
*Generated by Agentic LaunchPad on 2026-07-05 18:52 UTC*