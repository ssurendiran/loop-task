# Requirements Specification

## Original Requirement

> Create a chatbot trained on the two files below. The bot should be able to answer questions related to the data from the provided files. If the answer is not found in the given files then it should respond with "Sorry can not find the answer" instead of getting an answer from the internet.
>
> **Example Questions:**
> - Total number of holdings or trades for a given fund
> - Which funds performed better depending on the yearly Profit and Loss of that fund.

---

## Project Scope and Implementation Notes

Due to **limited time constraints**, this project was delivered as a **notebooks-only implementation** focusing on core functionality. All code is implemented in Jupyter notebooks for rapid development and demonstration.

**Core Features Implemented:**
- ✅ Complete chatbot workflow (Router → SQL Generator → Safety Gate → Execute → Validate → Answer)
- ✅ SQL generation using OpenAI GPT-4o-mini
- ✅ Database integration with PostgreSQL
- ✅ Safety validation and refusal handling
- ✅ Interactive testing and evaluation framework

**Note**: The technology choices documented below were selected considering **future easy scaling** and represent the architectural vision for production deployment.

---

## Technology Choices for Future Scaling

The technologies chosen for this project were selected with **future scalability and easy expansion** in mind. Each technology provides a clear path for growth without requiring architectural changes.

### PostgreSQL (Relational Database)

**Why PostgreSQL:**
- **Structured Data**: The CSV data is structured/tabular, making SQL the natural choice
- **Scalability**: Handles millions of rows efficiently with proper indexing
- **Future Scaling**: Easy to add read replicas, partitioning, and materialized views
- **Industry Standard**: Proven at scale, large talent pool, extensive tooling

**Scaling Path:**
- Current: Single instance handles ~1,000 holdings, ~650 trades
- Future: Add read replicas for query distribution, partition by date, add caching layer

### LangGraph (Workflow Orchestration)

**Why LangGraph:**
- **Modular Architecture**: Easy to add new agents/nodes without disrupting existing workflow
- **Agent Scalability**: Simple to add specialized agents (data analysis, reporting, validation)
- **State Management**: Shared state allows agents to collaborate
- **Future Expansion**: Add new nodes incrementally (caching, logging, analytics)

**Scaling Path:**
- Current: Single SQL generation agent
- Future: Add specialized agents, parallel execution, distributed processing

### OpenAI GPT-4o-mini (SQL Generation)

**Why GPT-4o-mini:**
- **Natural Language to SQL**: Converts questions to SQL without hardcoded templates
- **Cost-Effective**: Lower cost than larger models while maintaining good performance
- **Flexibility**: Handles various question patterns without code changes
- **Future Options**: Easy to switch to different models or add model routing

**Scaling Path:**
- Current: Single model for all questions
- Future: Add model routing, caching, fine-tuning, or switch to more powerful models

### Docker (Portability)

**Why Docker:**
- **Run Anywhere**: Works on any platform (local, cloud, on-premise)
- **Consistency**: Same environment across development, staging, production
- **Easy Scaling**: Spin up multiple containers for load distribution
- **Future Deployment**: Easy migration between platforms, CI/CD integration

**Scaling Path:**
- Current: Single container for database
- Future: Containerize application, add orchestration (Kubernetes), auto-scaling

### Why SQL Instead of Vector Databases?

**Vector databases are over-engineering for this use case** because:

- **Structured Data**: The fund holdings and trades data is structured/tabular (not unstructured text)
- **Exact Queries**: Questions require exact values and aggregations (COUNT, SUM, AVG), not semantic similarity
- **SQL is Optimal**: SQL provides precise, efficient querying for structured data
- **No Benefit**: Vector DBs are designed for document similarity search, which is not needed here

Vector databases would add unnecessary complexity (embeddings, similarity search) without providing any benefit for structured financial data queries.

**Note**: If unstructured data is needed in the future, a router can be added to route questions to a vector database while keeping SQL for structured data queries.

---

## Why This Architecture is Future-Proof

1. **Standard Technologies**: PostgreSQL and SQL are industry standards with proven scalability
2. **Separation of Concerns**: Each component can be optimized independently (database, LLM, workflow)
3. **Modular Design**: Easy to add new features (new question types, data sources, agents, validations)
4. **Horizontal Scaling**: Clear path to add read replicas, caching, API gateway, microservices
5. **Portability**: Docker ensures consistent deployment across any environment
6. **Cost-Effective**: No unnecessary infrastructure, can scale incrementally as needed

---

## Requirements Validation

All requirements have been validated and are satisfied. See `REQUIREMENTS_VALIDATION.md` for detailed validation results.
