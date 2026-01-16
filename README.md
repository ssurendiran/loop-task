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

**Enforcement mechanisms:**
- **Unsupported columns detection** (Step 3.C): Rejects queries asking for unavailable columns (sector, country, NAV, AUM, etc.)
- **Time-series rejection** (Step 3.D): Rejects queries requiring historical data (only snapshot data available)
- **P&L per trade rejection** (Step 3.E): Rejects P&L queries on trades table (P&L only in holdings)
- **Answerability gate** (Step 5): Validates fund existence in correct tables before and after SQL execution
- **Layer B guardrails**: LLM classifier has strict guardrails to reject out-of-scope queries

### Step 1 — Normalize Input

1. **Lowercase the user question** (for keyword matching)
2. **Keep the raw question** for entity extraction (fund name extraction needs original case)

### Step 2 — Extract Fund Name (Simple + Safe)

Try to detect a portfolio/fund name by:

1. **Regex pattern matching** to find fund names in various question formats:
   - "for/of/in/by FundName" → extracts name after preposition
   - "FundName fund/portfolio/account" → extracts name before keywords
   - "does/has FundName have" → extracts name between question words
2. **Fallback to known fund names** from `schema_info.json` if patterns fail (case-insensitive matching)
3. **If the intent requires a fund and no fund is found → `CANNOT_ANSWER`**

**Note:** The extracted fund name is validated later using `ILIKE %text%` pattern matching against the database tables in the answerability gate (Step 5).

### Step 3 — Rule-First Routing (Deterministic)

Layer A (`classify_query_intent_layer_a`) processes questions in this exact order:

#### Pre-Classification: False-Positive Trap Detection

**Before any intent classification, detect and reject ambiguous queries:**

1. **Mixed Holdings + Trades Wording:**
   - **Trap**: "How many holdings were bought for Garfield?"
   - **Detection**: If question has holdings signal AND trade action words (`bought`, `sold`, `purchased`, `executed`, `trade`, `traded`, `transaction`)
   - **Result**: `CANNOT_ANSWER` → "Holdings table does not track buy/sell transactions"

2. **Ambiguous Trades vs Holdings:**
   - **Trap**: "How many trades are currently in the portfolio?"
   - **Detection**: If question has trades signal AND (`current` OR (`portfolio` AND NOT `holdings`)) AND count signal
   - **Result**: `CANNOT_ANSWER` → "Ambiguous: trades table tracks transactions, not current portfolio holdings"

#### A) Count Questions

**If question contains any count cues:**
- **count cues**: `how many`, `count`, `number of`, `total holdings`, `total trades`

**Signal detection:**
- **Holdings signals**: `holding`, `holdings`, `positions`, `portfolio holdings`
- **Trades signals**: `trade`, `trades`, `transactions`, `buy`, `sell`, `counterparty`

**Special case: "or" queries (holdings or trades):**
- **Detection**: If question contains `" or "` AND has both holdings signal AND trades signal
- **Example**: "Total number of holdings or trades for Garfield?"
- **Behavior**: Returns `HOLDINGS_COUNT` with `fund_a` and `reason="OR_QUERY"`
  - Tries holdings table first
  - If holdings returns 0, automatically tries trades table
  - Returns result from whichever table has data
  - If both return 0 → `CANNOT_ANSWER` → "Fund not found in holdings or trades tables"
- **Note**: OR_QUERY intents skip table existence check in answerability gate (validation happens during execution)

**Then classify:**
1. **"Or" queries (handled first):**
   - If `has_or_query` is True:
     - If fund name present → `HOLDINGS_COUNT` with `fund_a` and `reason="OR_QUERY"`
     - If no fund name → `CANNOT_ANSWER` → "Fund name required for holdings or trades query"

2. **If has holdings signal AND NOT trades signal:**
   - If fund name present → `HOLDINGS_COUNT` with `fund_a`
   - If no fund name → `CANNOT_ANSWER` → "Holdings count requires a fund name"

3. **If has trades signal AND NOT holdings signal:**
   - If fund name present → `TRADES_COUNT` with `fund_a`
   - If no fund name → `CANNOT_ANSWER` → "Trades count requires a fund name"

4. **Ambiguous count (has count but unclear holdings vs trades):**
   - If fund name present AND NOT trades signal → default to `HOLDINGS_COUNT` (more common)
   - If no fund name → continue to other checks

**Validation (in answerability gate):**
- Must have `fund_a` present
- **Exception for OR_QUERY**: Skips table existence check (fund may be in either table)
- For non-OR_QUERY intents: Fund must exist in the correct table (holdings for `HOLDINGS_COUNT`, trades for `TRADES_COUNT`)
- If fund doesn't exist in correct table → `CANNOT_ANSWER`

#### B) Performance / P&L Questions

**If question contains any P&L/performance cues:**
- **P&L cues**: `p&l`, `profit`, `loss`, `performance`, `performed better`, `performed best`, `performed worst`, `best fund`, `worst fund`
- **Period cues**: `yearly`, `ytd`, `year-to-date`, `pl_ytd`, `plytd`, `monthly`, `pl_mtd`, `plmtd`, `quarterly`, `pl_qtd`, `plqtd`, `daily`, `pl_dtd`, `pldtd`
- **Ranking cues**: `rank`, `top`, `bottom`, `highest`, `lowest`, `compare`, `best`, `worst`, `better`

**P&L Metric Detection (defaults to yearly):**
- Default: `plytd` (yearly P&L)
- If `pl_mtd`/`plmtd`/`monthly` → `plmtd` (monthly)
- If `pl_qtd`/`plqtd`/`quarterly` → `plqtd` (quarterly)
- If `pl_dtd`/`pldtd`/`daily` → `pldtd` (daily)

**Then classify:**
- **If has P&L signal OR period signal OR (ranking signal AND "fund" in question):**
  1. **Comparison detection:**
     - If contains `compare`, `vs`, or `versus`:
       - Extract fund names using regex: `(\w+(?:\s+\w+)*)\s+(?:or|vs|versus)\s+(\w+(?:\s+\w+)*)`
       - If 2+ funds found → `FUND_PL_COMPARE` with `fund_a` and `fund_b`
       - If only 1 fund → `CANNOT_ANSWER` → "Comparison requires two fund names"
       - If no funds → `CANNOT_ANSWER` → "Comparison requires fund names"
  
  2. **Single fund or ranking:**
     - If fund name present → `FUND_PL_RANK` with `fund_a`
     - If no fund name BUT has ranking signal → `FUND_PL_RANK` (ranking all funds)

**Validation (in answerability gate):**
- For `FUND_PL_RANK`: Fund (if specified) must exist in holdings table (P&L only in holdings)
- For `FUND_PL_COMPARE`: Both funds must exist in holdings table
- Default metric is `plytd` (yearly) unless user specifies monthly/quarterly/daily

#### C) Unsupported Columns Detection

**Reject queries asking for unavailable columns:**
- **Unsupported columns**: `sector`, `country`, `nav`, `aum`, `assets under management`, `portfolio manager`, `manager`, `pm`, `attribution`, `realized`, `unrealized`, `cancelled`, `status`
- **Result**: `CANNOT_ANSWER` → "Requested column/metric not available in holdings or trades tables"

**Examples:**
- "What is the sector breakdown of Garfield?" → `CANNOT_ANSWER`
- "Show country exposure for Ytum." → `CANNOT_ANSWER`
- "What is the NAV/AUM of Garfield?" → `CANNOT_ANSWER`

#### D) Time-Series Rejection

**Reject time-series queries (data is snapshot, not historical):**
- **Time patterns**: `year over year`, `yearly breakdown`, `annual trend`, `by year`, `yearly pl breakdown`, `in 2022`, `in 2023`, `in 2024`, `last year`, `monthly trend`, `monthly performance`, `quarterly trend`, `q1`, `q2`, `q3`, `q4`
- **Result**: `CANNOT_ANSWER` → "Time-series analysis not supported (only snapshot data available)"

**Exception: Yearly P&L context:**
- If question contains `"yearly"` AND has P&L signals/periods/ranking signals → **NOT** rejected as time-series
- **Logic**: "yearly Profit and Loss" means PL_YTD (year-to-date) metric, not time-series analysis
- **Example**: "Which funds performed better by yearly Profit and Loss?" → Valid P&L ranking query (not time-series)

**Special case: Trade date queries:**
- If question has `trades` AND (`on` OR `date` OR `when`) AND contains digits → `CANNOT_ANSWER` → "Time-series queries on trade dates not supported"

**Examples:**
- "Which fund performed best in 2022?" → `CANNOT_ANSWER`
- "Show monthly performance trend for Garfield." → `CANNOT_ANSWER`
- "Trades done on 01/08/23 for HoldCo 1?" → `CANNOT_ANSWER`
- "Which funds performed better by yearly Profit and Loss?" → ✅ Valid (P&L ranking, not time-series)

#### E) P&L Per Trade / Attribution Detection

**Reject P&L queries on trades table (P&L only in holdings):**
- **Detection**: If question has `trade` AND (`pnl` OR `p&l` OR `profit` OR `loss`) AND (`per trade` OR `contributed` OR `attribution`)
- **Result**: `CANNOT_ANSWER` → "Trades table does not contain P&L fields (P&L only available in holdings table)"

**Examples:**
- "What is PnL per trade for HoldCo 1?" → `CANNOT_ANSWER`
- "Which trades contributed most to yearly P&L?" → `CANNOT_ANSWER`
- "Realized vs unrealized P&L split?" → `CANNOT_ANSWER`

#### F) Missing Fund Name Validation

**Reject queries missing required fund names:**

1. **Count queries without fund name:**
   - If has count signal AND no fund name AND NOT (`all` OR `every`) → `CANNOT_ANSWER` → "Count query requires a specific fund name"

2. **P&L queries without fund name:**
   - If (has P&L signal OR has P&L period signal OR ranking signal) AND (no fund name AND no ranking signal) → `CANNOT_ANSWER` → "P&L query requires fund name(s) or explicit ranking request"
   - **Logic**: If question has P&L/ranking/period signals but lacks both a fund name AND a ranking signal, reject it

**Examples:**
- "How many holdings are there?" → `CANNOT_ANSWER` (unless "all" is mentioned)
- "Which fund performed better?" → `CANNOT_ANSWER` (unless ranking signal present)

#### G) Fallback to Layer B

**If none of the above rules match confidently → return `None` → send to Layer B (LLM classifier)**

### Layer B: LLM Classifier (`classify_query_intent_layer_b`)

**Used when Layer A returns `None` (ambiguous cases):**

1. **LLM Availability Check:**
   - If `llm_intent` not available → `CANNOT_ANSWER` → "LLM not available for intent classification"

2. **LLM Prompt:**
   - System prompt defines strict allowed intents (same 4 intents + `CANNOT_ANSWER`)
   - Guardrails:
     - If question asks for unavailable columns → `CANNOT_ANSWER`
     - If fund name missing where required → `CANNOT_ANSWER`
     - Default P&L metric is `plytd` (yearly) unless specified
     - Yearly P&L → interpret as PL_YTD (year-to-date)
     - Specific year requests → `CANNOT_ANSWER`

3. **Response Handling:**
   - If LLM returns `None` → `CANNOT_ANSWER` → "LLM returned None"
   - If `QueryIntent` object → return as-is
   - If dict → validate with `QueryIntent.model_validate()`
   - If unexpected format → `CANNOT_ANSWER` → "Unexpected LLM response format"

4. **Error Handling:**
   - Any exception → `CANNOT_ANSWER` → "LLM classification error: {error}"

### Overall Classification Flow (`classify_query_intent`)

**2-Layer Intent Classifier with Answerability Gate:**

1. **Try Layer A first** (fast, deterministic rule-based)
   - If returns `QueryIntent` → proceed to step 3
   - If returns `None` → proceed to step 2

2. **Use Layer B** (LLM classifier for ambiguous cases)
   - Always returns `QueryIntent` (never `None`)

3. **Apply Answerability Gate** (`answerability_gate` function)
   - Validates fund existence in correct tables
   - Validates P&L queries use holdings table
   - Returns validated `QueryIntent`

4. **Return final `QueryIntent`**

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

#### HOLDINGS_COUNT (OR_QUERY variant)
**Special handling for "or" queries:**
- First tries: `SELECT COUNT(*) FROM fund_holdings WHERE portfolioname ILIKE '%fund_a%'`
- If holdings returns 0, automatically tries: `SELECT COUNT(*) FROM fund_trades WHERE portfolioname ILIKE '%fund_a%'`
- Returns result from whichever table has data (> 0)
- If both return 0 → `CANNOT_ANSWER` → "Fund not found in holdings or trades tables"

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

### Step 5 — Answerability Gates (Two-Stage Validation)

The system uses **two answerability gates** to prevent false positives:

#### Gate 1: Pre-SQL Validation (Before SQL Generation)

**Runs in `answerability_gate()` function after intent classification:**

1. **For COUNT intents**: Validates that `fund_a` exists in the correct table (holdings for `HOLDINGS_COUNT`, trades for `TRADES_COUNT`)
   - **Exception for OR_QUERY**: Skips table existence check (fund may be in either table; validation happens during execution)
2. **For P&L queries**: Validates that fund(s) exist in holdings table (P&L data only available in holdings)
3. **For comparisons**: Validates that both `fund_a` and `fund_b` exist in holdings table
4. **If validation fails → `CANNOT_ANSWER`** (prevents unnecessary SQL generation)

This gate prevents false positives like:
- Asking trades count for a fund that only exists in holdings (or vice-versa)
- Asking for P&L for a fund that doesn't exist in holdings

#### Gate 2: Post-SQL Validation (After SQL Execution)

**Runs in two places:**

1. **`db_executor_node()` (immediately after query execution):**
   - If query returns 0 rows → check for OR_QUERY intent
     - **OR_QUERY handling**: If `intent.reason == "OR_QUERY"` and holdings query returned 0, automatically try trades query
     - If trades query returns data (> 0) → use trades result
     - If both holdings and trades return 0 → set `query_error` → "Fund not found in holdings or trades tables"
   - For COUNT queries: Allows `0` values to pass through (only rejects `NULL`)
   - For non-COUNT queries: Rejects both `0` and `NULL` values

2. **`answer_composer_node()` (final answer composition):**
   - If `query_error` present → return "Sorry can not find the answer"
   - If `result_rows == 0` or empty `query_result` → return "Sorry can not find the answer"
   - If single value result: If value is `0` or `NULL` → return "Sorry can not find the answer"
   - If multiple values: If all values are `0` or `NULL` → return "Sorry can not find the answer"
   - Otherwise compose answer strictly from query result

**Note:** Even though `db_executor_node` allows COUNT queries with `0` to pass through, `answer_composer_node` rejects all `0` values (including COUNT), so the final behavior is that `0` is treated as "not found" for all query types.

### Interview Note (Minimal by Design)

** ⚠️ Note (Interview Scope):**

For this interview task, the intent set is intentionally minimal and aligned to the required question types (holdings count, trades count, yearly P&L ranking/compare). In a full product version, this layer would expand by adding new intents only when new supported question patterns and/or new columns are introduced. Until then, out-of-scope requests are routed to `CANNOT_ANSWER` to guarantee data-only responses and avoid hallucination.

### Allowed Intents (Strict Contract)

The system only supports these 4 intent types:

1. **`HOLDINGS_COUNT`** → "How many holdings does Fund X have?"
2. **`TRADES_COUNT`** → "How many trades for Fund X?"
3. **`FUND_PL_RANK`** → "Which fund performed best?" (rank funds by PL_YTD)
4. **`FUND_PL_COMPARE`** → "Compare Fund A vs Fund B by PL_YTD"
5. **`CANNOT_ANSWER`** → anything outside supported columns / unclear / missing fund

All other questions return `CANNOT_ANSWER` with the refusal message: "Sorry can not find the answer".



