# Financial AI Agent — n8n Workflow

An AI-powered n8n workflow that queries a **Supabase** financial table, performs statistical analysis, predicts future trends, and generates a natural-language financial report via **GPT-4o**.

---

## How It Works

```
POST /webhook/financial-query
        │
        ▼
 Validate Input
        │
        ▼ (parallel)
┌───────────────────────────────────────┐
│  Supabase Query 1: Daily Aggregates   │
│  Supabase Query 2: Category Breakdown │
│  Supabase Query 3: Monthly MoM Trends │
└───────────────────────────────────────┘
        │
        ▼
 Compute Analysis (JS Code Node)
 - Linear regression (trend prediction)
 - Anomaly detection (±2 std dev)
        │
        ▼
 GPT-4o Financial Agent
 - Answers user query
 - Generates Markdown report
        │
        ▼ (parallel)
┌──────────────────────────────────┐
│  Save report → Supabase          │
│  Return JSON response to caller  │
└──────────────────────────────────┘
```

---

## Setup

### 1. Prerequisites

- n8n instance (self-hosted or cloud)
- Supabase project with PostgreSQL access
- OpenAI API key (GPT-4o access required)

### 2. Supabase Tables

Run the following SQL in your Supabase SQL editor:

```sql
-- Source data table
CREATE TABLE financial_transactions (
  id          SERIAL PRIMARY KEY,
  created_at  TIMESTAMPTZ DEFAULT now(),
  amount      NUMERIC     NOT NULL,
  category    TEXT        NOT NULL,
  description TEXT
);

-- Report storage table
CREATE TABLE financial_reports (
  id               SERIAL PRIMARY KEY,
  report_id        TEXT UNIQUE,
  user_query       TEXT,
  date_range_start DATE,
  date_range_end   DATE,
  report_content   TEXT,
  analysis_summary JSONB,
  generated_at     TIMESTAMPTZ
);
```

### 3. n8n Credentials

Add these two credentials in n8n (Settings → Credentials):

| Name | Type | Details |
|---|---|---|
| `Supabase PostgreSQL` | Postgres | Your Supabase DB connection string (port 5432 or 6543 for pooler) |
| `OpenAI API` | OpenAI | Your OpenAI API key |

### 4. Import the Workflow

1. Open n8n → **Workflows** → **Import from file**
2. Select `financial-ai-agent.json`
3. Map both credentials to the matching nodes
4. Toggle the workflow **Active**

---

## Usage

Send a `POST` request to your webhook URL:

```
POST https://<your-n8n-host>/webhook/financial-query
Content-Type: application/json
```

### Request Body

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `query` | string | **Yes** | — | The financial question to answer |
| `dateRangeStart` | string | No | 90 days ago | Start date `YYYY-MM-DD` |
| `dateRangeEnd` | string | No | Today | End date `YYYY-MM-DD` |
| `tableName` | string | No | `financial_transactions` | Source table name |
| `reportType` | string | No | `full` | Report type hint for the AI |

### Example Request

```bash
curl -X POST https://your-n8n-instance/webhook/financial-query \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What are my top spending categories this quarter?",
    "dateRangeStart": "2026-01-01",
    "dateRangeEnd": "2026-03-31",
    "tableName": "financial_transactions"
  }'
```

### Example Response

```json
{
  "success": true,
  "reportId": "RPT-1748534400000",
  "generatedAt": "2026-05-29T10:00:00.000Z",
  "dateRange": "2026-01-01 to 2026-03-31",
  "userQuery": "What are my top spending categories this quarter?",
  "report": "## Financial Report\n\n### Top Spending Categories...",
  "analysisSummary": {
    "totalTransactions": 142,
    "totalAmount": 18450.75,
    "topCategory": "Housing",
    "trendDirection": "upward",
    "latestMonthGrowth": 4.2,
    "anomalies": [],
    "predictions": {
      "nextMonth": 6320.50,
      "twoMonthsOut": 6580.25
    }
  }
}
```

---

## Test Queries

| Query | Description |
|---|---|
| `"What are my top spending categories?"` | Category ranking |
| `"Give me a month-by-month spending breakdown."` | Monthly summary |
| `"What are my predicted expenses for the next 2 months?"` | Trend prediction |
| `"Are there any unusual spending patterns?"` | Anomaly detection |
| `"Generate a complete financial health report."` | Full analysis |
| `"Compare Q1 2026 vs Q4 2025 spending."` | Quarter comparison |
| `"Where should I cut back to improve financial health?"` | Recommendations |

---

## What the AI Analyses

- **Category breakdown** — total, average, median, and std deviation per category
- **Month-over-month growth** — % change between each month
- **Linear regression** — slope, R² fit score, and 1–2 month predictions
- **Anomaly detection** — flags categories more than 2 standard deviations from the mean
- **Natural-language report** — GPT-4o answers the user's exact question with data-backed insights and recommendations

---

## Files

```
workflows/
├── financial-ai-agent.json   # n8n workflow (import this)
└── README.md                 # This file
```
