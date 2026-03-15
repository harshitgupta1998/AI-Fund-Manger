## What is the Financial Analyst Agent?

The Financial Analyst is an intelligent AI agent designed to assess your personal financial situation and provide tailored investment recommendations. It analyzes your financial profile—including your age, investment experience, available capital, and investment goals—to generate a customized risk profile and recommend suitable investment sectors for your portfolio.

This agent serves as the **first stage** in the AI Fund Manager system, establishing the foundation for personalized wealth management by understanding your financial circumstances and risk tolerance.

### Core Features

- **Financial Analysis**: Comprehensive risk profile assessment based on age, investment experience, available capital, and investment timeline
- **Return Calculation**: Precise target return calculations using advanced calculation tools
- **Sector Recommendations**: Intelligent investment sector recommendations tailored to your risk profile and stated preferences
- **Risk Profile Generation**: Detailed assessment of your investment risk tolerance (Conservative, Moderate, Aggressive)

## How It Works

The Financial Analyst agent operates through the following intelligent process:

### Agent Workflow

1. **Information Collection**: The agent starts by collecting essential financial information from you:
   - Investment amount (in hundreds of millions)
   - Target return amount after 1 year
   - Age and investment experience level
   - Investment purpose and timeline
   - Interest areas across 10 investment sectors

2. **Financial Analysis**: The agent analyzes your profile using:
   - **Age-based risk assessment**: Younger investors typically have higher risk tolerance
   - **Experience evaluation**: Investment experience level affects risk capacity
   - **Capital analysis**: Available funds determine investment flexibility
   - **Return calculation**: Precise calculations of required returns to meet your targets

3. **Risk Profile Determination**: Based on the analysis, the agent determines your risk profile:
   - **Conservative**: Lower risk tolerance, focus on capital preservation
   - **Moderate**: Balanced approach between growth and stability
   - **Aggressive**: Higher risk tolerance, focus on capital growth

4. **Sector Recommendations**: The agent recommends investment sectors that align with:
   - Your calculated risk profile
   - Your stated areas of interest
   - Current market conditions and historical performance
   - Diversification principles

### Processing Architecture

```
User Input
    ↓
[AgentCore Runtime]
    ↓
Financial Analyst Agent
    ├─ Analyze financial profile
    ├─ Calculate risk assessment
    ├─ Compute target returns
    └─ Select recommended sectors
    ↓
Sector Recommendations & Risk Profile
```

## Technology Stack

- **AI Framework**: AWS Bedrock AgentCore Runtime
- **LLM Model**: OpenAI GPT-OSS 120B
- **Infrastructure**: AWS (serverless, auto-scaling)
- **Tools**: Built-in Calculator for precise return calculations
- **UI Framework**: Streamlit
- **Language**: Python

## Setup Instructions

### Prerequisites

- AWS Account with Bedrock access
- Python 3.8 or higher
- AWS CLI configured with credentials

### Step 1: Install Dependencies

```bash
# From root directory
cd ..
pip install -r requirements.txt

# Configure AWS credentials
aws configure

# Navigate to financial_analyst folder
cd financial_analyst
```

### Step 2: Deploy AgentCore Runtime

The agent must be deployed to AWS Bedrock before use:

```bash
# Deploy the Financial Analyst agent
python deploy.py

# Verify deployment (creates deployment_info.json)
cat deployment_info.json
```

**What this does:**

- Deploys the Financial Analyst agent to AWS Bedrock
- Configures the LLM model and tools
- Generates deployment metadata for the Streamlit app
- Sets up the agent endpoint for API calls

### Step 3: Run the Streamlit Web App

```bash
# Start the web interface
streamlit run app.py

# Access the app at http://localhost:8501
```

### Step 4: Use the Application

1. Open your browser to `http://localhost:8501`
2. Input your financial information using the form
3. Click "Analyze" to run the agent
4. View your personalized risk profile and recommendations

### Sample Output Breakdown

```
┌─────────────────────────────────────────────┐
│     YOUR FINANCIAL ANALYSIS RESULTS         │
├─────────────────────────────────────────────┤
│                                             │
│ Risk Profile: MODERATE                      │
│ • Age: 35-45 years                          │
│ • Experience: 5-10 years                    │
│ • Required Return: 18.2% annually           │
│                                             │
│ Investment Amount: $50M → $60M Target       │
│ Return on Investment: $10M (18.2%)          │
│ Feasibility: ACHIEVABLE with proper mix     │
│                                             │
│ Recommended Sectors:                        │
│ [Tech Stocks] [Healthcare] [Dividend]       │
│ [Infrastructure] [International]            │
│                                             │
│ Analysis Notes:                             │
│ Your moderate risk profile aligns well      │
│ with your experience level. Diversify       │
│ across growth and stability sectors...      │
│                                             │
└─────────────────────────────────────────────┘
```
