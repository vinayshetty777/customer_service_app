# Agentic AI - Customer Service Application

An AI-powered customer service agent built on Databricks that leverages LLMs, vector search, and tool-calling to provide intelligent customer support. The agent can retrieve product information, policies, and customer service history to answer customer queries accurately.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Data Flow](#data-flow)
- [Components](#components)
- [Setup & Installation](#setup--installation)
- [Usage](#usage)
- [Contributing](#contributing)

## 🎯 Overview

This application implements an intelligent customer service agent that:
- **Retrieves product information** using semantic search over vectorized product documentation
- **Looks up company policies** through SQL functions
- **Accesses customer service history** to provide personalized responses
- **Leverages LLMs** (via Databricks serving endpoints) for natural conversation
- **Uses tool-calling** to dynamically access data and services
- **Scales intelligently** with Databricks infrastructure

## 🏗️ Architecture

### High-Level Architecture Flow

```
User Query
    ↓
[AI Agent - Tool Calling]
    ├→ Vector Search (Semantic Search)
    │  └→ Product Documentation Index
    ├→ SQL Functions (Structured Data)
    │  ├→ get_return_policy()
    │  └→ get_service_history()
    └→ LLM (Databricks Serving Endpoint)
        └→ Generate Response
    ↓
Customer Response
```

### Component Architecture

#### 1. **Data Layer** (Databricks Unity Catalog)

- **`products`** - Product metadata (id, name, category, sub-category)
- **`product_docs`** - Extracted product documentation from PDFs
- **`product_docs_combined`** - Enriched documents with XML-formatted indexing
- **`cust_service_data`** - Customer service history and interactions
- **`policies`** - Company policies (returns, refunds, privacy, etc.)

#### 2. **Document Processing Pipeline**

```
PDF Files
    ↓
[Parse PDF Documents]
    └→ Extract text content from product PDFs
    ↓
[Enrich Documents]
    ├→ Join with product metadata
    ├→ Create XML-formatted indexed documents
    └→ Prepare for embedding
    ↓
[Vector Search Index]
    └→ Create embeddings for semantic search
```

#### 3. **Agent Layer**

```
[Tool-Calling Agent]
    │
    ├─ [LLM Backbone]
    │  └→ Databricks LLama 4 Maverick (or configured endpoint)
    │  └→ Handles reasoning and conversation
    │
    ├─ [Tool Interface]
    │  ├→ UC Functions (Structured data access)
    │  ├→ Vector Search (Semantic retrieval)
    │  └→ Custom Functions (Business logic)
    │
    └─ [Response Generation]
       └→ Synthesizes information from multiple sources
```

#### 4. **Deployment Architecture**

```
[Databricks Model Serving]
    ├→ Model Endpoint (Scale-to-zero capable)
    ├→ MLflow Tracking (Tracing & Monitoring)
    └→ Unity Catalog Registration
```

## 📁 Project Structure

```
customer_service_app/
├── README.md                           # This file
├── pyproject.toml                      # Project metadata and dependencies
├── .python-version                     # Python version specification
├── uv.lock                            # Dependency lock file
│
├── tool_calling_agent.ipynb           # Main agent implementation
│   ├─ Agent class definition
│   ├─ Tool configuration
│   ├─ LLM integration
│   ├─ Testing & evaluation
│   └─ Model registration & deployment
│
├── Parse_PDF_Docs.py.ipynb            # PDF extraction pipeline
│   ├─ PDF file processing
│   ├─ Text extraction
│   └─ DataFrame creation
│
├── Enrich_pdf_docx.ipynb              # Document enrichment pipeline
│   ├─ Product metadata join
│   ├─ XML indexing format
│   └─ Table creation for vector search
│
├── vector_search.ipynb                # Vector search testing
│   ├─ Semantic search queries
│   ├─ Hybrid search examples
│   └─ Index validation
│
└── Policy_Lookup_Function.ipynb       # SQL Function definitions
    ├─ get_return_policy() function
    └─ get_service_history() function
```

## 🔄 Data Flow

### 1. Data Preparation Phase

```
Source PDFs
    ↓
[Parse_PDF_Docs.py.ipynb]
    ├─ Extract text from product PDFs
    ├─ Create product_docs table
    └─ Store raw product documentation
    ↓
products table (existing)
    ↓
[Enrich_pdf_docx.ipynb]
    ├─ Join products + product_docs
    ├─ Create XML-formatted indexed_doc column
    └─ Create product_docs_combined table
    ↓
[Databricks Vector Search]
    └─ Generate embeddings
    └─ Create product_docx_index (HYBRID search enabled)
```

### 2. Agent Query Phase

```
User Question
    ↓
[Tool-Calling Agent]
    │
    ├─ Analyze query intent
    │
    ├─ [Parallel Tool Calls]
    │  ├─ Vector Search
    │  │  └─ Find relevant product docs
    │  ├─ UC Function: get_return_policy()
    │  │  └─ Fetch policy details
    │  └─ UC Function: get_service_history()
    │     └─ Get customer history
    │
    ├─ [LLM Processing]
    │  └─ Synthesize information
    │  └─ Generate contextual response
    │
    └─ Return Response
```

## 🛠️ Components

### Agent (`tool_calling_agent.ipynb`)

The core agent implementation based on MLflow's `ResponsesAgent`:

```python
class ToolCallingAgent(ResponsesAgent):
    - Configuration: LLM endpoint, system prompt
    - Tool Management: Register and execute tools
    - LLM Integration: OpenAI SDK with Databricks serving
    - Stream Handling: Support streaming responses
    - Tool Execution: Parallel & sequential tool calls
```

**Key Features:**
- Parallel tool execution for efficiency
- Streaming response support
- Tool call tracing via MLflow
- Session tracking for conversation context

### Tools Available

#### Vector Search Tool
```
VectorSearchRetrieverTool
├─ Index: agentic_catalog.agentic_schema.product_docx_index
├─ Search Type: HYBRID (dense + sparse)
└─ Returns: Relevant product documentation
```

#### UC Functions (Structured Data)

**`get_return_policy(policy_name: STRING)`**
- Input: Policy name (e.g., "Refund Policy", "Return Policy")
- Returns: policy, policy_details, last_updated
- Use case: Policy lookups

**`get_service_history(user_email: STRING)`**
- Input: Customer email
- Returns: returns_last_12_months, issue_category, todays_date
- Use case: Customer history analysis

### Document Processing

#### PDF Parsing (`Parse_PDF_Docs.py.ipynb`)
- Reads PDF files from Volumes
- Extracts text content per page
- Creates product_docs table with product_name and product_doc columns

#### Document Enrichment (`Enrich_pdf_docx.ipynb`)
- Joins products + product_docs on product_name
- Creates XML-formatted indexed_doc for LLM processing:
  ```xml
  <product_category>Electronics</product_category>
  <product_sub_category>Laptops</product_sub_category>
  <product_name>ProBook 15</product_name>
  <product_doc>
  Detailed specifications, features, warranty info...
  </product_doc>
  ```
- Saves to product_docs_combined table
- Enables Databricks vector search indexing

## 🚀 Setup & Installation

### Prerequisites
- Databricks workspace with Unity Catalog enabled
- Python 3.12+
- Access to Databricks serving endpoints
- Appropriate permissions in UC (catalog, schema, functions)

### Installation

1. **Clone and navigate to project:**
   ```bash
   cd customer_service_app
   ```

2. **Set Python version:**
   ```bash
   # Install via pyenv
   pyenv install 3.12
   pyenv local 3.12
   ```

3. **Install dependencies:**
   ```bash
   # Using uv package manager
   uv sync
   
   # Or using pip
   pip install -e .
   ```

4. **Configure Databricks credentials:**
   ```bash
   # Set environment variable
   export DATABRICKS_TOKEN=<your-token>
   export DATABRICKS_HOST=<workspace-url>
   ```

### Configuration

Edit the agent configuration in `tool_calling_agent.ipynb`:

```python
# LLM Endpoint
LLM_ENDPOINT_NAME = "databricks-llama-4-maverick"  # Change as needed

# System Prompt
SYSTEM_PROMPT = """You are a helpful customer service agent..."""

# UC Tools
UC_TOOL_NAMES = [
    "agentic_catalog.agentic_schema.get_service_history",
    "agentic_catalog.agentic_schema.get_return_policy"
]

# Vector Search Indexes
VECTOR_SEARCH_TOOLS = [
    VectorSearchRetrieverTool(
        index_name="agentic_catalog.agentic_schema.product_docx_index"
    )
]
```

## 📊 Usage

### Running the Agent

In a Databricks notebook:

```python
from agent import AGENT

# Single query
response = AGENT.predict({
    "input": [{"role": "user", "content": "What is your return policy?"}],
    "custom_inputs": {"session_id": "session-123"}
})

# Streaming response
for chunk in AGENT.predict_stream({
    "input": [{"role": "user", "content": "Tell me about laptop warranties"}],
    "custom_inputs": {"session_id": "session-456"}
}):
    print(chunk.model_dump(exclude_none=True))
```

### Testing the Agent

```python
# Test with conversation history
messages = [
    {"role": "user", "content": "Do you have a laptop in stock?"},
    {"role": "assistant", "content": "We have several laptop options..."},
    {"role": "user", "content": "What's the warranty?"}
]

response = AGENT.predict({
    "input": messages,
    "custom_inputs": {"session_id": "test-123"}
})
```

### Evaluation

The agent includes evaluation capabilities using MLflow:

```python
eval_dataset = [
    {
        "inputs": {
            "input": [{"role": "user", "content": "What is the return policy?"}]
        },
        "expected_response": None
    }
]

eval_results = mlflow.genai.evaluate(
    data=eval_dataset,
    predict_fn=lambda inp: AGENT.predict({"input": inp, "custom_inputs": {"session_id": "eval"}}),
    scorers=[RelevanceToQuery(), Safety()]
)
```

### Deployment

Register and deploy the agent to Databricks Model Serving:

```python
# Register to UC
mlflow.set_registry_uri("databricks-uc")
registered_model = mlflow.register_model(
    model_uri=logged_agent_info.model_uri,
    name="agentic_catalog.agentic_schema.customer_service_model"
)

# Deploy
from databricks import agents
agents.deploy(
    model_uri="agentic_catalog.agentic_schema.customer_service_model",
    version=registered_model.version,
    scale_to_zero=True  # Optional: for cost savings
)
```

## 🔧 Development

### Adding New Tools

1. **UC Function Tool:**
   ```python
   UC_TOOL_NAMES.append("catalog.schema.my_new_function")
   ```

2. **Vector Search Tool:**
   ```python
   VECTOR_SEARCH_TOOLS.append(
       VectorSearchRetrieverTool(
           index_name="catalog.schema.my_index",
           tool_description="Searches my custom documents"
       )
   )
   ```

3. **Custom Tool:**
   ```python
   class CustomTool(ToolInfo):
       name = "my_tool"
       spec = {...}  # OpenAI tool format
       def exec_fn(self, **kwargs):
           # Implementation
           pass
   
   TOOL_INFOS.append(CustomTool(...))
   ```

### Modifying System Prompt

Update the `SYSTEM_PROMPT` variable to customize agent behavior:

```python
SYSTEM_PROMPT = """
You are a professional customer service representative. 
- Be helpful and friendly
- Always check policies before providing information
- Include relevant warranty details
- Offer solutions that match company guidelines
"""
```

## 📈 Monitoring

The agent includes comprehensive MLflow tracing:

- **Tool execution traces** - Track each tool call and execution time
- **LLM call traces** - Monitor token usage and latency
- **Session tracking** - Group related interactions
- **Quality metrics** - Evaluate agent responses

View traces in Databricks MLflow UI:
```
Workspace → MLflow → Experiments → agent
```

## 🐛 Troubleshooting

### Issue: Tool not found
- Verify UC function exists in Unity Catalog
- Check function name format: `catalog.schema.function_name`
- Confirm permissions to access the function

### Issue: Vector search returns empty
- Verify index exists and is populated
- Check index name is correct
- Ensure documents are properly formatted and embedded

### Issue: Agent not responding
- Check LLM endpoint is running and accessible
- Verify DATABRICKS_TOKEN is set correctly
- Check MLflow traces for LLM call errors

## 📝 Contributing

When contributing to this project:

1. Add documentation for new features
2. Update this README if architecture changes
3. Test all new tools in isolated notebooks first
4. Follow the existing code structure and naming conventions
5. Include MLflow tracing for new operations

## 📄 License

This project is part of the Agentic AI initiative.

## 📞 Support

For issues or questions:
1. Check the Troubleshooting section
2. Review MLflow traces in Databricks UI
3. Consult Databricks documentation for agent frameworks
4. Check tool-specific documentation in notebooks

---

**Project Version:** 0.1.0  
**Last Updated:** 2025  
**Python Version:** ≥3.12
