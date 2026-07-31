# LangChain Examples

Practice notebooks and exercises from **Krish Naik's** Udemy courses, including LangChain examples from [The AI Agent Engineer Course — Complete AI Agent Bootcamp](https://blue.udemy.com/course/the-ai-agent-engineer-course-complete-ai-gent-bootcamp).

## Contents

| File | Description |
|------|-------------|
| `langchain_examples.ipynb` | Main notebook covering OpenAI API usage, LangChain fundamentals, and RAG |
| `requirements.txt` | Python dependencies and environment setup notes |
| `.env` | API keys (not committed); create locally with `OPENAI_API_KEY` |

### Topics covered

**OpenAI API**
- Chat completions (system/user messages)
- Sarcastic chatbot and sentiment classification examples
- `max_completion_tokens`, temperature, and streaming responses

**LangChain**
- `ChatOpenAI` model invocation
- Human/system messages and few-shot prompting
- Prompt templates and prompt values
- Output parsers (including comma-separated lists)
- LangChain Expression Language (LCEL): chaining, batching, streaming
- `RunnablePassThrough`, `RunnableParallel`, and `RunnableLambda`
- Chain visualization with the `grandalf` library

**Retrieval-Augmented Generation (RAG)**
- Document loading (PDF, DOCX)
- Text splitting (character and markdown splitters)
- Document embedding and vector stores (ChromaDB)
- Document retrieval (similarity search, MMR)
- LLM response generation from retrieved context

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

4. Launch Jupyter and open `langchain_examples.ipynb`.
