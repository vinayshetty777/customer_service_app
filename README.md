# Customer Service App

AI-powered customer service agent built on Databricks using LLM tool-calling. Integrates with vector search, UC functions, and policy lookup for intelligent customer support.

## 📋 Overview

An intelligent customer service agent leveraging Databricks, LLMs, and tool-calling to provide accurate, context-aware customer support. The agent retrieves product information via vector search, looks up company policies, and accesses customer history to deliver personalized responses.

**Tech:** Databricks, MLflow, Claude AI, Vector Search, SQL Functions  
**Features:** Real-time policy lookup, customer history retrieval, semantic search  
**Status:** 🤖 Production-Ready AI Agent

---

## 🏗️ Architecture

```
User Query
    ↓
[AI Agent - Tool Calling]
    ├→ Vector Search (Semantic Search)
    │  └→ Product Documentation
    ├→ SQL Functions (Structured Data)
    │  ├→ get_return_policy()
    │  └→ get_service_history()
    └→ LLM (Databricks Endpoint)
        └→ Generate Response
    ↓
Customer Response
```

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|------------|
| **Platform** | Databricks |
| **ML Framework** | MLflow, Responses Agent |
| **LLM** | Claude AI (Anthropic) |
| **Search** | Vector Search Index |
| **Functions** | Unity Catalog Functions |
| **Database** | Delta Lake, SQL |

---

## 📁 Project Structure

```
customer_service_app/
├── tool_calling_agent.ipynb       # Main agent code
├── Parse_PDF_Docs.py.ipynb        # PDF extraction
├── Enrich_pdf_docx.ipynb          # Document enrichment
├── vector_search.ipynb            # Search testing
├── Policy_Lookup_Function.ipynb   # SQL functions
├── pyproject.toml
├── uv.lock
└── README.md
```

---

## 🚀 Quick Start

### Prerequisites
- Databricks workspace
- Python 3.12+
- Vector search index configured
- UC functions deployed

### Installation

```bash
# Clone repository
git clone https://github.com/vinayshetty777/customer_service_app.git
cd customer_service_app

# Install dependencies
uv sync

# Set environment variables
export DATABRICKS_TOKEN=<your-token>
export DATABRICKS_HOST=<workspace-url>
```

---

## 📊 Data Flow

```
Customer Question
    ↓
[Agent Analysis]
    ├→ Intent recognition
    ├→ Context retrieval
    └→ Tool selection
    ↓
[Parallel Tool Calls]
    ├→ Vector Search (Product docs)
    ├→ get_return_policy() (Policies)
    └→ get_service_history() (Customer history)
    ↓
[LLM Processing]
    ├→ Information synthesis
    ├→ Response generation
    └→ Confidence scoring
    ↓
Customer Answer
```

---

## ✨ Key Features

- **Tool-Calling Agent** - Dynamic tool invocation
- **Vector Search** - Semantic product search
- **Policy Lookup** - Instant policy retrieval
- **Customer History** - Personalized context
- **Streaming Responses** - Real-time output
- **MLflow Tracing** - Complete audit trail

---

## 🚀 Usage

### Single Query

```python
from agent import AGENT

response = AGENT.predict({
    "input": [{"role": "user", "content": "What is your return policy?"}],
    "custom_inputs": {"session_id": "session-123"}
})
```

### Streaming Response

```python
for chunk in AGENT.predict_stream({
    "input": [{"role": "user", "content": "Tell me about warranties"}],
    "custom_inputs": {"session_id": "session-456"}
}):
    print(chunk.model_dump(exclude_none=True))
```

### With Conversation History

```python
messages = [
    {"role": "user", "content": "Do you have laptops in stock?"},
    {"role": "assistant", "content": "Yes, we have several options..."},
    {"role": "user", "content": "What's the warranty?"}
]

response = AGENT.predict({
    "input": messages,
    "custom_inputs": {"session_id": "test-123"}
})
```

---

## 🔧 Tools Available

### Vector Search
- **Index:** product_docx_index
- **Type:** Hybrid search (dense + sparse)
- **Returns:** Relevant product documentation

### SQL Functions

**get_return_policy(policy_name: STRING)**
- Returns: policy, policy_details, last_updated

**get_service_history(user_email: STRING)**
- Returns: returns_last_12_months, issue_category, todays_date

---

## 📈 Monitoring

- **MLflow Traces** - Tool execution tracking
- **LLM Calls** - Token usage and latency
- **Session Tracking** - Conversation continuity
- **Quality Metrics** - Response evaluation

---

## 🧪 Testing & Evaluation

```python
# Test with evaluation dataset
eval_dataset = [
    {
        "inputs": {
            "input": [{"role": "user", "content": "What is the return policy?"}]
        }
    }
]

eval_results = mlflow.genai.evaluate(
    data=eval_dataset,
    predict_fn=lambda inp: AGENT.predict({"input": inp, "custom_inputs": {"session_id": "eval"}}),
    scorers=[RelevanceToQuery(), Safety()]
)
```

---

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| Tool not found | Verify UC function exists in catalog |
| Vector search empty | Check index is populated and configured |
| Agent not responding | Verify LLM endpoint and API token |

---

## 🔐 Best Practices

✅ **Do:**
- Monitor token usage
- Test with real queries
- Review policy lookup results
- Log all interactions
- Update function documentation

❌ **Don't:**
- Expose API keys
- Skip response validation
- Ignore tool failures
- Disable error logging

---

## 🤝 Contributing

1. Test agent thoroughly
2. Document tool changes
3. Update configurations
4. Submit PR with examples

---

## 📚 Resources

- [Databricks Agent Framework](https://docs.databricks.com/generative-ai/agent-framework/)
- [MLflow Documentation](https://mlflow.org/docs/)
- [Claude API Docs](https://docs.anthropic.com/)
- [Vector Search Guide](https://docs.databricks.com/en/generative-ai/search/vector-search.html)

---

**Last Updated:** 2026-10-04  
**Status:** Production Ready  
**Python Version:** ≥3.12
