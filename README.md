##DEMO URL
https://www.loom.com/share/6479b66f77ce4a5b835dcd691bc2b1df

# loop-task
RAG chatbot that answers from provided CSVs, with strict "file-only" responses

## Project Structure

This project uses **notebooks-only** approach for simplicity:

### Notebooks
- `notebook2/00_explore_data.ipynb` - Data exploration and schema discovery
- `notebook2/01_data_ingestion.ipynb` - Data cleaning and loading into PostgreSQL
- `notebook2/02_test_chatbot.ipynb` - Test the chatbot interactively (includes test cases and evaluation)
- `notebook2/03_reset_database.ipynb` - Reset database (drops tables for fresh start)

### Core Code
All chatbot code is embedded directly in the notebooks. No separate source files are required.

### Data Files
- `data/holdings.csv` - Holdings data
- `data/trades.csv` - Trades data
- `schema_info.json` - Schema information
- `column_mappings.json` - Column name mappings

## Architecture

The following diagram illustrates the complete system architecture and data flow:

```mermaid
graph LR
    subgraph "Data Sources"
        CSV1[holdings.csv<br/>1022 rows] 
        CSV2[trades.csv<br/>649 rows]
    end
    
    subgraph "Data Ingestion"
        Clean[Data Cleaning<br/>& Column Mapping] 
        Ingest[Database Ingestion<br/>SQLAlchemy]
    end
    
    subgraph "Storage"
        DB[(PostgreSQL<br/>fund_holdings<br/>fund_trades)]
    end
    
    subgraph "Chatbot Processing - LangGraph Workflow"
        Q[User Question] --> IC[Intent Classifier<br/>classify_query_intent]
        IC -->|OUT_OF_SCOPE| AC[Answer Composer<br/>Return Refusal]
        IC -->|Allowed Intent| SG[SQL Generator<br/>LLM + Pydantic]
        SG -->|SQL Plan| SA[Safety Gate<br/>validate_sql_safety]
        SA -->|Safe SQL| EX[DB Executor<br/>Execute Query]
        SA -->|Unsafe SQL| AC
        EX -->|Query Results| AC
        EX -->|No Results/Zero| AC
        AC -->|Final Answer| User[👤 User]
    end
    
    subgraph "Helper Functions"
        EFN[extract_fund_name<br/>Find fund names]
        CQI[classify_query_intent<br/>Determine intent]
        GTS[get_table_schema<br/>Format schema]
        VSS[validate_sql_safety<br/>Check SQL safety]
        GSP[generate_sql_plan<br/>LLM wrapper]
    end
    
    subgraph "External Services"
        LLM[OpenAI API<br/>GPT-4o-mini]
    end
    
    CSV1 --> Clean
    CSV2 --> Clean
    Clean --> Ingest
    Ingest --> DB
    
    EX -->|Query| DB
    DB -->|Results| EX
    
    SG -.->|API Call| LLM
    LLM -.->|Structured SQL| SG
    
    IC -.->|Uses| EFN
    IC -.->|Uses| CQI
    SG -.->|Uses| GTS
    SG -.->|Uses| GSP
    SA -.->|Uses| VSS
    
    AC -->|Final Answer| User
```

### Architecture Overview

**Data Flow:**
1. **Data Sources**: CSV files (`holdings.csv`, `trades.csv`) contain raw fund data
2. **Data Ingestion**: Data is cleaned, column names are mapped (e.g., `PL_YTD` → `plytd`), and loaded into PostgreSQL
3. **Storage**: PostgreSQL database stores normalized data in `fund_holdings` and `fund_trades` tables
4. **Chatbot Processing**: LangGraph workflow processes user questions through multiple nodes:
   - **Intent Classifier**: Determines if query is valid and classifies intent type
   - **SQL Generator**: Uses LLM to generate SQL queries (with Pydantic structured output)
   - **Safety Gate**: Validates SQL to prevent dangerous operations
   - **DB Executor**: Executes SQL and validates results
   - **Answer Composer**: Formats results into natural language answers

**Helper Functions:**
- `extract_fund_name()`: Extracts fund names from user questions
- `classify_query_intent()`: Classifies query intent using deterministic rules
- `get_table_schema()`: Formats database schema for LLM prompts
- `validate_sql_safety()`: Validates SQL for security
- `generate_sql_plan()`: Wraps LLM calls to ensure consistent response format

**External Services:**
- OpenAI API (GPT-4o-mini) for SQL generation with structured output

## How to Start with this Project

### Prerequisites

Before starting, ensure you have:
- **Python 3.12+** installed
- **PostgreSQL** database (or Docker for containerized setup)
- **OpenAI API key** (for SQL generation)
- **Jupyter Notebook** or **JupyterLab** installed
- **uv** package manager (or use `pip` as alternative)

### Step 1: Clone and Navigate to Project

```bash
cd loop-task
```

### Step 2: Install Dependencies

Using `uv` (recommended):
```bash
uv sync
```

Or using `pip`:
```bash
pip install -r requirements.txt
# Or install manually: pip install langgraph langchain-openai pandas sqlalchemy psycopg sqlparse python-dotenv
```

### Step 3: Set Up Environment Variables

Create a `.env` file in the parent directory (`/Users/suren/interviews-tasks/.env`) with:

```env
# Database Configuration
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/funddb

# OpenAI Configuration
OPENAI_API_KEY=your_openai_api_key_here
OPENAI_MODEL=gpt-4o-mini
```

**Note:** Replace `your_openai_api_key_here` with your actual OpenAI API key.

### Step 4: Start PostgreSQL Database

**Option A: Using Docker (Recommended)**
```bash
cd ..  # Go to parent directory
make up  # Starts PostgreSQL container
```

**Option B: Using Local PostgreSQL**
- Ensure PostgreSQL is running on your machine
- Create database: `createdb funddb`
- Update `DATABASE_URL` in `.env` if using different credentials

### Step 5: Run Notebooks in Order

Open Jupyter Notebook or JupyterLab:
```bash
jupyter notebook
# or
jupyter lab
```

Then run notebooks in this exact order:

1. **`00_explore_data.ipynb`** - Data Exploration
   - Explores CSV files structure
   - Identifies key columns and data quality issues
   - Generates `schema_info.json`
   - **Time:** ~2-3 minutes

2. **`01_data_ingestion.ipynb`** - Data Ingestion
   - Loads and cleans CSV data
   - Normalizes column names
   - Loads data into PostgreSQL
   - Creates database indexes
   - **Time:** ~1-2 minutes

### Verify Installation

After running the first two notebooks, verify:
- ✅ Database tables created: `fund_holdings` and `fund_trades`
- ✅ Schema files generated: `schema_info.json` and `column_mappings.json`
- ✅ Data loaded successfully (check notebook outputs)


3. **`02_test_chatbot.ipynb`** - Test Chatbot
   - Initializes chatbot workflow
   - Test questions interactively
   - View generated SQL queries
   - Verify refusal behavior
   - Includes  must-pass and  must-fail test cases
   - Runs comprehensive evaluation with performance metrics
   - **Time:** ~5-10 minutes (includes test execution)


### Troubleshooting

**Database Connection Issues:**
- Verify PostgreSQL is running: `docker ps` (if using Docker)
- Check `DATABASE_URL` in `.env` file
- Ensure database `funddb` exists

**Missing Dependencies:**
- Run `uv sync` again or `pip install -r requirements.txt`
- Restart Jupyter kernel after installing packages

**OpenAI API Errors:**
- Verify `OPENAI_API_KEY` is set correctly in `.env`
- Check API key has sufficient credits
- Ensure network connectivity

**Notebook Errors:**
- Make sure notebooks are run in order
- Check that previous notebook completed successfully
- Restart kernel if variables are undefined

### Optional: Reset Database

If you need to start fresh and delete all existing data:

1. **Run `notebook2/03_reset_database.ipynb`** - This will:
   - Drop `fund_holdings` and `fund_trades` tables
   - Remove all indexes and constraints
   - Verify tables are deleted
   - Prepare database for fresh ingestion

2. **Then re-run `notebook2/01_data_ingestion.ipynb`** - This will:
   - Recreate the tables with fresh data
   - Rebuild indexes
   - Verify data is loaded correctly

**⚠️ WARNING**: The reset notebook will permanently delete all data. Use with caution!

### Next Steps

- Read `REQUIREMENTS.md` for detailed requirements
- Check `REQUIREMENTS_VALIDATION.md` for validation results
- Explore notebooks to understand the implementation
- Test your own questions in `notebook2/02_test_chatbot.ipynb`

## Features

- ✅ SQL-first approach (all answers from database)
- ✅ Router filters out-of-scope questions
- ✅ Exact refusal message: "Sorry can not find the answer"
- ✅ Prompt templates embedded in notebooks
- ✅ Supports both example questions from requirements

## Intent Classification Logic

The chatbot uses a **strict, rule-first intent classification system** to ensure reliable, data-only responses. This section documents the exact logic used.

### Step 0 — Closed-Domain Rule

**If the question cannot be answered using columns from `holdings.csv` or `trades.csv`, return `CANNOT_ANSWER` → final response = "Sorry can not find the answer".**

This is enforced even if the LLM "thinks" it knows the answer. The system only answers questions that can be answered from the available data columns.

### Step 1 — Normalize Input

1. **Lowercase the user question** (for keyword matching)
2. **Remove extra punctuation** (for cleaner pattern matching)
3. **Keep the raw question** for entity extraction (fund name extraction needs original case)

### Step 2 — Extract Fund Name (Simple + Safe)

Try to detect a portfolio/fund name by:

1. **Exact match** against unique `PortfolioName` values from both files (case-insensitive)
2. **If not found, try partial match** using `ILIKE %text%` pattern
3. **If the intent requires a fund and no fund is found → `CANNOT_ANSWER`**

Fund name extraction uses regex patterns to find fund names in various question formats:
- "for/of/in/by FundName" → extracts name after preposition
- "FundName fund/portfolio/account" → extracts name before keywords
- "does/has FundName have" → extracts name between question words
- Falls back to known fund names from `schema_info.json` if patterns fail

### Step 3 — Rule-First Routing (Deterministic)

We classify using keyword signals in this order:

#### A) Count Questions

**If question contains any count cues:**
- **count cues**: `how many`, `count`, `number of`, `total number of`

**Then:**
- **If it contains trades cues → intent = `TRADES_COUNT`**
  - **trades cues**: `trade`, `trades`, `transaction`, `buy`, `sell`, `counterparty`
- **Else → intent = `HOLDINGS_COUNT`**
  - **holdings cues**: `holding`, `holdings`, `position`, `positions`

**Validation:**
- Must have `fund_a` present
- Fund must exist in the correct table (holdings for `HOLDINGS_COUNT`, trades for `TRADES_COUNT`)
- If fund doesn't exist in correct table → `CANNOT_ANSWER`

#### B) Performance / P&L Questions

**If question contains any P&L/performance cues:**
- **cues**: `p&l`, `profit`, `loss`, `performance`, `performed better`, `best fund`, `rank`

**Then:**
- **If it mentions two funds (fund_a + fund_b found) → `FUND_PL_COMPARE`**
- **Else → `FUND_PL_RANK`**

**Metric mapping for the interview task:**
- **"yearly P&L" → use `PL_YTD`** (Year-to-date profit/loss)
- **If user asks for a specific year** (e.g., 2021/2022) or a time-series trend → `CANNOT_ANSWER` (because the data doesn't contain yearly history)

**Validation:**
- For `FUND_PL_RANK`: Fund (if specified) must exist in holdings table (P&L only in holdings)
- For `FUND_PL_COMPARE`: Both funds must exist in holdings table
- Default metric is `plytd` (yearly) unless user specifies monthly/quarterly/daily

#### C) Everything Else

**If none of the above match confidently → `CANNOT_ANSWER`**

The system uses a 2-layer classifier:
- **Layer A**: Fast deterministic rule-based classification (handles most cases)
- **Layer B**: LLM classifier for ambiguous cases (with strict Pydantic output)

### Step 4 — SQL Mapping Per Intent (Only Safe SELECT)

Each intent maps to a specific SQL query pattern:

#### HOLDINGS_COUNT
```sql
SELECT COUNT(*) FROM fund_holdings WHERE portfolioname ILIKE '%fund_a%'
```

#### TRADES_COUNT
```sql
SELECT COUNT(*) FROM fund_trades WHERE portfolioname ILIKE '%fund_a%'
```

#### FUND_PL_RANK
```sql
SELECT portfolioname, SUM(plytd) AS total_pl 
FROM fund_holdings 
WHERE plytd IS NOT NULL 
GROUP BY portfolioname 
ORDER BY total_pl DESC 
LIMIT 5
```

#### FUND_PL_COMPARE
```sql
SELECT portfolioname, SUM(plytd) AS total_pl 
FROM fund_holdings 
WHERE portfolioname ILIKE '%fund_a%' OR portfolioname ILIKE '%fund_b%'
GROUP BY portfolioname
```

**Safety Rules:**
- Only `SELECT` queries allowed (read-only)
- Dangerous keywords blocked: `DROP`, `DELETE`, `UPDATE`, `INSERT`, `ALTER`, `CREATE`, `TRUNCATE`
- Uses exact database column names (lowercase, no underscores for P&L: `plytd`, not `pl_ytd`)

### Step 5 — Final "Answerability Gate"

After SQL execution:

1. **If query returns 0 rows OR missing required columns → return "Sorry can not find the answer"**
2. **Otherwise compose answer strictly from query result**

This gate prevents false positives like:
- Asking trades count for a fund that only exists in holdings (or vice-versa)
- Asking for P&L for a fund that doesn't exist in holdings
- Any query that returns empty results

**Special handling:**
- For COUNT queries: `0` is a valid answer (fund exists but has 0 items)
- For non-COUNT queries: `0` or `NULL` → treated as not found

### Interview Note (Minimal by Design)

**Note (Interview Scope):**

For this interview task, the intent set is intentionally minimal and aligned to the required question types (holdings count, trades count, yearly P&L ranking/compare). In a full product version, this layer would expand by adding new intents only when new supported question patterns and/or new columns are introduced. Until then, out-of-scope requests are routed to `CANNOT_ANSWER` to guarantee data-only responses and avoid hallucination.

### Allowed Intents (Strict Contract)

The system only supports these 4 intent types:

1. **`HOLDINGS_COUNT`** → "How many holdings does Fund X have?"
2. **`TRADES_COUNT`** → "How many trades for Fund X?"
3. **`FUND_PL_RANK`** → "Which fund performed best?" (rank funds by PL_YTD)
4. **`FUND_PL_COMPARE`** → "Compare Fund A vs Fund B by PL_YTD"
5. **`CANNOT_ANSWER`** → anything outside supported columns / unclear / missing fund

All other questions return `CANNOT_ANSWER` with the refusal message: "Sorry can not find the answer".



