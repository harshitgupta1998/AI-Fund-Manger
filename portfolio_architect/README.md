## What is the Portfolio Architect Agent?

The Portfolio Architect is an intelligent AI agent that designs optimal investment portfolios using real-time ETF market data. Based on the risk profile and recommendations from the Financial Analyst, this agent constructs a diversified portfolio of 3 ETFs tailored to your financial goals, risk tolerance, and target returns.

This agent serves as the **second stage** in the AI Fund Manager system, transforming financial analysis into actionable portfolio allocations with real-time market data.

### Core Features

- **Real-time ETF Analysis**: Live ETF data retrieval and analysis through yfinance integrated via MCP Server
- **Smart ETF Selection**: Intelligent selection of 3 optimal ETFs from hundreds of candidates
- **Monte Carlo Simulation**: Advanced risk analysis with 1000 simulations for accurate predictions
- **Correlation Analysis**: Diversification optimization through correlation matrix measurement
- **Portfolio Evaluation**: Comprehensive assessment on profitability, risk management, and diversification
- **Weight Allocation**: Optimal investment weight distribution across selected ETFs

## How It Works

The Portfolio Architect agent operates through a sophisticated 6-step portfolio design process:

### Agent Workflow

1. **Risk Profile Reception**: The agent receives your risk profile and financial targets from the Financial Analyst:
   - Risk category (Conservative, Moderate, Aggressive)
   - Required annual return percentage
   - Recommended investment sectors
   - Risk profile reasoning and constraints

2. **Candidate ETF Selection**: The agent identifies 5 promising ETF candidates that align with:
   - Your risk profile and sector preferences
   - Target return requirements
   - Market conditions and historical performance
   - Diversification potential

3. **Performance Analysis**: For each candidate ETF, the agent:
   - Runs 1000 Monte Carlo simulations using 2 years of daily data
   - Calculates expected annual returns
   - Determines loss probability (risk of principal loss)
   - Analyzes return distribution across 7 ranges
   - Measures historical volatility

4. **Correlation Analysis**: The agent measures diversification effects by:
   - Generating a 5×5 correlation matrix
   - Calculating correlation coefficients between ETF pairs (-1 to 1)
   - Identifying assets with low correlation for better diversification
   - Measuring combined portfolio risk reduction

5. **Optimal Portfolio Construction**: The agent selects 3 final ETFs by:
   - Balancing profitability with risk management
   - Ensuring diversification through uncorrelated assets
   - Allocating investment weights in optimal percentages
   - Meeting your target return with minimum portfolio risk

6. **Portfolio Evaluation**: Final assessment across 3 dimensions:
   - **Profitability (1-10)**: Expected return achievement potential
   - **Risk Management (1-10)**: Downside protection and volatility control
   - **Diversification (1-10)**: Correlation-based risk reduction effectiveness

### Processing Architecture

```
Financial Analyst Results
    ↓
[AgentCore Runtime]
    ↓
Portfolio Architect Agent
    ├─ Select 5 ETF candidates
    ├─ [MCP Server → yfinance]
    │  ├─ Monte Carlo simulations (1000x)
    │  ├─ Performance analysis
    │  └─ Correlation matrix
    ├─ Calculate optimal weights
    └─ Evaluate portfolio (3 metrics)
    ↓
Optimized 3-ETF Portfolio with Allocations
```

## Technology Stack

- **AI Framework**: AWS Bedrock AgentCore Runtime (dual runtimes)
  - Portfolio Architect Runtime (main agent)
  - MCP Server Runtime (ETF data retrieval)
- **LLM Model**: OpenAI GPT-OSS 120B
- **Data Protocol**: MCP (Model Context Protocol)
- **Data Source**: yfinance (real-time market data)
- **Analysis Method**: Monte Carlo Simulation (1000 iterations)
- **Infrastructure**: AWS (serverless, auto-scaling)
- **Authentication**: AWS Cognito JWT
- **UI Framework**: Streamlit
- **Language**: Python

## Setup Instructions

### Prerequisites

- AWS Account with Bedrock access
- Python 3.8 or higher
- AWS CLI configured with credentials
- Financial Analyst agent deployed (generates deployment_info.json)

### Step 1: Install Dependencies

```bash
# From root directory
cd ..
pip install -r requirements.txt

# Configure AWS credentials
aws configure

# Navigate to portfolio_architect folder
cd portfolio_architect
```

### Step 2: Deploy MCP Server (Required First)

The MCP Server provides real-time ETF data access and must be deployed before the main agent:

```bash
# Deploy the MCP Server
cd mcp_server
python deploy_mcp.py

# Verify MCP deployment
cat mcp_deployment_info.json
```

**What this does:**

- Deploys the Model Context Protocol server to AWS Bedrock
- Enables real-time ETF data retrieval from yfinance
- Configures Monte Carlo simulation capabilities
- Sets up correlation analysis tools

### Step 3: Deploy Portfolio Architect Agent

```bash
# Deploy the Portfolio Architect agent
cd ..
python deploy.py

# Verify deployment
cat deployment_info.json
```

**What this does:**

- Deploys the Portfolio Architect agent to AWS Bedrock
- Links the agent to the MCP Server for data access
- Configures the LLM model and analysis tools
- Generates deployment metadata for the Streamlit app

### Step 4: Run the Streamlit Web App

```bash
# Start the web interface
streamlit run app.py

# Access the app at http://localhost:8501
```

### Step 5: Use the Application

1. Open your browser to `http://localhost:8501`
2. Input the Financial Analyst results (risk profile, target return, sectors)
3. Click "Design Portfolio" to run the agent
4. View your optimized 3-ETF portfolio with allocations and analysis

### Sample Output Breakdown

```
┌──────────────────────────────────────────────────┐
│     YOUR OPTIMIZED PORTFOLIO RESULTS             │
├──────────────────────────────────────────────────┤
│                                                  │
│ PORTFOLIO COMPOSITION:                           │
│ • VTI (US Total Market)         40%              │
│ • VXUS (International)          35%              │
│ • BND (US Bonds)                25%              │
│                                                  │
│ PROJECTED PERFORMANCE:                           │
│ Expected Annual Return: 16.8%                    │
│ Probability of Loss: 8.2%                        │
│ Portfolio Volatility: 14.5%                      │
│                                                  │
│ CORRELATION ANALYSIS:                            │
│ VTI-VXUS Correlation: 0.72 (Moderate)           │
│ VTI-BND  Correlation: 0.15 (Low - Good!)        │
│ VXUS-BND Correlation: 0.08 (Very Low - Good!)   │
│ Overall Diversification Score: 7.8/10            │
│                                                  │
│ PORTFOLIO SCORES:                                │
│ Profitability:    8.2/10 (Meets target return)  │
│ Risk Management:  8.1/10 (Conservative approach)│
│ Diversification:  8.5/10 (Well-balanced)        │
│ ─────────────────────────────                    │
│ OVERALL SCORE:    8.3/10                         │
│                                                  │
│ RECOMMENDATION:                                  │
│ This portfolio aligns with your moderate risk    │
│ profile and achieves your 18.2% target through   │
│ a well-diversified mix. Monitor quarterly and    │
│ rebalance annually.                              │
│                                                  │
└──────────────────────────────────────────────────┘
```

## MCP Server & Tools

### MCP Server Role

The MCP Server acts as the bridge between the Portfolio Architect agent and financial market data. It provides:

- Real-time ETF data retrieval from yfinance
- Monte Carlo simulation engine for risk analysis
- Correlation matrix calculations
- Performance metrics computation

### Available Tools

#### analyze_etf_performance(ticker)

**Purpose**: Comprehensive performance analysis for an individual ETF

**Analysis Details**:

- Expected annual return (%)
- Loss probability (principal loss possibility, %)
- Historical annual return (%)
- Return volatility (%)
- Return distribution across 7 ranges:
  - Below -20%
  - -20% to -10%
  - -10% to 0%
  - 0% to 10%
  - 10% to 20%
  - 20% to 30%
  - Above 30%

**Data Used**: 2 years of daily returns (minimum 500+ trading days)

**Simulation**: 1000 Monte Carlo iterations

#### calculate_correlation(tickers)

**Purpose**: Measure diversification benefits between multiple ETFs

**Analysis Details**:

- 5×5 correlation matrix (for up to 5 ETFs)
- Correlation coefficient for each pair (-1.0 to 1.0)
- Interpretation guide:
  - 1.0 = Perfect positive correlation (no diversification)
  - 0.0 = No correlation (some diversification)
  - -1.0 = Perfect negative correlation (maximum diversification)
- Diversification effect measurement

**Data Used**: 2 years of daily returns (minimum 100+ days of common data)

**Calculation Method**: Pearson correlation coefficient
