##DEMO URL
https://www.loom.com/share/6479b66f77ce4a5b835dcd691bc2b1df

# loop-task
RAG chatbot that answers fund holdings/trades & yearly PnL from provided CSVs, with strict "file-only" responses

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



