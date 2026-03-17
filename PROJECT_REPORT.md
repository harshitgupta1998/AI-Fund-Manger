# 📘 AgenticAI Fund Manager — Comprehensive Project Report

> **Purpose**: A learning guide to help you understand the full architecture, code structure, data flows, AWS services, and design decisions in this project.
>
> **Generated**: March 2026 · Based on full source-code analysis of every file in the repository.

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [High-Level Architecture](#2-high-level-architecture)
3. [Technology Stack](#3-technology-stack)
4. [Project Structure (Annotated)](#4-project-structure-annotated)
5. [Agent Deep Dives](#5-agent-deep-dives)
   - 5.1 [Financial Analyst (Stage 1)](#51-financial-analyst-stage-1)
   - 5.2 [Portfolio Architect (Stage 2)](#52-portfolio-architect-stage-2)
   - 5.3 [Risk Manager (Stage 3)](#53-risk-manager-stage-3)
   - 5.4 [Fund Manager — Orchestrator (Stage 4)](#54-fund-manager--orchestrator-stage-4)
6. [Shared Utilities](#6-shared-utilities)
7. [AWS Services Map](#7-aws-services-map)
8. [Data Flow — End-to-End](#8-data-flow--end-to-end)
9. [Deployment Pipeline](#9-deployment-pipeline)
10. [Authentication & Security](#10-authentication--security)
11. [Streaming & Real-Time UI](#11-streaming--real-time-ui)
12. [Key Design Patterns](#12-key-design-patterns)
13. [Configuration Management](#13-configuration-management)
14. [How to Run Locally](#14-how-to-run-locally)
15. [Glossary](#15-glossary)

---

## 1. Executive Summary

**AgenticAI Fund Manager** is a multi-agent AI system for end-to-end investment fund management. It combines **4 specialized AI agents** that run sequentially in a pipeline:

| Stage | Agent | Responsibility | LLM Model |
|-------|-------|---------------|-----------|
| 1 | Financial Analyst | Assess investor risk profile & calculate required returns | GPT-OSS 120B |
| 2 | Portfolio Architect | Design optimal 3-ETF portfolio using real-time market data | GPT-OSS 120B |
| 3 | Risk Manager | Analyze macro/geopolitical risks and propose scenario adjustments | GPT-OSS 120B |
| 4 | Fund Manager | Orchestrate all agents, manage memory & produce final report | LangGraph + AgentCore Memory |

Each agent is deployed as a containerized microservice on **AWS Bedrock AgentCore Runtime** and exposes a streaming HTTP endpoint. The Fund Manager orchestrates them via **LangGraph** (a state-machine graph library) and persists consultation history using **AgentCore Memory** with automatic summarization.

The user interacts through a **Streamlit** web application that renders real-time streaming output, charts (Plotly), and historical consultation summaries.

---

## 2. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     STREAMLIT WEB UI                            │
│           fund_manager/app.py  (port 8501)                      │
└───────────────────────────┬─────────────────────────────────────┘
                            │ invoke_agent_runtime (boto3)
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│              FUND MANAGER  (LangGraph Orchestrator)             │
│         Deployed on AWS Bedrock AgentCore Runtime               │
│                                                                 │
│   ┌──────────┐   ┌──────────────┐   ┌──────────────┐          │
│   │financial │──▶│  portfolio   │──▶│    risk      │          │
│   │  _node   │   │   _node      │   │   _node      │          │
│   └──────────┘   └──────────────┘   └──────────────┘          │
│         │                │                  │                   │
│         │ save_to_memory │ save_to_memory   │ save_to_memory   │
│         ▼                ▼                  ▼                   │
│   ┌──────────────────────────────────────────────────┐         │
│   │        AgentCore Memory (SUMMARY strategy)       │         │
│   └──────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────┘
        │                      │                    │
        ▼                      ▼                    ▼
 ┌──────────────┐   ┌───────────────────┐   ┌──────────────────┐
 │  Financial   │   │   Portfolio       │   │  Risk Manager    │
 │  Analyst     │   │   Architect       │   │                  │
 │  Runtime     │   │   Runtime         │   │  Runtime         │
 │              │   │                   │   │                  │
 │  Tool:       │   │  Tool:            │   │  Tool:           │
 │  calculator  │   │  MCP Server       │   │  MCP Gateway     │
 │  (strands)   │   │  (yfinance)       │   │  → Lambda        │
 └──────────────┘   │                   │   │  → yfinance      │
                    │  ┌──────────────┐ │   │                  │
                    │  │ MCP Server   │ │   │  ┌────────────┐  │
                    │  │ (FastMCP)    │ │   │  │ Lambda Fn  │  │
                    │  │ ETF perf +   │ │   │  │ news,econ, │  │
                    │  │ correlation  │ │   │  │ geopolitics│  │
                    │  └──────────────┘ │   │  └────────────┘  │
                    └───────────────────┘   └──────────────────┘
```

### Key architectural decisions:
- **Microservice agents**: Each agent is an independent container deployed on AgentCore Runtime. This allows independent scaling, versioning, and testing.
- **Sequential pipeline**: Financial → Portfolio → Risk. Each stage's output becomes the next stage's input.
- **Streaming everywhere**: All agents yield events as they process (text chunks, tool calls, tool results, final JSON). The UI renders incrementally.
- **Memory with auto-summarization**: AgentCore Memory's SUMMARY strategy automatically condenses each session into a retrievable summary.

---

## 3. Technology Stack

### Languages & Frameworks
| Component | Technology |
|-----------|-----------|
| Agent Framework | [Strands Agents](https://github.com/strands-agents/strands-agents) — Python agent framework with tool support |
| Orchestration | [LangGraph](https://github.com/langchain-ai/langgraph) — State-machine graph for multi-agent workflows |
| Web UI | [Streamlit](https://streamlit.io/) — Rapid Python web app framework |
| Charts | [Plotly](https://plotly.com/python/) — Interactive visualizations |
| Data Protocol | [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) — Standard protocol for tool/data access |
| Financial Data | [yfinance](https://github.com/ranaroussi/yfinance) — Yahoo Finance API wrapper |

### AWS Services
| Service | Purpose |
|---------|---------|
| **Bedrock AgentCore Runtime** | Hosts and runs each AI agent as a managed container |
| **Bedrock (Models)** | Provides LLM inference (GPT-OSS 120B via Bedrock) |
| **AgentCore Memory** | Long-term memory with SUMMARY strategy for session history |
| **AgentCore MCP Server** | Hosts the MCP server for Portfolio Architect's ETF data tools |
| **AgentCore MCP Gateway** | Routes MCP tool calls from Risk Manager to Lambda functions |
| **Lambda** | Serverless functions for news/market/geopolitical data retrieval |
| **Lambda Layer** | Packages yfinance + dependencies for Lambda |
| **Cognito** | OAuth2 M2M authentication for MCP Server & Gateway |
| **IAM** | Roles and policies for each agent/service |
| **ECR** | Docker image registry for agent containers |
| **CloudWatch** | Logging for agent runtimes |

### Python Dependencies (root `requirements.txt`)
```
strands-agents          # Agent framework
strands-agents-tools    # Built-in tools (calculator, etc.)
bedrock-agentcore-starter-toolkit  # Deployment toolkit
langgraph               # Multi-agent orchestration
mcp                     # Model Context Protocol client
yfinance                # Financial data
pandas / numpy          # Data processing
streamlit / plotly      # Web UI
boto3                   # AWS SDK
requests                # HTTP client
```

---

## 4. Project Structure (Annotated)

```
AI-Fund-Manger/
├── config.py                    # 🔧 Global configuration (region, agent names, etc.)
├── deploy_all.py                # 🚀 One-click deployment of all 4 stages
├── cleanup_all.py               # 🧹 One-click cleanup of all AWS resources
├── requirements.txt             # 📦 Root Python dependencies
├── Dockerfile                   # 🐳 Docker config for Fund Manager web app
├── README.md                    # 📖 Project documentation
│
├── financial_analyst/           # ── STAGE 1 ──
│   ├── financial_analyst.py     #    Agent logic: risk profiling with Calculator tool
│   ├── app.py                   #    Standalone Streamlit UI
│   ├── deploy.py                #    Deployment to AgentCore Runtime
│   ├── cleanup.py               #    Resource teardown
│   ├── deployment_info.json     #    Saved ARN/metadata after deployment
│   └── requirements.txt         #    Agent-specific dependencies
│
├── portfolio_architect/         # ── STAGE 2 ──
│   ├── portfolio_architect.py   #    Agent logic: ETF selection + Monte Carlo
│   ├── app.py                   #    Standalone Streamlit UI
│   ├── deploy.py                #    Deployment to AgentCore Runtime
│   ├── cleanup.py               #    Resource teardown
│   ├── deployment_info.json     #    Saved ARN/metadata after deployment
│   ├── requirements.txt         #    Agent-specific dependencies
│   └── mcp_server/              #    ── MCP Data Server ──
│       ├── server.py            #    FastMCP server: analyze_etf_performance + calculate_correlation
│       ├── deploy_mcp.py        #    Deploys MCP server to AgentCore
│       ├── requirements.txt     #    MCP server dependencies
│       └── mcp_deployment_info.json
│
├── risk_manager/                # ── STAGE 3 ──
│   ├── risk_manager.py          #    Agent logic: risk scenarios via MCP Gateway
│   ├── app.py                   #    Standalone Streamlit UI
│   ├── deploy.py                #    Deployment to AgentCore Runtime
│   ├── cleanup.py               #    Resource teardown
│   ├── deployment_info.json     #    Saved ARN/metadata after deployment
│   ├── requirements.txt         #    Agent-specific dependencies
│   ├── lambda_layer/            #    ── Lambda Layer ──
│   │   ├── deploy_lambda_layer.py   # Packages yfinance into Lambda layer
│   │   └── layer-yfinance.zip       # Pre-built layer ZIP
│   ├── lambda/                  #    ── Lambda Function ──
│   │   ├── lambda_function.py   #    3 tools: get_product_news, get_market_data, get_geopolitical_indicators
│   │   └── deploy_lambda.py     #    Deploys Lambda function
│   └── gateway/                 #    ── MCP Gateway ──
│       ├── target_config.py     #    Tool schemas for the Gateway Target
│       └── deploy_gateway.py    #    Creates Gateway + Target pointing to Lambda
│
├── fund_manager/                # ── STAGE 4 (Orchestrator) ──
│   ├── fund_manager.py          #    LangGraph workflow: financial→portfolio→risk
│   ├── app.py                   #    Full Streamlit UI (main entry point for users)
│   ├── deploy.py                #    Deployment to AgentCore Runtime
│   ├── cleanup.py               #    Resource teardown
│   ├── deployment_info.json     #    Saved ARN/metadata after deployment
│   ├── requirements.txt         #    Agent-specific dependencies
│   └── agentcore_memory/        #    ── Memory System ──
│       ├── deploy_agentcore_memory.py  # Creates AgentCore Memory with SUMMARY strategy
│       └── deployment_info.json
│
├── shared/                      # ── Shared Utilities ──
│   ├── runtime_utils.py         #    IAM role creation for AgentCore Runtime
│   ├── cognito_utils.py         #    Cognito User Pool, Resource Server, M2M client helpers
│   └── gateway_utils.py         #    Gateway IAM role, Gateway/Target creation helpers
│
└── static/                      # ── UI Assets ──
    └── *.png                    #    Architecture diagrams for each agent
```

---

## 5. Agent Deep Dives

### 5.1 Financial Analyst (Stage 1)

**File**: `financial_analyst/financial_analyst.py`

**Purpose**: Takes investor profile data (age, investment amount, target amount, experience, preferred sectors) and produces a risk profile assessment.

**How it works**:
1. Creates a **Strands Agent** with the `calculator` tool and GPT-OSS 120B model.
2. The system prompt instructs the LLM to:
   - Use the calculator tool to compute: `((target_amount / investment_amount) - 1) * 100`
   - Assess risk profile considering age, experience, and purpose
   - Output structured JSON with `risk_profile`, `required_annual_return_rate`, `key_sectors`, and `summary`
3. Uses `stream_async()` for real-time streaming of text chunks, tool calls, and results.

**Input** (example):
```json
{
  "total_investable_amount": 50000000,
  "target_amount": 70000000,
  "age": 37,
  "stock_investment_experience_years": 7,
  "investment_purpose": "Short-term Profit",
  "preferred_sectors": ["Growth Stocks (Tech/Bio)"]
}
```

**Output** (example):
```json
{
  "risk_profile": "Aggressive",
  "risk_profile_reason": "37 years old with 7 years experience...",
  "required_annual_return_rate": 40.0,
  "key_sectors": ["Technology", "Healthcare", "AI/Semiconductors"],
  "summary": "High growth strategy needed to achieve 40% annual return..."
}
```

**Key patterns**:
- `@app.entrypoint` decorator registers the function as the AgentCore Runtime entry point
- Global `analyst` instance is lazily initialized (singleton pattern)
- `extract_json_from_text()` helper strips non-JSON text from LLM responses
- The `BedrockAgentCoreApp` handles container lifecycle

---

### 5.2 Portfolio Architect (Stage 2)

**File**: `portfolio_architect/portfolio_architect.py`

**Purpose**: Takes the Financial Analyst's output (risk profile + sectors) and designs an optimal 3-ETF portfolio using real-time market data.

**How it works**:
1. Connects to an **MCP Server** (deployed separately) that provides 2 tools:
   - `analyze_etf_performance(ticker)` — Monte Carlo simulation (1000 runs) for a single ETF
   - `calculate_correlation(tickers)` — Correlation matrix between ETFs
2. The system prompt instructs the LLM to:
   - Select 5 candidate ETFs based on risk profile and sectors
   - Analyze each with Monte Carlo simulation
   - Calculate correlations between all 5
   - Select optimal 3 ETFs with best risk/return balance
   - Determine allocation weights (must sum to 100%)
   - Score the portfolio on profitability, risk management, and diversification (1-10)

**MCP Server** (`portfolio_architect/mcp_server/server.py`):
- Built with **FastMCP** framework
- `analyze_etf_performance()`: Fetches 2 years of daily prices via yfinance, calculates historical return/volatility, runs 1000 Monte Carlo simulations, returns expected return, loss probability, and return distribution
- `calculate_correlation()`: Computes pairwise correlation matrix from 2 years of daily returns

**Authentication flow**:
1. Reads MCP server info from environment variables (ARN, Cognito client ID/secret, user pool ID)
2. Acquires OAuth2 access token from Cognito token endpoint (client_credentials grant)
3. Passes Bearer token in MCP client HTTP headers

**Output** (example):
```json
{
  "portfolio_allocation": {"QQQ": 50, "XLV": 30, "SOXX": 20},
  "reason": "QQQ provides broad tech exposure...",
  "portfolio_scores": {
    "profitability": {"score": 8, "reason": "High growth potential..."},
    "risk_management": {"score": 6, "reason": "Tech-heavy concentration..."},
    "diversification": {"score": 7, "reason": "Healthcare provides offset..."}
  }
}
```

---

### 5.3 Risk Manager (Stage 3)

**File**: `risk_manager/risk_manager.py`

**Purpose**: Takes the Portfolio Architect's output and performs comprehensive risk analysis using real-time news, macro-economic indicators, and geopolitical data.

**How it works**:
1. Connects to an **MCP Gateway** (not a direct MCP Server) that routes tool calls to a Lambda function
2. The Gateway exposes 3 tools (defined in `target_config.py`):
   - `get_product_news(ticker)` — Latest news articles for an ETF
   - `get_market_data()` — 7 macro indicators (treasury yields, VIX, oil, gold, S&P 500, USD index)
   - `get_geopolitical_indicators()` — 5 regional ETFs (China, emerging markets, Europe, Japan, Korea)
3. The LLM analyzes all data and produces 2 economic scenarios with probability estimates and portfolio adjustment plans

**Lambda Function** (`risk_manager/lambda/lambda_function.py`):
- Runs in AWS Lambda with a yfinance Lambda Layer
- `lambda_handler()` routes calls based on tool name extracted from `context.client_context`
- Each function fetches real-time data via yfinance

**Architecture**: Agent → MCP Gateway → Lambda → yfinance

**Output** (example):
```json
{
  "scenario1": {
    "name": "Tech Correction",
    "description": "Rising interest rates cause tech valuations to compress...",
    "probability": "35%",
    "allocation_management": {"QQQ": 30, "XLV": 45, "SOXX": 25},
    "reason": "Shift toward defensive healthcare..."
  },
  "scenario2": {
    "name": "AI Boom Acceleration",
    "description": "Strong AI adoption drives semiconductor demand...",
    "probability": "40%",
    "allocation_management": {"QQQ": 45, "XLV": 25, "SOXX": 30},
    "reason": "Increase semiconductor exposure..."
  }
}
```

---

### 5.4 Fund Manager — Orchestrator (Stage 4)

**File**: `fund_manager/fund_manager.py`

**Purpose**: Orchestrates all 3 agents in sequence and manages consultation history.

**How it works**:
1. **LangGraph StateGraph** with 3 nodes: `financial` → `portfolio` → `risk`
2. Each node:
   - Calls `agent_client.call_agent_with_streaming()` which invokes the agent's AgentCore Runtime endpoint
   - Streams all events back to the caller via `get_stream_writer()`
   - Saves the conversation to AgentCore Memory
3. **State** is passed between nodes via `FundManagementState` TypedDict:
   ```python
   class FundManagementState(TypedDict):
       user_input: Dict[str, Any]
       session_id: str
       financial_analysis: str        # Stage 1 output
       portfolio_recommendation: str  # Stage 2 output
       risk_analysis: str             # Stage 3 output
   ```
4. **Memory**: Each node calls `save_to_memory()` which uses `memory_client.create_event()` to store the conversation pair (request + response) in AgentCore Memory. The SUMMARY strategy auto-summarizes the session.

**Retry logic**: `call_agent_with_streaming()` retries up to 3 times with exponential backoff.

**AgentCore Memory** (`fund_manager/agentcore_memory/deploy_agentcore_memory.py`):
- Creates a Memory resource with SUMMARY strategy
- Namespace pattern: `fund/session/{sessionId}`
- Auto-summarizes all events in a session into a condensed summary
- Retrievable via `memory_client.retrieve_memories()` for the History UI

---

## 6. Shared Utilities

### `shared/runtime_utils.py`
- `create_agentcore_runtime_role(agent_name, region)` — Creates an IAM role with permissions for:
  - Bedrock model invocation
  - AgentCore Runtime operations
  - AgentCore Memory operations
  - ECR image access
  - CloudWatch logging
  - X-Ray tracing
  - Workload identity tokens

### `shared/cognito_utils.py`
- `get_or_create_user_pool()` — Manages Cognito User Pool lifecycle
- `get_or_create_resource_server()` — Creates OAuth2 resource servers with scopes
- `get_or_create_m2m_client()` — Creates Machine-to-Machine OAuth2 clients (client_credentials grant)
- `get_token()` — Acquires OAuth2 access tokens

### `shared/gateway_utils.py`
- `create_agentcore_gateway_role()` — IAM role for MCP Gateway with Lambda invoke permissions
- `delete_existing_gateway()` — Safely deletes existing Gateway + Targets
- `create_gateway()` — Creates MCP Gateway with JWT authentication

---

## 7. AWS Services Map

```
┌─────────────────────────────────────────────────────────────────┐
│                        AWS Account                              │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Bedrock AgentCore Runtime                              │   │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐   │   │
│  │  │ Financial    │ │ Portfolio    │ │ Risk         │   │   │
│  │  │ Analyst      │ │ Architect    │ │ Manager      │   │   │
│  │  └──────────────┘ └──────────────┘ └──────────────┘   │   │
│  │  ┌──────────────┐                                      │   │
│  │  │ Fund Manager │                                      │   │
│  │  └──────────────┘                                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌──────────────────┐  ┌──────────────────┐                    │
│  │ AgentCore MCP    │  │ AgentCore MCP    │                    │
│  │ Server (ETF data)│  │ Gateway (Risk)   │                    │
│  │ (FastMCP+yfinance│  │   ↓              │                    │
│  └──────────────────┘  │ Gateway Target   │                    │
│                        │   ↓              │                    │
│  ┌──────────────────┐  │ Lambda Function  │                    │
│  │ AgentCore Memory │  │   + Layer        │                    │
│  │ (SUMMARY)        │  └──────────────────┘                    │
│  └──────────────────┘                                          │
│                                                                 │
│  ┌──────────────────┐  ┌──────────────────┐                    │
│  │ Cognito          │  │ IAM Roles        │                    │
│  │ (User Pool +     │  │ (per agent +     │                    │
│  │  M2M clients)    │  │  gateway)        │                    │
│  └──────────────────┘  └──────────────────┘                    │
│                                                                 │
│  ┌──────────────────┐  ┌──────────────────┐                    │
│  │ ECR              │  │ CloudWatch       │                    │
│  │ (Agent images)   │  │ (Logs)           │                    │
│  └──────────────────┘  └──────────────────┘                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. Data Flow — End-to-End

```
USER (Streamlit form)
  │
  │  POST: {amount, target, age, experience, purpose, sectors}
  ▼
FUND MANAGER app.py
  │  invoke_agent_runtime(fund_manager_arn, payload)
  ▼
FUND MANAGER Runtime (LangGraph)
  │
  ├─[1] financial_node
  │     │  invoke_agent_runtime(financial_analyst_arn, user_input)
  │     ▼
  │   FINANCIAL ANALYST Runtime
  │     │  LLM + calculator tool → risk_profile JSON
  │     │  save_to_memory(session, "financial", input, output)
  │     ▼
  │   Return: financial_analysis (JSON string)
  │
  ├─[2] portfolio_node
  │     │  invoke_agent_runtime(portfolio_architect_arn, financial_analysis)
  │     ▼
  │   PORTFOLIO ARCHITECT Runtime
  │     │  LLM → selects 5 ETFs
  │     │  ├── MCP: analyze_etf_performance("QQQ") → Monte Carlo result
  │     │  ├── MCP: analyze_etf_performance("XLV") → Monte Carlo result
  │     │  ├── MCP: analyze_etf_performance("SOXX") → Monte Carlo result
  │     │  ├── MCP: analyze_etf_performance("VGT") → Monte Carlo result
  │     │  ├── MCP: analyze_etf_performance("IBB") → Monte Carlo result
  │     │  └── MCP: calculate_correlation(["QQQ","XLV","SOXX","VGT","IBB"])
  │     │  LLM → selects best 3, allocates weights, scores portfolio
  │     │  save_to_memory(session, "portfolio", input, output)
  │     ▼
  │   Return: portfolio_recommendation (JSON string)
  │
  ├─[3] risk_node
  │     │  invoke_agent_runtime(risk_manager_arn, portfolio_recommendation)
  │     ▼
  │   RISK MANAGER Runtime
  │     │  LLM → calls tools via MCP Gateway:
  │     │  ├── get_product_news("QQQ") → Lambda → yfinance news
  │     │  ├── get_product_news("XLV") → Lambda → yfinance news
  │     │  ├── get_product_news("SOXX") → Lambda → yfinance news
  │     │  ├── get_market_data() → Lambda → 7 macro indicators
  │     │  └── get_geopolitical_indicators() → Lambda → 5 regional ETFs
  │     │  LLM → analyzes all data → 2 scenarios with adjusted allocations
  │     │  save_to_memory(session, "risk", input, output)
  │     ▼
  │   Return: risk_analysis (JSON string)
  │
  └─ Stream all events back to Streamlit UI
       │
       ▼
     STREAMLIT renders incrementally:
       - Financial analysis results (risk profile, sectors)
       - ETF Monte Carlo charts (bar charts)
       - Correlation matrix heatmap
       - Portfolio pie chart + scores
       - News tables, macro indicators, geopolitical data
       - Risk scenario pie charts + adjustment strategies
       - "Analysis Complete" banner
```

---

## 9. Deployment Pipeline

### One-Click Deploy (`deploy_all.py`)

```
python deploy_all.py
```

Runs these steps in order:

| Step | Command | What it does |
|------|---------|-------------|
| 1 | `aws sts get-caller-identity` | Verify AWS credentials |
| 2 | `python deploy.py` in `financial_analyst/` | Create IAM role → Build Docker image → Push to ECR → Create AgentCore Runtime → Wait for READY |
| 3a | `python deploy_mcp.py` in `portfolio_architect/mcp_server/` | Deploy MCP Server (FastMCP + yfinance) to AgentCore with Cognito auth |
| 3b | `python deploy.py` in `portfolio_architect/` | Deploy Portfolio Architect agent (with MCP env vars) |
| 4a | `python deploy_lambda_layer.py` in `risk_manager/lambda_layer/` | Create Lambda Layer with yfinance |
| 4b | `python deploy_lambda.py` in `risk_manager/lambda/` | Deploy Lambda function |
| 4c | `python deploy_gateway.py` in `risk_manager/gateway/` | Create MCP Gateway + Target pointing to Lambda |
| 4d | `python deploy.py` in `risk_manager/` | Deploy Risk Manager agent (with Gateway env vars) |
| 5a | `python deploy_agentcore_memory.py` in `fund_manager/agentcore_memory/` | Create AgentCore Memory with SUMMARY strategy |
| 5b | `python deploy.py` in `fund_manager/` | Deploy Fund Manager orchestrator (with all agent ARNs + memory ID) |

Each `deploy.py` saves a `deployment_info.json` file with the ARN and other metadata needed by downstream agents.

### Cleanup (`cleanup_all.py`)

Runs cleanup in **reverse order** (Fund Manager → Risk Manager → Portfolio Architect → Financial Analyst) to handle dependencies.

---

## 10. Authentication & Security

### MCP Server / Gateway Authentication
Both the MCP Server (Portfolio Architect) and MCP Gateway (Risk Manager) use **AWS Cognito OAuth2** with the **client_credentials** grant type.

**Flow**:
1. During deployment, `cognito_utils.py` creates:
   - A Cognito User Pool
   - A Resource Server with scopes (read, write)
   - An M2M (Machine-to-Machine) client with client_id + client_secret
2. At runtime, the agent:
   - POSTs to `https://<pool-domain>.auth.<region>.amazoncognito.com/oauth2/token` with client_credentials
   - Receives a JWT access token
   - Passes it as `Authorization: Bearer <token>` in MCP HTTP requests

### IAM Roles
Each agent has its own IAM role (`agentcore-runtime-<agent_name>-role`) with least-privilege permissions:
- Bedrock model invocation
- AgentCore Runtime/Memory operations
- ECR image access
- CloudWatch/X-Ray observability
- Workload identity tokens

### Gateway JWT Validation
The MCP Gateway validates incoming JWTs using the Cognito discovery URL (JWKS endpoint), ensuring only authorized clients can invoke tools.

---

## 11. Streaming & Real-Time UI

### Streaming Protocol
Every agent uses the same streaming event protocol:

```json
{"type": "text_chunk", "data": "The risk profile is..."}
{"type": "tool_use", "tool_name": "calculator", "tool_use_id": "abc123", "tool_input": {...}}
{"type": "tool_result", "tool_use_id": "abc123", "status": "success", "content": [...]}
{"type": "streaming_complete", "result": "{\"risk_profile\": \"Aggressive\", ...}"}
```

The Fund Manager adds orchestration events:
```json
{"type": "node_start", "agent_name": "financial", "session_id": "session_20260315_143022"}
{"type": "node_complete", "agent_name": "financial", "result": "{...}"}
```

### Streamlit UI Rendering
`fund_manager/app.py` processes the stream in a loop:
1. **text_chunk** → Appends to current thinking display (chat bubble)
2. **tool_use** → Maps tool_use_id to tool name
3. **tool_result** → Renders tool-specific visualization:
   - `calculator` → Code block with input/output
   - `analyze_etf_performance` → 4 metrics + bar chart
   - `calculate_correlation` → Heatmap
   - `get_product_news` → Data table
   - `get_market_data` → Metric cards
   - `get_geopolitical_indicators` → Metric cards
4. **node_start** → Creates new expander section for agent
5. **node_complete** → Renders final results (charts, scores) outside expander
6. **streaming_complete** → Processes final JSON output

---

## 12. Key Design Patterns

### 1. Singleton Agent Initialization
```python
analyst = None

@app.entrypoint
async def financial_analyst(payload):
    global analyst
    if analyst is None:
        analyst = FinancialAnalyst()  # Only created once
    ...
```
Agents are expensive to initialize (model setup, MCP client connection). The singleton pattern ensures this happens once per container lifecycle.

### 2. Async Generator Streaming
```python
async def analyze_financial_situation_async(self, user_input):
    async for event in self.agent.stream_async(user_input_str):
        if "data" in event:
            yield {"type": "text_chunk", "data": event["data"]}
        ...
```
All agents use Python async generators (`yield`) to produce events incrementally, enabling real-time UI updates.

### 3. JSON Extraction Helper
```python
def extract_json_from_text(text_content):
    start_idx = text_content.find('{')
    end_idx = text_content.rfind('}') + 1
    if start_idx != -1 and end_idx > start_idx:
        return text_content[start_idx:end_idx]
    return text_content
```
LLMs sometimes wrap JSON in explanatory text. This helper extracts the JSON portion reliably.

### 4. Environment-First Configuration
```python
arns = {
    "financial": os.getenv("FINANCIAL_ANALYST_ARN"),
    "portfolio": os.getenv("PORTFOLIO_ARCHITECT_ARN"),
    "risk": os.getenv("RISK_MANAGER_ARN")
}
if all(arns.values()):
    return arns
# Fallback: load from JSON files
```
Environment variables take precedence (for Docker/cloud deployment), with JSON file fallback (for local development).

### 5. Retry with Exponential Backoff
```python
for attempt in range(max_retries):
    try:
        response = self.client.invoke_agent_runtime(...)
        ...
    except Exception as e:
        if attempt < max_retries - 1:
            time.sleep(retry_delay * (2 ** attempt))
        else:
            raise
```
Network calls to AgentCore Runtime use retry logic to handle transient failures.

### 6. LangGraph State Machine
```python
workflow = StateGraph(FundManagementState)
workflow.add_node("financial", financial_node)
workflow.add_node("portfolio", portfolio_node)
workflow.add_node("risk", risk_node)
workflow.set_entry_point("financial")
workflow.add_edge("financial", "portfolio")
workflow.add_edge("portfolio", "risk")
workflow.add_edge("risk", END)
```
Clear, declarative workflow definition. Easy to extend (add nodes, conditional edges, parallel branches).

---

## 13. Configuration Management

### `config.py` — Single source of truth
```python
class Config:
    REGION = "us-west-2"
    FINANCIAL_ANALYST_NAME = "financial_analyst"
    PORTFOLIO_ARCHITECT_NAME = "portfolio_architect"
    RISK_MANAGER_NAME = "risk_manager"
    FUND_MANAGER_NAME = "fund_manager"
    MCP_SERVER_NAME = "mcp_server"
    GATEWAY_NAME = "gateway-risk-manager"
    TARGET_NAME = "target-risk-manager"
    LAMBDA_FUNCTION_NAME = "lambda-agentcore-risk-manager"
    LAMBDA_LAYER_NAME = "layer-yfinance"
    MEMORY_NAME = "FundManager_Memory"
```

All deployment scripts import this class to ensure consistent naming across AWS resources.

### `deployment_info.json` — Deployment metadata
Each agent's `deploy.py` saves a JSON file after successful deployment:
```json
{
  "agent_name": "financial_analyst",
  "agent_arn": "arn:aws:bedrock-agentcore:us-west-2:123456789:runtime/...",
  "agent_id": "abc123",
  "region": "us-west-2",
  "iam_role_name": "agentcore-runtime-financial_analyst-role",
  "ecr_repo_name": "financial_analyst",
  "deployed_at": "2026-03-15 14:30:22"
}
```
Downstream agents load these files to find ARNs of their dependencies.

---

## 14. How to Run Locally

### Prerequisites
1. **AWS Account** with Bedrock access enabled (GPT-OSS 120B model)
2. **AWS CLI** configured: `aws configure` (or set `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
3. **Python 3.8+** (project uses 3.12)

### Setup
```bash
cd /Users/harsgupta/Desktop/AI-Fund-Manger

# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Deploy (all agents)
```bash
python deploy_all.py
```
This takes 15-30 minutes as it creates IAM roles, builds Docker images, pushes to ECR, and waits for each runtime to become READY.

### Run the Web App
```bash
cd fund_manager
streamlit run app.py
# Open http://localhost:8501
```

### Run Individual Agents (standalone UIs)
```bash
cd financial_analyst && streamlit run app.py    # http://localhost:8501
cd portfolio_architect && streamlit run app.py  # http://localhost:8501
cd risk_manager && streamlit run app.py         # http://localhost:8501
```

### Cleanup
```bash
python cleanup_all.py
```

---

## 15. Glossary

| Term | Definition |
|------|-----------|
| **AgentCore Runtime** | AWS managed service for hosting AI agent containers with built-in scaling |
| **Strands Agent** | Python agent framework that combines an LLM with tools and a system prompt |
| **LangGraph** | Library for building multi-agent workflows as state machines (directed graphs) |
| **MCP (Model Context Protocol)** | Open protocol standardizing how AI agents access tools and data sources |
| **FastMCP** | Python framework for building MCP-compliant servers quickly |
| **MCP Gateway** | AWS service that routes MCP tool calls to backend services (like Lambda) |
| **Monte Carlo Simulation** | Statistical technique that uses random sampling to model uncertainty (here: 1000 simulations of future ETF returns) |
| **Cognito M2M** | Machine-to-Machine OAuth2 flow using client_credentials grant (no human login) |
| **SUMMARY Strategy** | AgentCore Memory strategy that auto-summarizes all events in a session |
| **yfinance** | Python library wrapping Yahoo Finance API for stock/ETF data |
| **ETF** | Exchange-Traded Fund — a basket of securities traded like a stock |
| **Streamlit** | Python framework for building data-driven web apps with minimal code |
| **Plotly** | Interactive charting library (pie charts, heatmaps, bar charts) |
| **ECR** | Elastic Container Registry — AWS Docker image hosting |
| **Lambda Layer** | Reusable package of dependencies attached to Lambda functions |

---

*End of report. This document covers every source file in the project and should give you a solid foundation for understanding, modifying, and extending the system.*
