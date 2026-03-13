# AgentNexus - Architecture Document

## Scenario B: Multi-Tenant Agentic AI Platform with Task Orchestration

---

## 1. System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CLIENTS / DEMO                               │
│                   (curl, Postman, Browser)                           │
└──────────────────────────┬──────────────────────────────────────────┘
                           │ HTTPS (port 80/443)
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    ALIBABA CLOUD VPC (10.0.0.0/16)                  │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │              ECS Instance (Ubuntu 22.04, 4vCPU/8GB)           │  │
│  │                                                               │  │
│  │  ┌─────────────────────────────────────────────────────────┐  │  │
│  │  │            Python FastAPI Application                    │  │  │
│  │  │                                                         │  │  │
│  │  │  ┌──────────┐  ┌──────────┐  ┌───────────────────────┐ │  │  │
│  │  │  │  Tenant   │  │   AI     │  │    Provisioning       │ │  │  │
│  │  │  │  Router   │  │  Service │  │    Service            │ │  │  │
│  │  │  │          │  │ (Qwen)   │  │                       │ │  │  │
│  │  │  └────┬─────┘  └────┬─────┘  └───────────┬───────────┘ │  │  │
│  │  │       │              │                     │             │  │  │
│  │  │  ┌────▼──────────────▼─────────────────────▼───────────┐ │  │  │
│  │  │  │           Connection Pool Manager                   │ │  │  │
│  │  │  │     (per-tenant DB user connections)                │ │  │  │
│  │  │  └────────────────────┬────────────────────────────────┘ │  │  │
│  │  └───────────────────────┼─────────────────────────────────┘  │  │
│  └──────────────────────────┼────────────────────────────────────┘  │
│                             │ TCP 4000 (TLS)                        │
└─────────────────────────────┼───────────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────────┐
          │                   │                       │
          ▼                   ▼                       ▼
┌──────────────┐  ┌────────────────────┐  ┌──────────────────┐
│  TiDB Cloud  │  │  Alibaba Model     │  │    Quick BI      │
│  Starter     │  │  Studio            │  │    (SaaS)        │
│              │  │  (Qwen-Plus API)   │  │                  │
│ ┌──────────┐ │  │                    │  │  Connects via    │
│ │tenant_   │ │  │  HTTPS :443        │  │  TiDB public     │
│ │acme      │ │  │  dashscope-intl    │  │  endpoint        │
│ ├──────────┤ │  │  .aliyuncs.com     │  │  (port 4000)     │
│ │tenant_   │ │  │                    │  │                  │
│ │globex    │ │  └────────────────────┘  │  Read-only user  │
│ ├──────────┤ │                          │  on SQL Views    │
│ │tenant_   │ │                          └──────────────────┘
│ │initech   │ │
│ ├──────────┤ │
│ │tenant_   │ │
│ │umbrella  │ │
│ ├──────────┤ │
│ │tenant_   │ │
│ │wayne     │ │
│ ├──────────┤ │
│ │platform  │ │  (cross-tenant metadata)
│ └──────────┘ │
└──────────────┘
```

---

## 2. Four Layers Architecture

### Layer 1: Data Layer (TiDB Cloud Starter)

| Component | Detail |
|-----------|--------|
| **Cluster** | TiDB Cloud Starter (Free Tier) on Alibaba Cloud Singapore |
| **Multi-tenancy** | Database-per-tenant model (5 schemas + 1 platform schema) |
| **HTAP** | TiFlash replicas on `sessions`, `tool_calls`, `state_snapshots` |
| **RBAC** | Per-tenant DB user with scoped permissions |
| **Views** | 3 analytical views per tenant for Quick BI |
| **Keys** | AUTO_RANDOM on high-write PKs (`sessions`, `tool_calls`) |

### Layer 2: Application Layer (ECS + FastAPI)

| Component | Detail |
|-----------|--------|
| **Runtime** | Python 3.11 + FastAPI + uvicorn |
| **Hosting** | ECS instance (ecs.c7.xlarge, 4vCPU/8GB, Ubuntu 22.04) |
| **Network** | VPC 10.0.0.0/16, vSwitch 10.0.1.0/24, Elastic IP |
| **Routing** | Tenant-aware routing via `{tenant_id}` path parameter |
| **DB Driver** | `mysql-connector-python` with connection pooling per tenant |
| **Security** | No hardcoded credentials, env vars for all secrets |

### Layer 3: AI Layer (Qwen-Plus via Model Studio)

| Component | Detail |
|-----------|--------|
| **Model** | `qwen-plus` (cost-effective, solid reasoning) |
| **SDK** | OpenAI Python SDK (compatible drop-in) |
| **Base URL** | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` |
| **Pattern** | Query TiDB → Format JSON → Send to Qwen → Parse → Store in TiDB |

### Layer 4: Analytics Layer (Quick BI)

| Component | Detail |
|-----------|--------|
| **Connection** | TiDB Cloud public endpoint via TLS (port 4000) |
| **User** | Dedicated `quickbi_reader` with SELECT on views only |
| **Datasets** | 1 dataset per view per tenant (tenant isolation at DB layer) |
| **Dashboards** | 3 charts: agent performance, tool usage, daily sessions |

---

## 3. Data Layer - TiDB Schema Design

### 3.1 Tenant Naming Convention

```
tenant_acme      -- Tenant 1: Acme Corp
tenant_globex    -- Tenant 2: Globex Corporation
tenant_initech   -- Tenant 3: Initech
tenant_umbrella  -- Tenant 4: Umbrella Corp
tenant_wayne     -- Tenant 5: Wayne Enterprises
platform         -- Shared metadata
```

### 3.2 Table Structure (per tenant schema)

```sql
-- ================================================================
-- TENANT SCHEMA: tenant_acme (replicated for each tenant)
-- ================================================================

CREATE TABLE agents (
    agent_id      INT PRIMARY KEY AUTO_INCREMENT,
    name          VARCHAR(100) NOT NULL,
    type          VARCHAR(50) NOT NULL,       -- 'researcher', 'coder', 'analyst', 'web_navigator', 'orchestrator'
    capabilities  JSON,                        -- {"languages": ["python","js"], "tools": ["search","code_exec"]}
    created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_type (type)
);

CREATE TABLE sessions (
    session_id       BIGINT PRIMARY KEY AUTO_RANDOM,   -- distributed writes
    agent_id         INT NOT NULL,
    task_description TEXT NOT NULL,
    status           VARCHAR(20) DEFAULT 'running',     -- 'running', 'completed', 'failed', 'timeout'
    started_at       TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at     TIMESTAMP NULL,
    INDEX idx_agent_id (agent_id),
    INDEX idx_status (status),
    INDEX idx_started_at (started_at)
);

CREATE TABLE state_snapshots (
    snapshot_id  BIGINT PRIMARY KEY AUTO_INCREMENT,
    session_id   BIGINT NOT NULL,
    step_number  INT NOT NULL,
    state_data   JSON NOT NULL,               -- {"current_task": "...", "progress": 0.5, "variables": {...}}
    created_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_session_id (session_id),
    INDEX idx_session_step (session_id, step_number)
);

CREATE TABLE tool_calls (
    call_id      BIGINT PRIMARY KEY AUTO_RANDOM,      -- distributed writes
    session_id   BIGINT NOT NULL,
    tool_name    VARCHAR(100) NOT NULL,                -- 'web_search', 'code_exec', 'file_read', 'api_call', 'db_query'
    input        JSON,
    output       JSON,
    duration_ms  INT,
    called_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_session_id (session_id),
    INDEX idx_tool_name (tool_name),
    INDEX idx_called_at (called_at)
);

CREATE TABLE insights (
    insight_id       BIGINT PRIMARY KEY AUTO_INCREMENT,
    tenant_id        VARCHAR(50) NOT NULL,
    qwen_analysis    TEXT NOT NULL,
    recommendations  JSON,                     -- {"patterns": [...], "failures": [...], "optimizations": [...]}
    generated_at     TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3.3 TiFlash Replicas (Analytical Tables)

```sql
-- Enable columnar replicas for analytics (per tenant)
ALTER TABLE tenant_acme.sessions SET TIFLASH REPLICA 1;
ALTER TABLE tenant_acme.tool_calls SET TIFLASH REPLICA 1;
ALTER TABLE tenant_acme.state_snapshots SET TIFLASH REPLICA 1;

-- Wait for replicas to sync (check with):
SELECT TABLE_NAME, REPLICA_COUNT, AVAILABLE
FROM information_schema.tiflash_replica
WHERE TABLE_SCHEMA = 'tenant_acme';
-- AVAILABLE must = 1 before querying
```

### 3.4 SQL Views (3 per tenant)

```sql
-- VIEW 1: Agent Performance Summary
CREATE VIEW tenant_acme.agent_performance AS
SELECT
    a.name,
    a.type,
    COUNT(s.session_id)       AS total_sessions,
    AVG(TIMESTAMPDIFF(SECOND, s.started_at, s.completed_at)) AS avg_duration_sec,
    SUM(CASE WHEN s.status = 'completed' THEN 1 ELSE 0 END) AS successes,
    SUM(CASE WHEN s.status = 'failed'    THEN 1 ELSE 0 END) AS failures
FROM agents a
JOIN sessions s ON a.agent_id = s.agent_id
GROUP BY a.name, a.type;

-- VIEW 2: Tool Usage Statistics
CREATE VIEW tenant_acme.tool_usage_stats AS
SELECT
    tool_name,
    COUNT(*)            AS call_count,
    AVG(duration_ms)    AS avg_ms,
    MIN(duration_ms)    AS min_ms,
    MAX(duration_ms)    AS max_ms
FROM tool_calls
GROUP BY tool_name
ORDER BY call_count DESC;

-- VIEW 3: Daily Sessions Trend
CREATE VIEW tenant_acme.daily_sessions AS
SELECT
    DATE(started_at) AS day,
    COUNT(*)         AS sessions,
    SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END) AS completed,
    SUM(CASE WHEN status = 'failed'    THEN 1 ELSE 0 END) AS failed
FROM sessions
GROUP BY DATE(started_at)
ORDER BY day;
```

### 3.5 RBAC Configuration

```sql
-- Per-tenant user (note: TiDB Cloud Starter requires cluster prefix)
-- Replace {prefix} with actual cluster prefix

-- Tenant users (app backend)
CREATE USER '{prefix}.user_tenant_acme'@'%' IDENTIFIED BY '<strong_password>';
GRANT SELECT, INSERT, UPDATE ON tenant_acme.* TO '{prefix}.user_tenant_acme'@'%';

CREATE USER '{prefix}.user_tenant_globex'@'%' IDENTIFIED BY '<strong_password>';
GRANT SELECT, INSERT, UPDATE ON tenant_globex.* TO '{prefix}.user_tenant_globex'@'%';

-- ... repeat for each tenant

-- Quick BI read-only user
CREATE USER '{prefix}.quickbi_reader'@'%' IDENTIFIED BY '<strong_password>';
-- Grant SELECT only on views, NOT on raw tables
GRANT SELECT ON tenant_acme.agent_performance TO '{prefix}.quickbi_reader'@'%';
GRANT SELECT ON tenant_acme.tool_usage_stats  TO '{prefix}.quickbi_reader'@'%';
GRANT SELECT ON tenant_acme.daily_sessions    TO '{prefix}.quickbi_reader'@'%';
-- ... repeat for each tenant's views

-- Admin user for provisioning (reserved, never used by app or Quick BI)
-- Already exists from TiDB Cloud setup
```

### 3.6 Cross-Schema Plan Binding (Module 2)

```sql
-- Identify optimal plan for common analytical query
USE tenant_acme;
EXPLAIN SELECT tool_name, COUNT(*) AS call_count, AVG(duration_ms) AS avg_ms
FROM tool_calls GROUP BY tool_name ORDER BY call_count DESC;

-- Create global binding that applies across ALL tenant schemas
CREATE GLOBAL BINDING FOR
    SELECT tool_name, COUNT(*) AS call_count, AVG(duration_ms) AS avg_ms
    FROM tool_calls GROUP BY tool_name ORDER BY call_count DESC
USING
    SELECT /*+ HASH_AGG() */ tool_name, COUNT(*) AS call_count, AVG(duration_ms) AS avg_ms
    FROM tool_calls GROUP BY tool_name ORDER BY call_count DESC;

-- Verify binding exists
SHOW GLOBAL BINDINGS;

-- Verify same plan is used in different tenant
USE tenant_globex;
EXPLAIN SELECT tool_name, COUNT(*) AS call_count, AVG(duration_ms) AS avg_ms
FROM tool_calls GROUP BY tool_name ORDER BY call_count DESC;
-- Should show same plan as tenant_acme
```

### 3.7 Platform Schema (Cross-Tenant Metadata)

```sql
CREATE DATABASE platform;

CREATE TABLE platform.tenants (
    tenant_id    VARCHAR(50) PRIMARY KEY,   -- 'acme', 'globex', etc.
    db_name      VARCHAR(100) NOT NULL,     -- 'tenant_acme'
    db_user      VARCHAR(100) NOT NULL,     -- '{prefix}.user_tenant_acme'
    display_name VARCHAR(200),
    status       VARCHAR(20) DEFAULT 'active',
    created_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 4. Application Layer - FastAPI Backend

### 4.1 Project Structure

```
agentnexus/
├── app/
│   ├── __init__.py
│   ├── main.py                  # FastAPI app entry point
│   ├── config.py                # Environment variables, settings
│   ├── database.py              # Connection pool manager (per-tenant)
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── sessions.py          # POST /api/{tenant_id}/sessions
│   │   ├── insights.py          # GET  /api/{tenant_id}/insights
│   │   ├── provisioning.py      # POST /api/admin/provision/{tenant_id}
│   │   └── health.py            # GET  /health
│   ├── services/
│   │   ├── __init__.py
│   │   ├── tenant_service.py    # Tenant validation & routing
│   │   ├── session_service.py   # Session CRUD operations
│   │   ├── ai_service.py        # Qwen integration
│   │   └── provisioning_service.py  # Automated tenant provisioning
│   ├── models/
│   │   ├── __init__.py
│   │   └── schemas.py           # Pydantic models (request/response)
│   └── sql/
│       ├── create_tables.sql    # Table DDL template
│       ├── create_views.sql     # View DDL template
│       └── seed_data.py         # Seed data generator
├── scripts/
│   ├── provision_tenant.py      # Standalone provisioning script
│   ├── seed_all_tenants.py      # Populate all tenants with data
│   └── demo_online_ddl.py       # Module 4 demo helper
├── requirements.txt
├── .env.example
└── README.md
```

### 4.2 Environment Variables

```bash
# TiDB Cloud Connection (admin - only for provisioning)
TIDB_HOST=gateway01.prod.aws.tidbcloud.com  # or Alibaba Cloud endpoint
TIDB_PORT=4000
TIDB_ADMIN_USER={prefix}.root
TIDB_ADMIN_PASSWORD=<admin_password>

# Per-tenant credentials (loaded from platform.tenants or env)
TIDB_PASSWORD_ACME=<password>
TIDB_PASSWORD_GLOBEX=<password>
TIDB_PASSWORD_INITECH=<password>
TIDB_PASSWORD_UMBRELLA=<password>
TIDB_PASSWORD_WAYNE=<password>

# TiDB Cloud Cluster Prefix (required for Starter tier usernames)
TIDB_CLUSTER_PREFIX=4FxNxxxxx

# Qwen API
DASHSCOPE_API_KEY=sk-xxxxxxxxxxxxxxxx

# App
APP_PORT=8000
```

### 4.3 Core Components

#### Connection Pool Manager (`database.py`)

```python
import mysql.connector
from mysql.connector import pooling
from typing import Dict
import os
import ssl

class TenantConnectionManager:
    """Manages per-tenant database connection pools."""

    def __init__(self):
        self._pools: Dict[str, pooling.MySQLConnectionPool] = {}
        self.host = os.getenv("TIDB_HOST")
        self.port = int(os.getenv("TIDB_PORT", "4000"))
        self.prefix = os.getenv("TIDB_CLUSTER_PREFIX")
        self.ssl_ca = "/etc/ssl/certs/ca-certificates.crt"  # TLS for TiDB Cloud

    def get_pool(self, tenant_id: str) -> pooling.MySQLConnectionPool:
        if tenant_id not in self._pools:
            db_name = f"tenant_{tenant_id}"
            db_user = f"{self.prefix}.user_tenant_{tenant_id}"
            db_pass = os.getenv(f"TIDB_PASSWORD_{tenant_id.upper()}")

            self._pools[tenant_id] = pooling.MySQLConnectionPool(
                pool_name=f"pool_{tenant_id}",
                pool_size=5,
                host=self.host,
                port=self.port,
                user=db_user,
                password=db_pass,
                database=db_name,
                ssl_ca=self.ssl_ca,
                ssl_verify_cert=True,
            )
        return self._pools[tenant_id]

    def get_connection(self, tenant_id: str):
        return self.get_pool(tenant_id).get_connection()

    def get_admin_connection(self):
        """Admin connection for provisioning only."""
        return mysql.connector.connect(
            host=self.host,
            port=self.port,
            user=f"{self.prefix}.root",
            password=os.getenv("TIDB_ADMIN_PASSWORD"),
            ssl_ca=self.ssl_ca,
            ssl_verify_cert=True,
        )

# Singleton
tenant_db = TenantConnectionManager()
```

#### Tenant Router Middleware

```python
from fastapi import Request, HTTPException

VALID_TENANTS = {"acme", "globex", "initech", "umbrella", "wayne"}

async def validate_tenant(tenant_id: str):
    """Validate tenant exists. Returns 404 for unknown tenants."""
    if tenant_id not in VALID_TENANTS:
        raise HTTPException(status_code=404, detail=f"Tenant '{tenant_id}' not found")
    return tenant_id
```

### 4.4 API Endpoints

#### POST `/api/{tenant_id}/sessions` (Write API)

Creates a new agent session and simulates execution.

```python
# Request Body
{
    "agent_id": 3,
    "task_description": "Research latest trends in distributed databases"
}

# Response (201 Created)
{
    "session_id": 8392047561234,
    "agent_id": 3,
    "task_description": "Research latest trends in distributed databases",
    "status": "completed",
    "started_at": "2026-03-12T10:30:00",
    "completed_at": "2026-03-12T10:30:45",
    "tool_calls_count": 5,
    "state_snapshots_count": 3
}
```

**Implementation logic:**
1. Validate tenant_id exists
2. Connect using the tenant's DB user (isolation enforced)
3. INSERT into `sessions` with status='running'
4. Generate simulated tool_calls and state_snapshots
5. UPDATE session status to 'completed'
6. Return session summary

#### GET `/api/{tenant_id}/insights` (AI Endpoint)

Generates Qwen-powered insights from tenant's agent execution logs.

```python
# Response (200 OK)
{
    "insight_id": 42,
    "tenant_id": "acme",
    "analysis": {
        "patterns": [
            {
                "pattern": "Sequential tool dependency",
                "description": "web_search is always followed by code_exec in researcher agents",
                "frequency": "78% of researcher sessions"
            }
        ],
        "failures": [
            {
                "failure": "Timeout on API calls",
                "affected_agents": ["web_navigator"],
                "suggested_fix": "Implement retry with exponential backoff"
            }
        ],
        "optimizations": [
            {
                "optimization": "Parallel tool execution",
                "impact": "Could reduce avg session duration by 35%",
                "implementation": "Use async tool calls for independent steps"
            }
        ],
        "summary": "Acme's agent fleet shows strong task completion (87%) but web_navigator agents suffer from timeout failures. Parallelizing independent tool calls would significantly improve throughput."
    },
    "generated_at": "2026-03-12T10:35:00"
}
```

**Implementation logic:**
1. Query TiDB for last 50 tool calls and 20 completed sessions
2. Format as clean JSON context payload
3. Send to Qwen-Plus with system prompt
4. Parse Qwen's JSON response
5. Store in `insights` table
6. Return insight

### 4.5 Qwen Integration Pattern (`ai_service.py`)

```python
from openai import OpenAI
import json
import os

client = OpenAI(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url="https://dashscope-intl.aliyuncs.com/compatible-mode/v1"
)

SYSTEM_PROMPT = """You are an AI operations analyst. You will receive an enterprise \
tenant's agent execution logs as JSON. Analyze tool usage patterns, common failure \
modes, and session efficiency. Respond ONLY with a JSON object:
{
  "patterns": [{"pattern": "...", "description": "...", "frequency": "..."}],
  "failures": [{"failure": "...", "affected_agents": [...], "suggested_fix": "..."}],
  "optimizations": [{"optimization": "...", "impact": "...", "implementation": "..."}],
  "summary": "brief overall insight"
}
No other text."""

def generate_insights(tenant_context: dict) -> dict:
    """Send structured context to Qwen and parse response."""
    response = client.chat.completions.create(
        model="qwen-plus",
        messages=[
            {"role": "system", "content": SYSTEM_PROMPT},
            {"role": "user", "content": json.dumps(tenant_context, default=str)}
        ],
        temperature=0.7,
        max_tokens=2000
    )

    raw = response.choices[0].message.content
    # Strip potential markdown code fences
    if raw.startswith("```"):
        raw = raw.split("\n", 1)[1].rsplit("```", 1)[0]

    return json.loads(raw)
```

### 4.6 Provisioning Script (`scripts/provision_tenant.py`)

This is the critical script for Module 1 (30 pts, MANDATORY). Must add a 6th tenant in < 60 seconds.

```python
"""
Automated tenant provisioning script.
Usage: python provision_tenant.py <tenant_id> <display_name>
Example: python provision_tenant.py stark "Stark Industries"

Creates: database, 5 tables, DB user, grants, TiFlash replicas, 3 SQL views
Target: < 60 seconds end-to-end
"""

import sys
import time
import mysql.connector
import os

def provision_tenant(tenant_id: str, display_name: str):
    start = time.time()
    prefix = os.getenv("TIDB_CLUSTER_PREFIX")
    db_name = f"tenant_{tenant_id}"
    db_user = f"{prefix}.user_tenant_{tenant_id}"
    db_pass = generate_secure_password()  # or from env

    conn = get_admin_connection()
    cursor = conn.cursor()

    # Step 1: Create database
    cursor.execute(f"CREATE DATABASE IF NOT EXISTS `{db_name}`")

    # Step 2: Create tables (5 tables)
    cursor.execute(f"USE `{db_name}`")
    cursor.execute(AGENTS_DDL)
    cursor.execute(SESSIONS_DDL)
    cursor.execute(STATE_SNAPSHOTS_DDL)
    cursor.execute(TOOL_CALLS_DDL)
    cursor.execute(INSIGHTS_DDL)

    # Step 3: Create user with scoped permissions
    cursor.execute(f"CREATE USER IF NOT EXISTS '{db_user}'@'%' IDENTIFIED BY '{db_pass}'")
    cursor.execute(f"GRANT SELECT, INSERT, UPDATE ON `{db_name}`.* TO '{db_user}'@'%'")

    # Step 4: Enable TiFlash replicas on analytical tables
    cursor.execute(f"ALTER TABLE `{db_name}`.sessions SET TIFLASH REPLICA 1")
    cursor.execute(f"ALTER TABLE `{db_name}`.tool_calls SET TIFLASH REPLICA 1")
    cursor.execute(f"ALTER TABLE `{db_name}`.state_snapshots SET TIFLASH REPLICA 1")

    # Step 5: Create SQL views (3 views)
    create_views(cursor, db_name)

    # Step 6: Grant Quick BI reader access on views
    cursor.execute(f"GRANT SELECT ON `{db_name}`.agent_performance TO '{prefix}.quickbi_reader'@'%'")
    cursor.execute(f"GRANT SELECT ON `{db_name}`.tool_usage_stats TO '{prefix}.quickbi_reader'@'%'")
    cursor.execute(f"GRANT SELECT ON `{db_name}`.daily_sessions TO '{prefix}.quickbi_reader'@'%'")

    # Step 7: Register in platform metadata
    cursor.execute("""
        INSERT INTO platform.tenants (tenant_id, db_name, db_user, display_name)
        VALUES (%s, %s, %s, %s)
    """, (tenant_id, db_name, db_user, display_name))

    conn.commit()
    elapsed = time.time() - start
    print(f"✓ Tenant '{tenant_id}' provisioned in {elapsed:.1f}s")
    print(f"  Database: {db_name}")
    print(f"  User:     {db_user}")
    print(f"  Views:    agent_performance, tool_usage_stats, daily_sessions")
    print(f"  TiFlash:  sessions, tool_calls, state_snapshots")

    cursor.close()
    conn.close()
    return elapsed
```

---

## 5. Seed Data Strategy

### 5.1 Requirements per Tenant

| Table | Min Records | Strategy |
|-------|-------------|----------|
| agents | 10+ | 12 agents across 5 types |
| sessions | 100+ | 120 sessions, distributed across agents and statuses |
| state_snapshots | 500+ | 3-8 snapshots per session |
| tool_calls | 1000+ | 5-15 tool calls per session |

### 5.2 Agent Types and Definitions

```python
AGENT_TYPES = [
    # (name, type, capabilities)
    ("Atlas Researcher",    "researcher",      {"tools": ["web_search", "doc_reader"], "languages": []}),
    ("Nova Researcher",     "researcher",      {"tools": ["web_search", "api_call"], "languages": []}),
    ("CodeForge",           "coder",           {"tools": ["code_exec", "file_read", "file_write"], "languages": ["python", "javascript"]}),
    ("ByteSmith",           "coder",           {"tools": ["code_exec", "file_read"], "languages": ["python", "go", "rust"]}),
    ("DataLens",            "analyst",         {"tools": ["db_query", "code_exec"], "languages": ["python", "sql"]}),
    ("InsightEngine",       "analyst",         {"tools": ["db_query", "api_call", "code_exec"], "languages": ["python"]}),
    ("WebCrawler",          "web_navigator",   {"tools": ["web_search", "web_scrape", "screenshot"], "languages": []}),
    ("PageRunner",          "web_navigator",   {"tools": ["web_search", "form_fill", "screenshot"], "languages": []}),
    ("TaskMaster",          "orchestrator",    {"tools": ["task_dispatch", "agent_spawn"], "languages": []}),
    ("FlowDirector",        "orchestrator",    {"tools": ["task_dispatch", "agent_spawn", "monitoring"], "languages": []}),
    ("SecScan",             "security_auditor",{"tools": ["code_exec", "file_read", "api_call"], "languages": ["python"]}),
    ("DocWriter",           "content_creator", {"tools": ["web_search", "file_write", "code_exec"], "languages": ["markdown"]}),
]
```

### 5.3 Tool Names Pool

```python
TOOL_NAMES = [
    "web_search", "code_exec", "file_read", "file_write",
    "api_call", "db_query", "web_scrape", "screenshot",
    "doc_reader", "form_fill", "task_dispatch", "agent_spawn",
    "monitoring", "email_send", "slack_notify"
]
```

### 5.4 Realistic Session Statuses

```python
# Distribution: 70% completed, 15% failed, 10% running, 5% timeout
STATUS_WEIGHTS = {
    "completed": 0.70,
    "failed":    0.15,
    "running":   0.10,
    "timeout":   0.05,
}
```

---

## 6. Module Demos

### 6.1 Module 1: Rapid Provisioning (30 pts, MANDATORY)

**Demo steps:**
1. Show 5 existing tenant databases: `SHOW DATABASES;`
2. Run provisioning script live: `python provision_tenant.py stark "Stark Industries"`
3. Show new database created: `SHOW DATABASES;`
4. Show tables: `USE tenant_stark; SHOW TABLES;`
5. Show user created: login as `user_tenant_stark`, `SHOW DATABASES;` (only sees `tenant_stark`)
6. Attempt cross-tenant access: `SELECT * FROM tenant_acme.agents;` → ACCESS DENIED
7. Timer visible: must be < 60 seconds

### 6.2 Module 2: Cross-Schema Plan Binding (20 pts)

**Demo steps:**
1. Show GLOBAL BINDING: `SHOW GLOBAL BINDINGS;`
2. Run EXPLAIN in tenant_acme: show plan with hint
3. Run same EXPLAIN in tenant_globex: show identical plan
4. Explain why this matters: consistent performance across 500 tenants

### 6.3 Module 3: TiFlash HTAP (30 pts, MANDATORY)

**Demo steps:**
1. Show TiFlash replicas enabled:
   ```sql
   SELECT TABLE_SCHEMA, TABLE_NAME, REPLICA_COUNT, AVAILABLE
   FROM information_schema.tiflash_replica;
   ```
2. Run EXPLAIN on analytical view query:
   ```sql
   USE tenant_acme;
   EXPLAIN SELECT * FROM tool_usage_stats;
   ```
3. Look for `cop[tiflash]` in output
4. Explain: "Analytical queries route to TiFlash columnar engine. Transactional writes continue on TiKV row store. Zero interference."

### 6.4 Module 4: Online Schema Change (20 pts)

**Demo steps:**
1. Start a write stream (continuous INSERT into tenant_acme.tool_calls)
2. In parallel, run DDL on same tenant:
   ```sql
   ALTER TABLE tenant_acme.sessions ADD COLUMN priority VARCHAR(10) DEFAULT 'normal';
   ```
3. Show write stream continues uninterrupted (no errors)
4. Show column exists: `DESCRIBE tenant_acme.sessions;`
5. Show existing rows have default value, new rows can use column

---

## 7. Network Architecture

### 7.1 VPC Configuration

```
VPC: 10.0.0.0/16  (region: ap-southeast-1, Singapore)
  └─ vSwitch: 10.0.1.0/24  (AZ: ap-southeast-1a)
       └─ ECS Instance
            ├─ Private IP: 10.0.1.x
            └─ Elastic IP: x.x.x.x (public)
```

### 7.2 Security Group Rules

| Direction | Protocol | Port | Source/Dest | Purpose |
|-----------|----------|------|-------------|---------|
| Inbound | TCP | 22 | Team IPs only | SSH access |
| Inbound | TCP | 80 | 0.0.0.0/0 | HTTP API |
| Inbound | TCP | 443 | 0.0.0.0/0 | HTTPS API |
| Outbound | TCP | 4000 | TiDB Cloud IP | TiDB connection |
| Outbound | TCP | 443 | 0.0.0.0/0 | Qwen API (HTTPS) |

### 7.3 Connection Paths

```
ECS ──TCP 4000 (TLS)──► TiDB Cloud Starter (public endpoint)
                         Allowlist: ECS Elastic IP

ECS ──HTTPS 443────────► dashscope-intl.aliyuncs.com (Qwen API)
                         Auth: DASHSCOPE_API_KEY header

Quick BI ──TCP 4000 (TLS)──► TiDB Cloud Starter (public endpoint)
                              Allowlist: Quick BI egress IPs
                              User: {prefix}.quickbi_reader (read-only on views)
```

---

## 8. Quick BI Dashboard Design

### 8.1 Three Charts Required

| # | Chart | Type | SQL View Source | Description |
|---|-------|------|-----------------|-------------|
| 1 | Agent Performance by Type | Grouped Bar Chart | `agent_performance` | X: agent type, Y: total_sessions + successes + failures |
| 2 | Tool Usage Ranking | Horizontal Bar Chart | `tool_usage_stats` | X: call_count, Y: tool_name, sorted desc |
| 3 | Daily Sessions Trend | Line Chart | `daily_sessions` | X: day, Y: sessions + completed lines |

### 8.2 Setup Checklist

1. Add Quick BI egress IPs to TiDB Cloud IP access list
2. Create `quickbi_reader` user (SELECT on views only)
3. Add TiDB as data source in Quick BI (host, port 4000, quickbi_reader credentials)
4. Create datasets: one per view per tenant
5. Build dashboard with 3 charts
6. Test tenant isolation: Tenant A dashboard shows ONLY Tenant A data

---

## 9. Team Work Division (3 Developers, 6 Hours)

### Dev 1: Infrastructure + Database (TiDB Modules)

| Hour | Task |
|------|------|
| 0-1 | Set up VPC, vSwitch, Security Group, ECS instance. Install Python 3.11, dependencies |
| 1-2 | Connect to TiDB Cloud. Write and test provisioning script (Module 1). Create 5 tenant schemas |
| 2-3 | Enable TiFlash replicas (Module 3). Create SQL views. Set up RBAC users |
| 3-4 | Create cross-schema plan binding (Module 2). Write seed data script. Run seed data |
| 4-5 | Prepare Online DDL demo (Module 4). Quick BI data source connection |
| 5-6 | Quick BI dashboards (3 charts). Demo rehearsal |

### Dev 2: Application Backend (FastAPI)

| Hour | Task |
|------|------|
| 0-1 | Scaffold FastAPI project. Set up config, database connection pool manager |
| 1-2 | Implement POST /api/{tenant_id}/sessions endpoint with tenant routing |
| 2-3 | Implement session creation logic with simulated tool_calls and state_snapshots |
| 3-4 | Implement GET /api/{tenant_id}/insights with Qwen integration |
| 4-5 | Error handling (invalid tenant → 4xx). Testing all endpoints |
| 5-6 | Deploy to ECS. Environment variables. Final testing. Demo rehearsal |

### Dev 3: AI Integration + Demo Preparation

| Hour | Task |
|------|------|
| 0-1 | Activate Model Studio, get API key. Test Qwen-Plus connectivity from ECS |
| 1-2 | Craft and iterate system prompt for agent performance analysis |
| 2-3 | Build AI service: query TiDB context → format JSON → call Qwen → parse response → store |
| 3-4 | Write seed data generator script with realistic patterns |
| 4-5 | Prepare architecture slide. Write demo script commands |
| 5-6 | Full demo rehearsal. Edge case testing. Backup commands ready |

---

## 10. Demo Script (5 Minutes)

### 0:00-0:30 — Architecture Presentation
- Show architecture diagram (1 slide)
- "AgentNexus: Multi-tenant AI agent platform. 5 tenants, TiDB for state + analytics, Qwen for meta-insights."
- Mention 4 TiDB modules

### 0:30-1:15 — Isolation + RBAC [M1 + M3]
```bash
# Login as tenant user
mysql -h <tidb-host> -P 4000 -u {prefix}.user_tenant_acme -p --ssl-mode=VERIFY_IDENTITY

# Show only own database visible
SHOW DATABASES;
# Output: tenant_acme

# Attempt cross-tenant access
SELECT * FROM tenant_globex.agents;
# Output: ERROR 1044 (42000): Access denied
```

### 1:15-2:00 — Live Write + AI
```bash
# POST a new session
curl -X POST http://<ecs-ip>/api/acme/sessions \
  -H "Content-Type: application/json" \
  -d '{"agent_id": 1, "task_description": "Research distributed database trends 2026"}'

# GET AI insights
curl http://<ecs-ip>/api/acme/insights | python -m json.tool

# Point out: Qwen analyzed tool usage patterns, found failures, suggested optimizations
```

### 2:00-2:45 — Provisioning [M1]
```bash
# Run provisioning script LIVE
time python scripts/provision_tenant.py stark "Stark Industries"
# Should complete in < 60 seconds

# Verify
mysql> SHOW DATABASES;  -- tenant_stark appears
mysql> USE tenant_stark; SHOW TABLES;  -- 5 tables
mysql> SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA='tenant_stark';
```

### 2:45-3:15 — Online DDL [M4]
```bash
# Terminal 1: Start continuous writes
python scripts/demo_online_ddl.py --write-stream tenant_acme

# Terminal 2: Run ALTER TABLE
mysql> ALTER TABLE tenant_acme.sessions ADD COLUMN priority VARCHAR(10) DEFAULT 'normal';

# Show Terminal 1: writes continue (no errors)
# Show column exists
mysql> DESCRIBE tenant_acme.sessions;
```

### 3:15-4:00 — Quick BI + Plan Binding [M2]
```bash
# Open Quick BI dashboard in browser
# Show 3 charts for tenant_acme
# Refresh after live POST → charts update
# Switch to tenant_globex dashboard → different data (isolation verified)

# Show global binding
mysql> SHOW GLOBAL BINDINGS;
mysql> USE tenant_acme;   EXPLAIN SELECT ...; -- Show plan with binding hint
mysql> USE tenant_globex;  EXPLAIN SELECT ...; -- Same plan
```

### 4:00-5:00 — Q&A Preparation

| Question | Answer |
|----------|--------|
| "How would you add a 50th tenant?" | Run the same provisioning script. TiDB handles thousands of schemas on one cluster. Linear process, no architectural changes |
| "How does plan binding help at 500 tenants?" | Without it, optimizer might pick different plans per schema based on data distribution. Binding ensures identical, optimized plans everywhere → predictable SLAs |
| "Why qwen-plus over qwen-max?" | Cost-effective for hackathon budget, still has solid reasoning. qwen-max would be used in production for highest quality |
| "What if a tenant overloads the system?" | TiDB's HTAP separation ensures analytics (TiFlash) don't impact transactions (TiKV). Further isolation via Resource Groups (stretch goal) |

---

## 11. Dependencies

### requirements.txt

```
fastapi==0.115.0
uvicorn[standard]==0.30.0
mysql-connector-python==9.1.0
openai==1.50.0
pydantic==2.9.0
python-dotenv==1.0.0
```

### System packages (ECS)

```bash
sudo apt update && sudo apt install -y python3.11 python3.11-venv python3-pip
python3.11 -m venv /opt/agentnexus/venv
source /opt/agentnexus/venv/bin/activate
pip install -r requirements.txt
```

---

## 12. Scoring Checklist

### Mandatory (must achieve to exceed 60 pts)

| Module | Points | Deliverable | Status |
|--------|--------|-------------|--------|
| M1: Rapid Provisioning | 30 | 5 tenants + script adds 6th in <60s + RBAC verified | ☐ |
| M3: TiFlash HTAP | 30 | Replicas enabled + EXPLAIN shows cop[tiflash] | ☐ |

### Optional (extra points)

| Module | Points | Deliverable | Status |
|--------|--------|-------------|--------|
| M2: Plan Binding | 20 | GLOBAL BINDING + same plan across schemas | ☐ |
| M4: Online DDL | 20 | ALTER while writing + zero downtime | ☐ |
| Qwen AI Quality | 15 | Coherent insights + stored in TiDB + JSON parsed | ☐ |
| Quick BI Dashboard | 10 | 3 charts + live data + tenant isolation | ☐ |
| Write API | 10 | POST works + tenant routing + 4xx on invalid | ☐ |
| Code Quality | 10 | Clean routing, no hardcoded creds, pool usage | ☐ |

**Maximum possible: 145 points** (100 base + 45 optional)

---

## 13. Critical Pitfalls to Avoid

| Pitfall | Prevention |
|---------|-----------|
| Quick BI IP not allowlisted | Pre-add IPs before event |
| Qwen returns free text instead of JSON | System prompt: "Respond ONLY with valid JSON. No other text." |
| Quick BI on raw tables (no isolation) | Always use SQL views, never raw tables |
| Admin user for Quick BI | Dedicated quickbi_reader user |
| TiFlash not ready | Wait for AVAILABLE=1 before querying (2-5 min) |
| Wrong username format | Always use `{prefix}.username` format on Starter |
| Hardcoded credentials | All secrets in environment variables |
| No connection pooling | Use mysql.connector.pooling per tenant |
| Sequential tenant provisioning | Script handles one tenant at a time (fast enough) |
