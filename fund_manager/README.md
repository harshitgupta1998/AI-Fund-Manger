## What is the Fund Manager Agent?

The Fund Manager is an intelligent AI orchestrator that integrates all three specialized agents (Financial Analyst, Portfolio Architect, and Risk Manager) into a seamless, automated workflow. It serves as the central hub that coordinates comprehensive fund management services, processes your complete financial information through all analysis stages, and maintains permanent records of all consultations for future reference.

This agent serves as the **final orchestration stage** in the AI Fund Manager system, providing end-to-end intelligent fund management with complete history tracking and AI-powered memory management.

### Core Features

- **Multi-Agent Orchestration**: Seamless coordination of 3 specialized agents through LangGraph workflows
- **Real-time Streaming**: Live visualization of each agent's reasoning, tool usage, and decision-making process
- **Automatic Memory Management**: Permanent consultation history with AI-powered summarization
- **Comprehensive Reporting**: Complete fund management analysis in one integrated report
- **Full Automation**: Complete end-to-end process with single user input

## How It Works

The Fund Manager orchestrates a sophisticated 4-stage workflow that coordinates all AI agents:

### Agent Workflow

1. **User Input Reception**: You provide comprehensive financial information:
   - Investment amount and target amount
   - Age and investment experience
   - Investment purpose and timeline
   - Investment sector preferences

2. **Financial Analysis Stage** (Agent: Financial Analyst)
   - Agent analyzes your financial profile
   - Determines your risk profile (Conservative/Moderate/Aggressive)
   - Calculates required annual returns
   - Recommends suitable investment sectors
   - Output: Risk profile & sector recommendations

3. **Portfolio Design Stage** (Agent: Portfolio Architect)
   - Agent receives risk profile from Financial Analyst
   - Selects 3 optimal ETFs using real-time market data
   - Performs Monte Carlo simulations for risk analysis
   - Calculates diversification benefits through correlation analysis
   - Determines optimal investment allocations
   - Output: 3-ETF portfolio with allocations and scores

4. **Risk Analysis Stage** (Agent: Risk Manager)
   - Agent receives portfolio from Portfolio Architect
   - Analyzes latest market news for each ETF
   - Monitors macroeconomic indicators
   - Assesses geopolitical risks across regions
   - Develops economic scenarios
   - Recommends portfolio adjustments for different scenarios
   - Output: Risk scenarios & adjustment strategies

5. **Memory & Reporting**:
   - All analysis results compiled into comprehensive report
   - Consultation automatically summarized and stored
   - History accessible for future reference and tracking

### Processing Architecture

```
Your Financial Information
    ↓
[Fund Manager Orchestrator - LangGraph]
    ↓
Stage 1: Financial Analyst Agent
├─ Analyze financial profile
├─ Calculate risk profile
└─ Recommend sectors → Output to Portfolio Architect
    ↓
Stage 2: Portfolio Architect Agent
├─ Retrieve risk profile
├─ Select ETFs using real-time data
├─ Run Monte Carlo simulations
├─ Analyze correlations
└─ Calculate allocations → Output to Risk Manager
    ↓
Stage 3: Risk Manager Agent
├─ Retrieve portfolio
├─ Collect market news
├─ Monitor economic indicators
├─ Assess geopolitical risks
├─ Develop scenarios
└─ Calculate adjustments → Output to Fund Manager
    ↓
Stage 4: Memory & Reporting
├─ Compile comprehensive report
├─ Summarize consultation
├─ Store in permanent memory
└─ Display to user
    ↓
Complete Fund Management Report with History
```

## Technology Stack

- **Orchestration Framework**: LangGraph (agent workflow coordination)
- **AI Framework**: AWS Bedrock AgentCore Runtime
- **Memory System**: AgentCore Memory with SUMMARY strategy
- **LLM Models**: GPT-OSS
- **Data Protocol**: MCP (Model Context Protocol) via integrated agents
- **Data Source**: yfinance (through agent integrations)
- **Infrastructure**: AWS (serverless, auto-scaling)
- **Authentication**: AWS Cognito JWT
- **UI Framework**: Streamlit
- **Language**: Python

## Setup Instructions

### Prerequisites

- AWS Account with Bedrock, Lambda, and API Gateway access
- Python 3.8 or higher
- AWS CLI configured with credentials
- All 3 agent systems deployed:
  - Financial Analyst
  - Portfolio Architect (with MCP Server)
  - Risk Manager (with Lambda Layer, Lambda, Gateway)

### Step 1: Install Dependencies

```bash
# From root directory
pip install -r requirements.txt

# Configure AWS credentials
aws configure

# Navigate to fund_manager folder
cd fund_manager
```

### Step 2: Deploy AgentCore Memory (Required First)

The AgentCore Memory system stores consultation history:

```bash
# Deploy the Memory service
cd agentcore_memory
python deploy_agentcore_memory.py

# Verify deployment
cat deployment_info.json
```

**What this does:**

- Deploys the Memory service to AWS Bedrock
- Configures SUMMARY strategy for automatic summarization
- Sets up namespace structure for consultation storage
- Enables history retrieval and tracking

### Step 3: Deploy Fund Manager Agent

```bash
# Deploy the Fund Manager orchestrator
cd ..
python deploy.py

# Verify deployment (creates deployment_info.json)
cat deployment_info.json
```

**What this does:**

- Deploys the Fund Manager agent to AWS Bedrock
- Loads references to all 3 integrated agents
- Configures LangGraph workflow orchestration
- Sets up memory integration for history tracking

### Step 4: Run the Streamlit Web App

```bash
# Start the web interface
streamlit run app.py

# Access the app at http://localhost:8501
```

### Step 5: Use the Application

1. Open your browser to `http://localhost:8501`
2. Enter your complete financial information in the form
3. Click "Start Fund Consultation" to begin
4. Watch real-time progress as each agent runs
5. Review your complete comprehensive fund management report
6. Access past consultations from the history panel

### Sample Output Breakdown

```
┌──────────────────────────────────────────────────────────┐
│      COMPREHENSIVE FUND MANAGEMENT REPORT                │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ ════════ STAGE 1: FINANCIAL ANALYSIS ════════            │
│ Risk Profile: MODERATE                                   │
│ • Age: 35-45 years                                       │
│ • Experience: 5-10 years                                 │
│ • Required Return: 18.2% annually                        │
│ • Recommended Sectors: [Tech] [Healthcare] [Finance]    │
│                                                          │
│ ════════ STAGE 2: PORTFOLIO DESIGN ════════              │
│ Your Optimized Portfolio:                                │
│ • VTI (US Total Market)         40%  (Profitability: 8.2)│
│ • VXUS (International)          35%  (Risk Mgmt: 8.1)    │
│ • BND (US Bonds)                25%  (Diversification: 8.5)│
│                                                          │
│ Expected Return: 16.8% annually                          │
│ Portfolio Risk Score: 8.3/10 (Well-balanced)             │
│                                                          │
│ ════════ STAGE 3: RISK ANALYSIS ════════                │
│ Market Environment: Stable growth, moderate volatility   │
│                                                          │
│ SCENARIO 1: Continued Growth (65% probability)           │
│ Action: Maintain current 40-35-25 allocation             │
│ Expected Return: +18.5%                                  │
│                                                          │
│ SCENARIO 2: Market Correction (35% probability)          │
│ Action: Adjust to 25-20-55 (reduce equity, increase bonds)│
│ Expected Return: +2.3%                                   │
│                                                          │
│ Key Risks: Interest rate spike, tech earnings weakness   │
│                                                          │
│ ════════ EXECUTIVE SUMMARY ════════                      │
│ Your moderate risk profile aligns perfectly with the     │
│ recommended portfolio. This well-diversified mix of      │
│ US equities, international stocks, and bonds should     │
│ achieve your 18.2% return target while managing risk.   │
│                                                          │
│ Recommended Actions:                                     │
│ 1. Allocate initial capital according to 40-35-25 split │
│ 2. Monitor quarterly and rebalance annually              │
│ 3. Adjust if VIX exceeds 25 or rates spike               │
│ 4. Review again in 6 months or if major market shift     │
│                                                          │
│ ════════ CONSULTATION METADATA ════════                  │
│ Consultation ID: fund-20260315-a7f3b2e1                  │
│ Timestamp: March 15, 2026, 14:30 UTC                     │
│ Status: Completed & Saved to History                     │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## AgentCore Memory System

### SUMMARY Strategy

The Fund Manager uses an intelligent memory system to track all consultations:

**Memory Features**:

- **Automatic Summarization**: Each consultation is automatically analyzed and summarized
- **Permanent Storage**: Summaries stored permanently in AgentCore Memory
- **Smart Organization**: Consultations organized by date, risk profile, and sectors
- **History Access**: Review past consultations and track portfolio performance over time
- **Pattern Recognition**: AI identifies trends and common recommendations

**Memory Namespace**: `fund/session/{sessionId}`

**Storage Tiers**:

- Short-term: Individual agent interactions (7 days)
- Long-term: Consultation summaries (permanent)
- Metadata: Tags, categories, and searchable attributes

### History Panel Features

In the Streamlit app, you can:

1. View all past consultations with timestamps
2. Review automatically generated summaries
3. Compare recommendations across consultations
4. Track allocation changes over time
5. Export consultation reports
