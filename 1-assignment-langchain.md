
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


__7.Explain why streaming produces AIMessageChunk objects instead of plain text fragments, and what property of these chunks makes them genuinely different from a string split into pieces.__

Streaming returning AIMessageChunk because each chunk is a partial AIMessage when combined to give the full message. Each AIMessage is not merely a string but can carry structured message information in addition to content. 

__8.What's the difference between .batch() and .batch_as_completed()? Describe a real situation where you would specifically want the second one over the first.__
.batch() will wait for all the request to be processed and response is generated. .batch_as_completed() will start returning results as soon as its available. This is useful if we want to show results of unrelated questions that are submitted in a batch and want to start showing the response immediately. 

__9.A ToolMessage has both a .content field and an .artifact field. What's the difference, and why would a RAG-style tool specifically want to use .artifact?.__
.content is visible to the model as an input, while .artifact is not made visible to the model by langchain. it can contain internal application data/information that are not needed for the model to see. A RAG tool wants this to attach a document ID or citation link the UI needs, without bloating what the model has to process.

__10.You're writing a ChatPromptTemplate whose system message needs to include a literal JSON example like {"status": "ok"}. What will go wrong if you paste that in directly, and how do you fix it?__
ChatPromptTemplate treats {..} as tempalte variable. So if you want to include literal strings in curly braces you have to escape it with another curly brace outside. LangChain tries to interpret the literal { and } in the JSON example as MORE template variables, and fails (INVALID_PROMPT_INPUT}

__11.What does MessagesPlaceholder do, and why can't you achieve the same result with a normal string-based template variable?__

MessagePlaceHolder is useful if you have to insert a list of already-created LangChain messages into a ChatPromptTemplate. A normal variable like ("human", "{history}") cannot do the same because it will treat history as a string text not as a list of messages. 
```
history = [
    HumanMessage(content="My name is John"),
    AIMessage(content="Nice to meet you, John."),
    HumanMessage(content="What is my name?")
]
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    MessagesPlaceholder("history"),
    ("human", "{question}")
])
prompt.invoke({
    "history": history,
    "question": "What did I tell you?"
})
```








