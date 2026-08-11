# LangChain Examples

Practice notebooks and exercises from **Krish Naik's** courses.

**Udemy —** LangChain fundamentals and RAG from [The AI Agent Engineer Course — Complete AI Agent Bootcamp](https://blue.udemy.com/course/the-ai-agent-engineer-course-complete-ai-gent-bootcamp)

**Krish Naik Academy —** LangChain 3.0 examples (tools and structured schema) from the [Krish Naik Academy](https://www.krishnaik.in/) Agentic AI curriculum

## Contents

| File | Course | Description |
|------|--------|-------------|
| `langchain_examples.ipynb` | Udemy bootcamp | OpenAI API usage, LangChain fundamentals, and RAG |
| `langchain-tools.ipynb` | Krish Naik Academy (LangChain 3.0) | Defining and using LangChain tools with `init_chat_model` |
| `langchain-structured-schema.ipynb` | Krish Naik Academy (LangChain 3.0) | Structured output with `ToolStrategy`, `ProviderStrategy`, and agent `response_format` |
| `requirements.txt` | — | Python dependencies and environment setup notes |
| `.env` | — | API keys (not committed); create locally with `OPENAI_API_KEY` |

### Topics covered

**`langchain_examples.ipynb` (Udemy)**

*OpenAI API*
- Chat completions (system/user messages)
- Sarcastic chatbot and sentiment classification examples
- `max_completion_tokens`, temperature, and streaming responses

*LangChain*
- `ChatOpenAI` model invocation
- Human/system messages and few-shot prompting
- Prompt templates and prompt values
- Output parsers (including comma-separated lists)
- LangChain Expression Language (LCEL): chaining, batching, streaming
- `RunnablePassThrough`, `RunnableParallel`, and `RunnableLambda`
- Chain visualization with the `grandalf` library

*Retrieval-Augmented Generation (RAG)*
- Document loading (PDF, DOCX)
- Text splitting (character and markdown splitters)
- Document embedding and vector stores (ChromaDB)
- Document retrieval (similarity search, MMR)
- LLM response generation from retrieved context

**`langchain-tools.ipynb` (Krish Naik Academy — LangChain 3.0)**
- LangChain 3.0 `init_chat_model` setup
- Tool definitions with Pydantic schemas and the `@tool` decorator
- Structured output alongside tool use

**`langchain-structured-schema.ipynb` (Krish Naik Academy — LangChain 3.0)**
- Structured output at model and agent levels (`with_structured_output`, `response_format`)
- `ToolStrategy` vs `ProviderStrategy` for structured schemas
- Union schemas, multi-format support, and validation error handling

## Setup

1. Create and activate a conda environment (Python 3.9 recommended).
2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Add your OpenAI API key to a `.env` file:

```
OPENAI_API_KEY=your-key-here
```

4. Launch Jupyter and open the notebook for the course you are following.
