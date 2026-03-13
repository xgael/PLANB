
CHALLENGE DESCRIPTION

Scalable Multi-Tenant SaaS Platform with AI Integration

| ⏱<br><br>6 Hours<br><br>**Duration** | 👥<br><br>3 Devs<br><br>**Team Size** |
| ------------------------------------ | ------------------------------------- |

| **TiDB Services**<br><br>• TiDB Cloud Starter (Free Tier)<br><br>• Multi-Tenant Schema (DB per Tenant)<br><br>• TiFlash HTAP Replicas (analytics)<br><br>• RBAC (DB users and roles per tenant)<br><br>• Horizontal Scalability (elastic nodes)<br><br>• SQL Views (analytics feeds per tenant)<br><br>• TiDB Cloud Branching (beta, free) | **Alibaba Cloud Services**<br><br>• ECS (app backend hosting)<br><br>• VPC + vSwitch + Security Group<br><br>• Model Studio (Qwen-Plus API)<br><br>• Quick BI (analytics dashboards)<br><br>• RAM (Identity and Access Management) |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

Participants will implement a multi-tenant SaaS platform on Alibaba Cloud using TiDB as the scalable database backbone. The focus is on the database patterns that real SaaS companies need: tenant isolation, HTAP analytics separation, elastic horizontal scaling, and AI integration.

Teams choose one of two scenarios. Both share the same infrastructure requirements but differ in domain and data model. The scalability patterns are universal - inspired by how **Alibaba's AI** partnered with TiDB to solve the multiplicative scaling challenge of running thousands of concurrent AI agents across hundreds of tenants.

**1\. Challenge Scenarios**

**Scenario A: GameVault Corp**

_Multi-Tenant Video Game Storefront with AI Recommendations_

GameVault Corp provides white-label e-commerce infrastructure to independent video game stores. Each store is a tenant on a shared TiDB cluster, completely isolated in data and compute budget. Qwen provides AI-powered game recommendations by reasoning over each customer's purchase history queried directly from TiDB.

**The Problem**

GameVault powers 5 independent video game stores. Store owners need:

<div class="joplin-table-wrapper"><table><tbody><tr><th><ul><li>Data Isolation - Store A must never see Store B's customers, orders, or purchase history, even though they share infrastructure.</li></ul></th></tr><tr><td><ul><li>AI Recommendations - When a customer browses, the system should recommend games based on their personal purchase history, reasoned by an LLM, not just a rules engine.</li></ul></td></tr><tr><td><ul><li>Scalability - Adding a 6th, 10th, or 50th store should require zero architectural changes - just provision a new schema, user, and TiFlash replicas.</li></ul></td></tr><tr><td><ul><li>Analytics - Each store manager needs a dashboard showing sales by genre, top-selling titles, and revenue trends, scoped to their tenant only.</li></ul></td></tr></tbody></table></div>

**Table Structure (per store schema)**

| **Table**       | **Key Columns**                                                           | **Purpose**                                |
| --------------- | ------------------------------------------------------------------------- | ------------------------------------------ |
| games           | game_id (AUTO_RANDOM), title, genre, platform, price, release_date, stock | Per-store product catalog                  |
| customers       | customer_id, name, email, registered_at, last_active                      | Customer records scoped to each store      |
| orders          | order_id (AUTO_RANDOM), customer_id, total_amount, status, ordered_at     | Transaction records                        |
| order_items     | item_id, order_id, game_id, quantity, unit_price                          | Line items per order                       |
| recommendations | rec_id, customer_id, game_ids (JSON), qwen_reasoning (TEXT), generated_at | Stores Qwen output per recommendation call |

**Seed Data Requirements**

Each store schema must be populated with: 100+ games across 6 genres (Action, RPG, Strategy, Sports, Indie, Horror), 50+ customers, and 200+ orders with varied purchase patterns to make Qwen's recommendations meaningful.

**Scenario B: AgentNexus**

_Multi-Tenant Agentic AI Platform with Task Orchestration_

Real-World Inspiration: Alibaba's AI × TiDB

Alibaba's AI faced the X×Y×Z scaling problem: X tenants running Y agents, each exploring Z parallel branches simultaneously. They partnered with TiDB to build a database architecture that treats the database like code - with instant branching for isolated state, massive parallel agent swarms, and the database serving as each agent's long-term memory rather than just a system of record.

AgentNexus is a fictional SaaS platform that provides agentic AI infrastructure to enterprise customers. Each enterprise tenant deploys autonomous AI agents that perform multi-step tasks: research, code generation, data analysis, and web interaction. Agents need persistent state, isolated execution contexts, and a shared knowledge base - all backed by TiDB.

**The Problem**

AgentNexus serves 5 enterprise customers. Each customer's agents need:

<div class="joplin-table-wrapper"><table><tbody><tr><th><ul><li>Tenant Isolation - Enterprise A's agent sessions, execution logs, and knowledge base must be invisible to Enterprise B, even on shared infrastructure.</li></ul></th></tr><tr><td><ul><li>Agent State Persistence - Each agent session maintains structured state in TiDB: task plans, intermediate results, tool call history, and final outputs. The database is the agent's long-term memory.</li></ul></td></tr><tr><td><ul><li>Scalability Under Concurrency - When a tenant launches 50 agents simultaneously, the platform must handle the write amplification without degrading other tenants. TiFlash ensures analytical queries don't compete with agent write operations.</li></ul></td></tr><tr><td><ul><li>AI Orchestration - A meta-agent powered by Qwen analyzes completed task logs and recommends optimizations: better tool sequences, common failure patterns, and efficiency improvements.</li></ul></td></tr></tbody></table></div>

**Table Structure (per tenant schema)**

| **Table**       | **Key Columns**                                                                                   | **Purpose**                                 |
| --------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| agents          | agent_id, name, type, capabilities (JSON), created_at                                             | Registered agent definitions per tenant.    |
| sessions        | session_id (AUTO_RANDOM), agent_id, task_description, status, started_at, completed_at            | Each agent execution session                |
| state_snapshots | snapshot_id, session_id, step_number, state_data (JSON), created_at                               | Agent state at each step (long-term memory) |
| tool_calls      | call_id (AUTO_RANDOM), session_id, tool_name, input (JSON), output (JSON), duration_ms, called_at | Audit log of every tool invocation          |
| insights        | insight_id, tenant_id, qwen_analysis (TEXT), recommendations (JSON), generated_at                 | Qwen meta-analysis of agent performance     |

**Seed Data Requirements**

Each tenant schema must be populated with: 10+ agent definitions, 100+ completed sessions across agent types, 500+ state snapshots, and 1000+ tool call records with realistic timing data to give Qwen meaningful patterns to analyze.

**2\. System Architecture (Both Scenarios)**

Both scenarios share four logical layers that teams must build and connect:

| **Data Layer**        | TiDB Cloud: multi-tenant schemas, TiFlash HTAP replicas for analytics isolation, RBAC for access control, SQL views for analytics. The database scales horizontally - add TiKV/TiFlash nodes as tenants grow. |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Application Layer** | ECS (Python/Node.js): REST API managing tenant operations, routed by tenant_id. One service, multi-tenant routing using per-tenant database users.                                                            |
| **AI Layer**          | Alibaba Cloud Model Studio (Qwen-Plus): Receives structured context from the app, returns intelligent responses. GameVault: game recommendations. AgentNexus: agent performance insights.                     |
| **Analytics Layer**   | Quick BI: Connected to TiDB Cloud via public endpoint (TLS). Reads from per-tenant SQL views. Parameterized dashboards with row-level tenant isolation enforced at the TiDB layer.                            |

**2.1 Network Architecture**

There are two distinct connection paths teams must understand:

| ECS ↔ TiDB Cloud (Application Traffic)<br><br>ECS instances live inside a VPC. They connect to TiDB Cloud's public endpoint via TLS (port 4000) with ECS's public IP allowlisted in TiDB Cloud firewall rules. The Security Group on ECS allows outbound TCP 4000 to TiDB Cloud. The app connects using a per-tenant database user, ensuring SQL queries are automatically scoped to the correct schema.                                                      |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Quick BI ↔ TiDB Cloud (Analytics Traffic)<br><br>Quick BI is a managed SaaS service: it does NOT reside inside your VPC. Quick BI connects to TiDB Cloud via the public endpoint over TLS. Before configuring the Quick BI data source, teams must add Quick BI's egress IP addresses to TiDB Cloud's IP access list (allowlist). A dedicated read-only TiDB user is created for Quick BI with SELECT permissions only on the SQL views, never on raw tables. |
| ECS ↔ Model Studio (Qwen API Traffic)<br><br>Qwen API is called via HTTPS from ECS - no VPC configuration needed. Teams activate Model Studio, generate a DashScope API key, and set it as an environment variable on ECS. Base URL: <https://dashscope-intl.aliyuncs.com/compatible-mode/v1> (Singapore region). Model: qwen-plus (cost-effective, solid reasoning). The API is OpenAI SDK-compatible.                                                       |

**3\. TiDB Cloud - Scalability-First Configuration**

Both This challenge is structured around four core TiDB capabilities that real SaaS platforms need at scale. Teams will implement all four during the hackathon.

| **Module**                          | **TiDB Feature**                                    | **Challenge Application**                                                                                                                                  |
| ----------------------------------- | --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Module 1: Rapid Schema Provisioning | Rapidly create a large number of schemas and tables | Provision 5 tenant databases with identical table structures in a single script. Demonstrate adding a 6th tenant in under 60 seconds.                      |
| Module 2: Cross-Schema Plan Binding | Execution plan binding across schemas               | Bind optimized execution plans so that identical queries across tenant schemas use the same efficient plan, preventing plan regression.                    |
| Module 3: TiFlash HTAP Analytics    | TiFlash columnar replicas for real-time analytics   | Enable TiFlash replicas on analytical tables. Show that Quick BI queries route to TiFlash (cop\[tiflash\] in EXPLAIN) while transactions continue on TiKV. |
| Module 4: Online Schema Change      | Non-blocking DDL for zero-downtime migrations       | Add a column to a table in one tenant schema while another tenant is actively writing. Show zero downtime.                                                 |

**3.1 Module 1: Rapid Multi-Tenant Schema Provisioning**

Teams must implement the database-per-tenant model. Each tenant gets its own TiDB database (schema), isolating data at the SQL level without requiring separate clusters. This is the same pattern used by SaaS companies scaling from 5 to 5,000 tenants on TiDB.

The key insight: TiDB can handle thousands of schemas on a single cluster. Teams should write an automated provisioning script that creates a new tenant end-to-end: database, tables, user, resource group, and SQL views. During the demo, teams will show they can onboard a 6th tenant in under 60 seconds.

Performance tip: Use AUTO_RANDOM instead of AUTO_INCREMENT on high-write primary keys (orders, sessions, tool_calls). AUTO_RANDOM distributes inserts across TiKV regions, preventing write hotspots that occur when all tenants generate sequential IDs on the same node. This becomes critical as the number of concurrent tenants grows.

Schema Naming Convention

\-- GameVault: CREATE DATABASE store_alpha; CREATE DATABASE store_beta; ...

\-- AgentNexus: CREATE DATABASE tenant_acme; CREATE DATABASE tenant_globex; ...

\-- Shared platform database for cross-tenant metadata

CREATE DATABASE platform;

\-- Provisioning script should automate: CREATE DATABASE, CREATE TABLE (x5),

\-- CREATE USER (with cluster prefix), GRANT, CREATE VIEW (x3)

\-- Use AUTO_RANDOM on high-write primary keys (orders, sessions, tool_calls),

\-- ALTER TABLE SET TIFLASH REPLICA 1 (on analytical tables)

**3.2 Module 3: TiFlash HTAP - Analytics Without Impacting Transactions**

**TiFlash: Columnar Analytics Engine** TiFlash is TiDB's columnar storage engine. When enabled on a table, TiDB automatically maintains a columnar replica alongside the row-based TiKV data. Analytical queries (aggregations, JOINs, GROUP BY) are routed to TiFlash, while transactional writes continue on TiKV. This means Quick BI dashboards can query live data without degrading purchase/session processing performance.

Teams must enable TiFlash on analytical tables and verify that queries route to the columnar engine using EXPLAIN:

\-- Enable TiFlash replicas on analytical tables (per tenant schema)

ALTER TABLE store_alpha.games SET TIFLASH REPLICA 1;

ALTER TABLE store_alpha.orders SET TIFLASH REPLICA 1;

ALTER TABLE store_alpha.order_items SET TIFLASH REPLICA 1;

\-- Verify replica sync status (wait for AVAILABLE = 1)

SELECT TABLE_NAME, REPLICA_COUNT, AVAILABLE

FROM information_schema.tiflash_replica

WHERE TABLE_SCHEMA = 'store_alpha';

\-- Verify analytical queries route to TiFlash

EXPLAIN SELECT g.genre, COUNT(\*) FROM order_items oi

JOIN games g ON oi.game_id = g.game_id GROUP BY g.genre;

\-- Look for cop\[tiflash\] in the output

**Why This Matters for SaaS**

Without HTAP separation, running a heavy GROUP BY query for a dashboard could lock rows and slow down customer transactions. TiFlash eliminates this conflict by serving analytics from a separate columnar engine, keeping the row store free for writes.

**3.3 Module 2: Cross-Schema Execution Plan Binding**

In a multi-tenant SaaS database, every tenant runs the same queries against identically structured tables. Without plan binding, TiDB's optimizer might choose different (and sometimes suboptimal) execution plans for different tenant schemas, causing unpredictable performance.

Cross-schema plan binding solves this: teams bind an optimized execution plan once, and TiDB reuses it across all tenant schemas with the same table structure. This ensures consistent, predictable query performance regardless of which tenant is querying.

Cross-Schema Plan Binding (TiDB 7.x+)

\-- Step 1: Identify the optimal plan for a common query

EXPLAIN SELECT g.genre, COUNT(\*) FROM order_items oi

JOIN games g ON oi.game_id = g.game_id GROUP BY g.genre;

\-- Step 2: Create a cross-schema binding (available in TiDB 7.x+)

CREATE GLOBAL BINDING FOR

SELECT g.genre, COUNT(\*) FROM order_items oi

JOIN games g ON oi.game_id = g.game_id GROUP BY g.genre

USING

SELECT /\*+ HASH_AGG() USE_INDEX(oi, idx_game_id) \*/ g.genre, COUNT(\*)

FROM order_items oi JOIN games g ON oi.game_id = g.game_id GROUP BY g.genre;

**Why This Matters for SaaS**

Without plan binding, adding more tenants with different data distributions can cause plan regression - a query that was fast for Tenant A might be slow for Tenant B. Cross-schema binding eliminates this unpredictability, which is critical for SaaS platforms with SLA commitments.

**3.4 Module 4: Online Schema Change**

SaaS platforms must evolve their schema without downtime. TiDB supports non-blocking DDL operations, meaning teams can add columns, create indexes, or modify tables while tenants are actively reading and writing data.

Teams must demonstrate this during the demo: add a new column to one tenant's table while another tenant is actively inserting rows. Show that neither tenant experiences downtime or errors.

Online Schema Change Demo

\-- While tenant_alpha is actively receiving writes:

ALTER TABLE tenant_alpha.orders ADD COLUMN discount_pct DECIMAL(5,2) DEFAULT 0.00;

\-- Verify: the write stream from the application continues uninterrupted

\-- Verify: existing rows have the default value, new rows can use the column

**3.5 Horizontal Scalability**

TiDB's architecture separates compute (TiDB nodes) from storage (TiKV nodes). As the number of tenants grows, teams can scale horizontally by adding nodes - no application changes required. This is fundamentally different from traditional databases where scaling means bigger machines. Teams should be able to explain this architecture during the demo.

| **TiDB nodes**    | Stateless SQL processing. Add more for increased concurrent connections.                                                        |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **TiKV nodes**    | Distributed key-value storage with automatic data sharding across regions. Add more for increased storage and write throughput. |
| **TiFlash nodes** | Columnar replicas for analytical queries. Useful for Quick BI dashboards but not the primary focus of this challenge.           |

**3.6 RBAC - Access Control per Tenant**

Every tenant must have a dedicated database user with permissions scoped only to its own schema. The admin user is reserved for platform operations and must never be used by the application or Quick BI.

RBAC Configuration

\-- Note: On TiDB Cloud Starter, usernames require a cluster prefix

\-- Example: CREATE USER '{prefix}.user_tenant_alpha'@'%' IDENTIFIED BY '&lt;password&gt;';

\-- GRANT SELECT, INSERT, UPDATE ON tenant_alpha.\* TO '{prefix}.user_tenant_alpha'@'%';

\-- Verify isolation: logged in as {prefix}.user_tenant_alpha

SHOW DATABASES; -- Should only show tenant_alpha

SELECT \* FROM tenant_beta.orders; -- Should return ACCESS DENIED

**3.7 SQL Views for Quick BI (Per-Tenant Analytics)**

Quick BI should not query raw tables directly. Per-store SQL views encapsulate the analytics logic and enforce tenant isolation at the database layer.

**GameVault views:**

GameVault views:

CREATE VIEW store_alpha.sales_by_genre AS

SELECT g.genre, COUNT(oi.item_id) AS units_sold,

SUM(oi.unit_price \* oi.quantity) AS revenue

FROM order_items oi JOIN games g ON oi.game_id = g.game_id

GROUP BY g.genre;

CREATE VIEW store_alpha.top_titles_this_week AS

SELECT g.title, g.genre, COUNT(\*) AS purchases

FROM orders o JOIN order_items oi ON o.order_id = oi.order_id

JOIN games g ON oi.game_id = g.game_id

WHERE o.ordered_at >= NOW() - INTERVAL 7 DAY

GROUP BY g.title, g.genre ORDER BY purchases DESC LIMIT 10;

CREATE VIEW store_alpha.daily_revenue AS

SELECT DATE(ordered_at) AS sale_date, SUM(total_amount) AS revenue

FROM orders GROUP BY DATE(ordered_at) ORDER BY sale_date;

**AgentNexus views:**

AgentNexus views:

CREATE VIEW tenant_acme.agent_performance AS

SELECT a.name, a.type, COUNT(s.session_id) AS total_sessions,

AVG(TIMESTAMPDIFF(SECOND, s.started_at, s.completed_at)) AS avg_duration_sec,

SUM(CASE WHEN s.status = 'completed' THEN 1 ELSE 0 END) AS successes

FROM agents a JOIN sessions s ON a.agent_id = s.agent_id

GROUP BY a.name, a.type;

CREATE VIEW tenant_acme.tool_usage_stats AS

SELECT tool_name, COUNT(\*) AS call_count, AVG(duration_ms) AS avg_ms

FROM tool_calls GROUP BY tool_name ORDER BY call_count DESC;

CREATE VIEW tenant_acme.daily_sessions AS

SELECT DATE(started_at) AS day, COUNT(\*) AS sessions,

SUM(CASE WHEN status='completed' THEN 1 ELSE 0 END) AS completed

FROM sessions GROUP BY DATE(started_at) ORDER BY day;

**4\. Alibaba Cloud - Infrastructure Configuration**

**4.1 VPC, vSwitch, and Security Group**

| **VPC**            | CIDR block 10.0.0.0/16. One VPC per team in the selected region (Singapore recommended for international access to Qwen API and TiDB Cloud).                             |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **vSwitch**        | Subnet within the VPC, e.g. 10.0.1.0/24, in the availability zone where ECS will be deployed.                                                                            |
| **Security Group** | Inbound: allow TCP 22 (SSH, from team IPs only), TCP 80/443 (app API). Outbound: allow TCP 4000 (TiDB Cloud public endpoint), TCP 443 (Model Studio HTTPS for Qwen API). |
| **ECS Instance**   | Deploy the application backend. Ubuntu 22.04, 4 vCPU / 8 GB RAM (ecs.c7.xlarge or similar). Assign an Elastic IP for inbound access. Python 3.11 or Node.js 20 runtime.  |
| **RAM (IAM)**      | Create a RAM user for the team with scoped permissions: ECS access, Model Studio API key management. Avoid using root account credentials.                               |

**4.2 Model Studio - Qwen Integration**

Alibaba Cloud Model Studio provides access to Qwen models via an OpenAI-compatible API. No VPC configuration needed - it is a pure HTTPS API call from the ECS backend.

Setup Checklist

1\. In the Alibaba Cloud console, navigate to Model Studio and activate the service (accept Terms of Service).

2\. Go to Key Management and click Create API Key. Store it securely: it will be set as the DASHSCOPE_API_KEY environment variable on ECS.

3\. The recommended model is qwen-plus: great reasoning capability at lower cost than qwen-max, within a hackathon budget.

4\. Base URL for Singapore region: <https://dashscope-intl.aliyuncs.com/compatible-mode/v1>. The API is fully compatible with the OpenAI Python SDK: only change api_key and base_url.

**Integration Pattern (Both Scenarios)**

The app must NOT pass raw SQL dumps to Qwen. The correct pattern is:

- **1\.** Query TiDB for structured context (last 20 purchases or last 50 tool calls).
- **2\.** Format as a clean JSON context payload.
- **3\.** Send to Qwen with a carefully crafted system prompt.
- **4\.** Parse Qwen's JSON response and store it in TiDB.

**GameVault prompt pattern:**

GameVault prompt pattern:

System: 'You are a video game recommendation engine. You will receive a customer's purchase history as JSON. Respond ONLY with a JSON object: {"recommendations": \[{"title": "...", "genre": "...", "reason": "..."}\], "summary": "brief overall insight"}. No other text.'

**AgentNexus prompt pattern:**

AgentNexus prompt pattern:

System: 'You are an AI operations analyst. You will receive an enterprise tenant's agent execution logs as JSON. Analyze tool usage patterns, common failure modes, and session efficiency. Respond ONLY with a JSON object: {"patterns": \[...\], "failures": \[...\], "optimizations": \[...\], "summary": "brief overall insight"}. No other text.'

**4.3 Quick BI - Dashboard Configuration**

Quick BI connects to TiDB Cloud over the internet via the public endpoint. Four mandatory steps before building charts:

| ①                                                                                                                                                                                                                                                                                                                                         | Add Quick BI IPs to TiDB Cloud allowlist: In the TiDB Cloud console, add Quick BI's egress IP addresses to the IP access list.         |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| ②                                                                                                                                                                                                                                                                                                                                         | Create a read-only TiDB user for Quick BI: GRANT SELECT only on views, not raw tables. Use the cluster prefix format for the username. |
| ③                                                                                                                                                                                                                                                                                                                                         | Add TiDB as a data source in Quick BI: Enter TiDB Cloud public endpoint, port 4000, quickbi_reader credentials. Test connection.       |
| ④                                                                                                                                                                                                                                                                                                                                         | Build datasets on SQL views: One Quick BI dataset per view, per tenant. This enforces tenant isolation at the BI layer.                |
| **Critical - Tenant Isolation in Quick BI**<br><br>Quick BI does NOT enforce row-level isolation automatically. Tenant isolation MUST be enforced at the TiDB layer using schema-scoped SQL views. The correct architecture: one Quick BI dataset per tenant view. If a team queries raw tables across stores, all managers see all data. |                                                                                                                                        |

**5\. Deliverables**

**5.1 TiDB Cloud Deliverables**

- **\[Module 1\] Rapid Schema Provisioning** 5 tenant databases created with full table structures. An automated provisioning script that can add a 6th tenant in under 60 seconds.
- **\[Module 2\] Cross-Schema Plan Binding** At least one GLOBAL BINDING created for a common analytical query. Demonstrate that the same plan is reused across tenant schemas.
- **\[Module 3\] TiFlash HTAP** TiFlash replicas enabled on analytical tables across all tenants. EXPLAIN output shows cop\[tiflash\] for at least one analytical view query. Quick BI queries confirmed to route through TiFlash.
- **\[Module 4\] Online Schema Change** Demonstrate adding a column to a tenant table while another tenant is actively writing. Zero downtime confirmed.
- **RBAC verification** Demonstrate that user_tenant_alpha cannot SELECT from tenant_beta tables.
- **SQL Views** At least 3 analytical views per tenant, serving as Quick BI data sources.
- **Scalability explanation** Teams must explain TiDB's horizontal scaling architecture and how to onboard 50+ tenants.

**5.2 Alibaba Cloud ECS - Application Backend**

- **VPC + Security Group** VPC created with correct CIDR, vSwitch in deployment AZ, Security Group with scoped rules.
- **ECS Instance** App deployed and running. Accessible via public IP. Environment variables set: TIDB_HOST, TIDB_PORT, TIDB_USER, TIDB_PASSWORD (per tenant), DASHSCOPE_API_KEY.
- **Write API Endpoint** GameVault: POST /api/{store_id}/purchase. AgentNexus: POST /api/{tenant_id}/sessions.
- **AI API Endpoint** GameVault: GET /api/{store_id}/recommend/{customer_id}. AgentNexus: GET /api/{tenant_id}/insights.
- **Seed data script** Script that populates each tenant schema with realistic data as specified per scenario.

**5.3 Model Studio - Qwen Integration (15 pts)**

**CHOOSE ONE (or BOTH) OF THE NEXT TASKS TO GET EXTRA POINTS**

- **API key configured** Model Studio activated, DashScope API key as environment variable. No hardcoded keys.
- **RAG function** Queries TiDB for structured context, formats as JSON, sends to Qwen-Plus, parses response, stores in TiDB.
- **Response quality** Qwen must produce coherent, contextually relevant output. Judges evaluate reasoning quality against the input data.

**5.4 Quick BI - Analytics Dashboard (10 pts)**

- **TiDB data source connected** Connection test passes using quickbi_reader user.
- **Three datasets per tenant** Built on the three SQL views for at least one tenant.
- **Dashboard with 3 live charts** GameVault: sales by genre bar chart, top titles ranking, daily revenue trend. AgentNexus: agent performance by type, tool usage ranking, daily sessions trend.
- **Tenant isolation test** Demonstrate that Tenant A's dashboard shows ONLY Tenant A's data. Insert data in Tenant B during demo and show it does NOT appear in Tenant A's dashboard.

**6\. Evaluation Rubric**

Total: 100 points. Items marked YES in Mandatory are binary: if not demonstrated, the team cannot exceed 60 points regardless of others.

| **Criterion**                         | **What Judges Evaluate**                                                                                                                    | **Pts** | **Mandatory** |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ------- | ------------- |
| \[M1\] Rapid Provisioning & Isolation | 5 tenant schemas created; provisioning script adds 6th tenant in <60s; RBAC verified (ACCESS DENIED cross-tenant).                          | 30      | YES           |
| \[M3\] TiFlash HTAP Verified          | TiFlash replicas enabled on analytical tables. EXPLAIN shows cop\[tiflash\]. Team explains HTAP separation benefit for SaaS analytics.      | 30      | YES           |
| \[M2\] Cross-Schema Plan Binding      | At least one GLOBAL BINDING created. Team shows same plan reused across tenant schemas. Explains why this matters for SaaS.                 | 20      | NO            |
| \[M4\] Online Schema Change           | Column added to one tenant while another is actively writing. Zero downtime confirmed.                                                      | 20      | NO            |
| Qwen AI Quality                       | Recommendations/insights are contextually coherent with input data. Qwen output is stored in TiDB. JSON response is correctly parsed.       | 15      | NO            |
| Quick BI Dashboard                    | 3 charts present; data is live (reflects demo inserts); tenant data isolation visually verified by judges.                                  | 10      | NO            |
| Write API Correctness                 | POST endpoint correctly writes to the correct tenant schema using the tenant DB user. An invalid tenant_id returns a 4xx error.             | 10      | NO            |
| Code & Architecture Quality           | Clean routing logic; no hardcoded credentials; automated provisioning; realistic seed data; connection pool usage; scalability explanation. | 10      | NO            |

**CHOOSE ONE (or MORE) OF THE NEXT TASKS TO GET EXTRA POINTS**

| **Criterion**               | **What Judges Evaluate**                                                                                          | **Pts** | **Mandatory** |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------- | ------------- |
| Qwen AI Quality             | Recommendations/insights are contextually coherent with input data. Output stored in TiDB. JSON correctly parsed. | 15      | NO            |
| Quick BI Dashboard          | 3 charts present; data is live (reflects demo inserts); tenant isolation visually verified.                       | 10      | NO            |
| Write API Correctness       | POST endpoint writes to correct tenant schema using tenant DB user. Invalid tenant_id returns 4xx.                | 10      | NO            |
| Code & Architecture Quality | Clean routing; no hardcoded credentials; automated provisioning; connection pool; scalability explanation.        | 10      | NO            |

**7\. Demo Script (5 Minutes)**

Judges will follow this script. Teams must rehearse each step and have commands ready. The demo must be live: no pre-recorded videos.

| **Time**  | **Phase**                             | **What Teams Do**                                                                                                                                                                                                  |
| --------- | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 0:00-0:30 | Architecture Presentation             | The team presents a 1-slide architecture diagram showing TiDB schemas, ECS, Qwen, and Quick BI. Explain chosen scenario and the 4 TiDB Lab modules.                                                                |
| 0:30-1:15 | Isolation + Resource Groups \[M1+M3\] | SHOW DATABASES while logged in as user_tenant_alpha. Show that only the tenant's own database is visible. Attempt SELECT on another tenant: show ACCESS DENIED. Show Resource Group binding and RU_PER_SEC limits. |
| 1:15-2:00 | Live Write + AI                       | GameVault: POST /api/store_alpha/purchase then GET /api/store_alpha/recommend/{customer_id}. AgentNexus: POST session then GET insights. Show the raw Qwen JSON response: point out the contextual reasoning.      |
| 2:00-2:45 | Provisioning \[M1\]                   | Run the automated provisioning script live to add a 6th tenant in under 60 seconds. Show the new database, user, resource group, and views are all created.                                                        |
| 2:45-3:15 | Online DDL \[M4\]                     | Start a write stream to one tenant. Run ALTER TABLE ADD COLUMN on that tenant's table. Show writes continue uninterrupted. Show the column exists with default values.                                             |
| 3:15-4:00 | Quick BI + Plan Binding \[M2\]        | Open the Quick BI dashboard. Show 3 charts. Refresh after the live write: charts update. Show tenant B's dashboard does NOT show tenant A's data. Show GLOBAL BINDING is active (EXPLAIN output).                  |
| 4:00-5:00 | Q&A                                   | Judges ask: 'What happens if a tenant's Resource Group is exhausted?' / 'How does plan binding help at 500 tenants?' / 'Why did you choose qwen-plus over qwen-max?' / 'How would you add a 50th tenant?'          |

**8\. Facilitator Notes and Pre-Event Checklist**

**8.1 Pre-Event Infrastructure**

- **TiDB Cloud Starter** Provision a TiDB Cloud Starter (Free Tier) cluster on Alibaba Cloud. Distribute connection strings and a master admin user to all teams. Up to 5 instances per organization.
- **Quick BI IPs** Look up Quick BI egress IPs for the event region. Pre-add them to the TiDB Cloud IP access list.
- **Model Studio pre-activated** Activate Model Studio and pre-create one API key per team. Confirm qwen-plus is accessible from the Singapore endpoint.
- **Seed data generator** Prepare and test seed data SQL scripts for both scenarios. Teams should run these scripts, not write them from scratch.
- **TiFlash guidance** Inform teams: after ALTER TABLE SET TIFLASH REPLICA 1, wait 2-5 minutes for AVAILABLE = 1 before querying. Check status with information_schema.tiflash_replica.

**8.2 Common Mistakes to Watch For**

- **Quick BI IP not allowlisted** The most common "connection refused" problem. Pre-add the IPs before the event.
- **Qwen returns free text instead of JSON** Teams forget to instruct Qwen to respond only in JSON. Fix: reinforce the system prompt with "Respond ONLY with valid JSON."
- **Connecting Quick BI to raw tables** If teams build datasets with SELECT \* FROM orders (raw table), they see all tenants' data. Redirect to SQL views.
- **Using admin user for Quick BI** Exposes all schemas. Enforce the dedicated quickbi_reader user.
- **TiFlash not ready** Teams enable TiFlash and immediately query before AVAILABLE = 1. Incorporate a wait step in setup instructions.
- **Wrong username format** On TiDB Starter, all usernames require the cluster prefix. Teams forgetting the prefix will get 'User name must start with…' errors.

**8.3 Stretch Goals (for teams that finish early)**

These stretch goals are organized in three tiers, each teaching a distinct TiDB capability. Teams can attempt any tier independently, but we recommend this progression:

**Tier 1: Distributed Fundamentals**

- **AUTO_RANDOM on primary keys** Use AUTO_RANDOM instead of AUTO_INCREMENT on high-write tables (orders, sessions, tool_calls). Explain how this eliminates write hotspots by distributing inserts across TiKV regions. Compare EXPLAIN output before and after.
- **RU awareness via EXPLAIN ANALYZE** Run EXPLAIN ANALYZE on your key queries and report the Request Unit (RU) cost of each. Identify which queries are most expensive and explain how you would optimize them in a production SaaS with per-tenant cost attribution.
- **TiFlash vs TiKV performance comparison** Run the same analytical query with and without TiFlash (SET SESSION tidb_isolation_read_engines='tikv' vs default). Compare EXPLAIN output and execution times.
- **Plan binding performance comparison** Run the same query with and without the GLOBAL BINDING. Compare EXPLAIN output and execution times to demonstrate the impact on SaaS query consistency.

**Tier 2: Vector Search with Qwen Embeddings**

- **Generate embeddings with Qwen** Instead of sending plain text to Qwen for recommendations, use Qwen's embedding API to generate vector representations of products (GameVault) or agent capabilities (AgentNexus). Store the vectors in TiDB using the VECTOR column type.
- **Semantic similarity search** Implement a similarity search endpoint: given a product description or agent task, find the most similar items using TiDB's built-in vector distance functions (VEC_COSINE_DISTANCE). This replaces keyword matching with AI-powered semantic understanding.
- **Hybrid query** Combine vector similarity with traditional SQL filters (e.g., find games similar to "open-world RPG" but only in the Action genre and under \$30). Show how TiDB handles both structured and vector queries in a single statement.

**Tier 3: TiDB Cloud Branching (Inspired by Alibaba's AI)**

**Database as a Forkable Workspace**

Alibaba's AI treats the database like code - agents can instantly branch the database to test risky hypotheses in isolation, compare results across branches, and continue from the best path. TiDB Cloud Branching is available on Starter (Free) with up to 5 branches per organization.

- **Create a branch for schema migration testing** Before applying a schema change to production, create a TiDB Cloud branch. Apply the DDL on the branch, run your test suite against it, verify data integrity, then apply to the main cluster with confidence.
- **Parallel approach comparison** Create 2-3 branches from the same baseline. On each branch, try a different indexing strategy or table structure. Run identical analytical queries on each branch and compare performance. Choose the best approach and explain your reasoning.
- **Agent sandbox (AgentNexus only)** Use a branch as an isolated execution sandbox for an agent session. The agent can freely modify data on the branch without risking the production database. This mirrors how Alibaba's agents use TiDB branching for the X×Y×Z scaling problem.
- **Mass tenant provisioning** Extend the provisioning script to create 20 tenants in a single run. Benchmark the total provisioning time and show all 20 are independently functional.

**9\. Complete Service Map**

Each service used in this challenge, its role, and the specific required configuration:

**TiDB Cloud**

| **Service**                      | **Purpose in Challenge**                                       | **Key Configuration**                                                 |
| -------------------------------- | -------------------------------------------------------------- | --------------------------------------------------------------------- |
| TiDB Cloud Starter (Free)        | Hosts all tenant schemas and platform schema in one cluster    | Multi-node cluster for horizontal scalability; TiFlash nodes optional |
| Rapid Schema Provisioning \[M1\] | Create tenant databases and tables at scale                    | Automated script: CREATE DATABASE + tables + user + RG + views        |
| Cross-Schema Plan Binding \[M2\] | Consistent query plans across tenant schemas                   | CREATE GLOBAL BINDING for common analytical queries                   |
| TiFlash HTAP \[M3\]              | Columnar replicas for analytics without impacting transactions | ALTER TABLE ... SET TIFLASH REPLICA 1; verify with EXPLAIN            |
| Online Schema Change \[M4\]      | Zero-downtime DDL migrations across tenants                    | ALTER TABLE ... ADD COLUMN while writes continue                      |
| RBAC (Users & Permissions)       | Per-tenant DB user scoped to the correct schema only           | CREATE USER + GRANT SELECT/INSERT/UPDATE ON tenant_x.\*               |
| SQL Views                        | Per-tenant analytics feeds for Quick BI                        | CREATE VIEW per analytical KPI per tenant schema                      |
| AUTO_RANDOM (Stretch)            | Distributed primary keys to eliminate write hotspots           | ALTER TABLE ... AUTO_RANDOM(5) on BIGINT PK                           |
| Vector Search (Stretch)          | Semantic similarity on embeddings stored in TiDB               | VECTOR column type + VEC_COSINE_DISTANCE()                            |
| Cloud Branching (Stretch)        | Isolated database copies for safe experimentation              | Create branch in TiDB Cloud console; 5 max per org                    |

**Alibaba Cloud**

| **Service**    | **Purpose in Challenge**                         | **Key Configuration**                                                          |
| -------------- | ------------------------------------------------ | ------------------------------------------------------------------------------ |
| ECS            | Hosts the Python/Node.js application backend     | Ubuntu 22.04, 4vCPU/8GB, Elastic IP assigned                                   |
| VPC            | Private network for ECS isolation                | CIDR 10.0.0.0/16, one vSwitch per availability zone                            |
| Security Group | Firewall rules for ECS                           | Outbound: TCP 4000 (TiDB), TCP 443 (Qwen API)                                  |
| Model Studio   | Qwen-Plus LLM API for AI integration             | Activate service → create API key → set DASHSCOPE_API_KEY environment variable |
| Quick BI       | No-code analytics dashboards for tenant managers | Connect via TiDB public endpoint; add Quick BI IPs to TiDB access list         |
| RAM (IAM)      | Access management for team members               | Create RAM users; avoid using root account                                     |