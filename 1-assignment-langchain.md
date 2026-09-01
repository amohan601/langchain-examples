
# KrishNaik AgenticAI 3.0 questions and answers.

### Part A — Conceptual Questions
#### A1. Foundations (Easy)

<a href="https://github.com/mayank953/Live-Class-2026/blob/main/Assignment%20-%20Middleware/Assignment_LangChain_Fundamentals_SOLUTIONS.ipynb" target="_blank">Assignment answers from the instructor</a>

**1. In your own words, explain the sentence: "An agent is a model calling tools in a loop until a task is complete. 
A harness is everything around that loop." What specifically counts as part of the "harness"?**

Harness is responsible for conversation management to pass down the past conversations to model which is otherwise known as the context. 
It helps with tool management where it binds the model to the tool, and helps to actually run the python code of the tool. It also manages the memory that sometimes is needed for the agent to remember things about the customer. It helps to handle logging, and error handling of tool calls. 
Thus harness is the surrounding runtime that manages context, memory, tools, permissions, state, retries, logging, and other infrastructure needed to run the agent safely and reliably.

**2. Name the four products in the Lang family (LangChain, LangGraph, LangSmith, Deep Agents) and state, in one sentence each, what job each one does. Which one is fundamentally different in kind from the other three, and why?**

Langchain - create agent with some base setup
Langgraph - foundation only create agents on your own
Deep agents - ready to use agents
Langsmith - monitoring,tracing, and tracking agents

__3. If you found a 2026-dated tutorial using AgentExecutor or initialize_agent, what would you conclude, and what should you use instead?__
If I found a 2026-dated tutorial still using AgentExecutor or initialize_agent, I would conclude that the tutorial is likely teaching the legacy LangChain API rather than the current LangChain v1 agent API. LangChain's current documentation says create_agent is the standard way to build agents in LangChain 1.0, and legacy functionality has been moved toward langchain-classic

__4.Why does a .env file exist? What specifically goes wrong if you skip it and hardcode an API key directly into a notebook cell?__
.env keeps a real API key out of the notebook file itself. Hardcoding it directly means the moment that notebook is committed to GitHub or shared, the key is exposed — often found and abused by automated scanners within minutes.

__5.What is the difference between a plain text prompt, a message-object list, and a dictionary-based message list? Give one situation where each is the natural choice.__

Plain text single standalone request. Message objects in the form of SystemMessage, HumanMessage, AIMessage and so on to provide list of messages in a conversational format of specifc message type. Dictionaries are message objects for specific message types identifed using role but the format is of type JSON. 

#### A2. Models, Messages, and Templates (Medium)
__6. An AIMessage carries more than .content. Name at least four other fields or attributes it can carry, and what each one is actually useful for.__
print("text :                    ",response.text) \
print("content_blocks            ", response.content_blocks)\
print("id:                       ", response.id)\
print("tool_calls                ", response.tool_calls)\
print("usage_metadata                ", response.usage_metadata)\
print("response_metadata                ", response.response_metadata)\
