## What is the Risk Manager Agent?

The Risk Manager is an intelligent AI agent that monitors real-time market conditions, analyzes financial news, and evaluates macroeconomic indicators to identify risks to your portfolio. Based on the portfolio design from the Portfolio Architect, this agent develops strategic risk scenarios and recommends portfolio adjustment strategies to protect your investments under various economic conditions.

This agent serves as the **third stage** in the AI Fund Manager system, transforming portfolio allocations into risk-aware strategies by anticipating market challenges and economic shifts.

### Core Features

- **Real-time News Analysis**: Automated collection and analysis of latest news affecting your portfolio ETFs
- **Macroeconomic Monitoring**: Continuous tracking of key economic indicators (interest rates, volatility, commodities)
- **Geopolitical Risk Assessment**: Regional ETF analysis for global economic and political risk factors
- **Scenario Planning**: Development of 2 core economic scenarios with portfolio adjustment strategies
- **Risk Mitigation Strategies**: Specific allocation adjustments to protect portfolio performance under different conditions

## How It Works

The Risk Manager agent operates through a sophisticated 6-step risk analysis process:

### Agent Workflow

1. **Portfolio Reception & Analysis**: The agent receives your portfolio design from the Portfolio Architect:
   - 3 selected ETFs with allocation percentages
   - Portfolio performance scores (profitability, risk management, diversification)
   - Investment strategy and composition reasoning

2. **ETF-Specific News Collection**: For each of your 3 ETFs, the agent:
   - Retrieves latest 5 news articles using the get_product_news tool
   - Analyzes news sentiment and risk implications
   - Identifies sector-specific threats and opportunities
   - Flags major announcements or events affecting holdings

3. **Macroeconomic Indicator Analysis**: The agent monitors 7 key economic indicators:
   - **Interest Rates**: US 2-year & 10-year Treasury yields, Dollar Index
   - **Volatility & Commodities**: VIX Index, WTI crude oil, gold futures
   - **Equity Markets**: S&P 500 Index
   - Interprets current levels and trends relative to historical averages

4. **Geopolitical Risk Assessment**: The agent tracks 5 major regional ETFs:
   - China A-Shares (ASHR), Emerging Markets (EEM), Europe (VGK)
   - Japan (EWJ), South Korea (EWY)
   - Evaluates regional political/economic risks affecting global portfolios

5. **Scenario Development**: The agent synthesizes all data to create:
   - **Scenario 1**: Best-case or stable economic conditions
   - **Scenario 2**: Challenging or high-risk economic conditions
   - Probability assessment for each scenario
   - Expected portfolio performance under each scenario

6. **Portfolio Adjustment Strategy**: The agent recommends:
   - New allocation weights for each ETF under each scenario
   - Specific rationale for each adjustment
   - Implementation guidance
   - Timeline and conditions for rebalancing

### Processing Architecture

```
Portfolio Design Results
    ↓
[AgentCore Runtime + Gateway]
    ↓
Risk Manager Agent
    ├─ Retrieve portfolio ETFs
    ├─ [Lambda Functions → yfinance]
    │  ├─ News collection (per ETF)
    │  ├─ Macroeconomic indicators
    │  └─ Geopolitical data
    ├─ Analyze market sentiment
    ├─ Develop economic scenarios
    └─ Calculate adjustment strategies
    ↓
Risk Scenarios with Portfolio Recommendations
```

## Technology Stack

- **AI Framework**: AWS Bedrock AgentCore Runtime + Gateway
- **Gateway Protocol**: MCP (Model Context Protocol)
- **LLM Model**: OpenAI GPT-OSS 120B
- **Lambda Infrastructure**: AWS Lambda with layers for dependencies
- **Data Source**: yfinance (real-time market data and news)
- **Authentication**: AWS Cognito JWT OAuth2
- **Infrastructure**: AWS (serverless, auto-scaling)
- **UI Framework**: Streamlit
- **Language**: Python

## Setup Instructions

### Prerequisites

- AWS Account with Bedrock, Lambda, and API Gateway access
- Python 3.8 or higher
- AWS CLI configured with credentials
- Portfolio Architect agent deployed (generates deployment_info.json)

### Step 1: Install Dependencies

```bash
# From root directory
cd ..
pip install -r requirements.txt

# Configure AWS credentials
aws configure

# Navigate to risk_manager folder
cd risk_manager
```

### Step 2: Deploy Lambda Layer (Required First)

The Lambda Layer packages the yfinance library for Lambda functions:

```bash
# Deploy the Lambda Layer
cd lambda_layer
python deploy_lambda_layer.py

# Verify deployment
cat layer_deployment_info.json
```

**What this does:**

- Packages yfinance library for AWS Lambda
- Creates a layer that Lambda functions can reference
- Enables market data retrieval capabilities

### Step 3: Deploy Lambda Functions

The Lambda functions provide data retrieval capabilities:

```bash
# Deploy Lambda functions
cd ../lambda
python deploy_lambda.py

# Verify deployment
cat lambda_deployment_info.json
```

**What this does:**

- Deploys functions for news, market data, and geopolitical data retrieval
- Links to the yfinance Lambda Layer
- Configures IAM permissions for API access

### Step 4: Deploy MCP Gateway

The Gateway exposes Lambda functions as tools for the AI agent:

```bash
# Deploy the MCP Gateway
cd ../gateway
python deploy_gateway.py

# Verify deployment
cat gateway_deployment_info.json
```

**What this does:**

- Creates MCP Gateway to expose Lambda functions as AI tools
- Configures tool schemas and parameters
- Sets up authentication and authorization

### Step 5: Deploy Risk Manager Agent

```bash
# Deploy the Risk Manager agent
cd ..
python deploy.py

# Verify deployment (creates deployment_info.json)
cat deployment_info.json
```

**What this does:**

- Deploys the Risk Manager agent to AWS Bedrock
- Links the agent to the MCP Gateway for tool access
- Configures the LLM model and analysis parameters

### Step 6: Run the Streamlit Web App

```bash
# Start the web interface
streamlit run app.py

# Access the app at http://localhost:8501
```

### Step 7: Use the Application

1. Open your browser to `http://localhost:8501`
2. Input your portfolio design results from Portfolio Architect
3. Click "Analyze Risks" to run the agent
4. View your risk scenarios and adjustment strategies

### Sample Output Breakdown

```
┌─────────────────────────────────────────────────────┐
│     YOUR PORTFOLIO RISK ANALYSIS RESULTS            │
├─────────────────────────────────────────────────────┤
│                                                     │
│ CURRENT MARKET ENVIRONMENT:                         │
│ • 10-Year Treasury Yield: 4.25% (↑ rising)         │
│ • VIX Volatility Index: 18.5 (normal range)        │
│ • USD Index: 104.2 (strong dollar)                 │
│ • Oil Price: $82/barrel (stable)                    │
│                                                     │
│ ECONOMIC SCENARIOS:                                 │
│                                                     │
│ SCENARIO 1: Continued Expansion (65% probability)  │
│ Market Conditions: Strong earnings, stable rates   │
│ Recommended Allocation:                             │
│   • VTI: 45% (↑ increase exposure)                 │
│   • VXUS: 40% (maintain international)             │
│   • BND: 15% (reduce defensive bonds)              │
│ Expected Return: +18.5% annual                      │
│                                                     │
│ SCENARIO 2: Market Correction (35% probability)    │
│ Market Conditions: Rising rates, earnings miss      │
│ Recommended Allocation:                             │
│   • VTI: 25% (↓ reduce equity risk)                │
│   • VXUS: 20% (↓ de-risk international)            │
│   • BND: 55% (↑ increase defensive)                │
│ Expected Return: +2.3% annual                       │
│                                                     │
│ TOP RISKS IDENTIFIED:                               │
│ 1. Interest rate spike (Fed policy shift)           │
│ 2. China slowdown (emerging market exposure)        │
│ 3. Tech earnings weakness (sector concentration)    │
│                                                     │
│ RECOMMENDED ACTIONS:                                │
│ • Monitor Fed meeting dates (next: March 20)       │
│ • Set rebalance trigger if VIX > 25                │
│ • Consider reducing tech overweight                 │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## Lambda Tools Details

### get_product_news(ticker)

**Purpose**: Retrieve latest ETF-specific news and sentiment

**Data Retrieved**:

- Latest 5 news articles related to the ETF
- Article titles, summaries, and publication dates
- Source attribution and links
- Sentiment indicators (positive/negative/neutral)

**Data Source**: yfinance API
**Use Case**: Identify immediate risk factors and market sentiment for each holding

### get_market_data()

**Purpose**: Monitor key macroeconomic indicators

**Indicators Tracked** (7 total):

| Indicator              | Ticker | Category       | Purpose                             |
| ---------------------- | ------ | -------------- | ----------------------------------- |
| 2-Year Treasury Yield  | ^IRX   | Interest Rates | Short-term rate expectations        |
| 10-Year Treasury Yield | ^TNX   | Interest Rates | Long-term rate environment          |
| USD Strength Index     | DXY    | Interest Rates | Dollar strength effects             |
| VIX Volatility Index   | ^VIX   | Volatility     | Market fear/complacency             |
| WTI Crude Oil          | CL=F   | Commodities    | Energy/inflation trends             |
| Gold Futures           | GC=F   | Commodities    | Inflation hedge, risk-off indicator |
| S&P 500 Index          | ^GSPC  | Equity         | Overall market health               |

**Data Source**: yfinance
**Update Frequency**: Real-time market prices
**Use Case**: Develop economic scenarios based on current market conditions

### get_geopolitical_indicators()

**Purpose**: Assess regional economic and political risks

**Regional ETFs Monitored** (5 total):

| Region           | ETF Ticker | ETF Name             | Risk Factors                      |
| ---------------- | ---------- | -------------------- | --------------------------------- |
| China            | ASHR       | China A-Shares       | Regulatory, geopolitical tensions |
| Emerging Markets | EEM        | Emerging Markets ETF | Currency, political instability   |
| Europe           | VGK        | Vanguard Europe ETF  | Economic slowdown, energy         |
| Japan            | EWJ        | iShares Japan        | Deflation, currency, aging        |
| South Korea      | EWY        | iShares Korea ETF    | North Korea, tech concentration   |

**Data Source**: yfinance regional ETF data
**Use Case**: Understand global risk concentration and diversification effectiveness
