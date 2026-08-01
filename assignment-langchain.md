# KrishNaik AgenticAI 3.0 questions and answers.

### Part A — Conceptual Questions
#### A1. Foundations (Easy)

__1. In your own words, explain the sentence: "An agent is a model calling tools in a loop until a task is complete. 
A harness is everything around that loop." What specifically counts as part of the "harness"?__

Harness is responsible for conversation management to pass down the past conversations to model which is otherwise known as the context. 
It helps with tool management where it binds the model to the tool, and helps to actually run the python code of the tool. It also manages the memory that sometimes is needed for the agent to remember things about the customer. It helps to handle logging, and error handling of tool calls. 
Thus harness is the surrounding runtime that manages context, memory, tools, permissions, state, retries, logging, and other infrastructure needed to run the agent safely and reliably.

__2. Name the four products in the Lang family (LangChain, LangGraph, LangSmith, Deep Agents) and state, in one sentence each, what job each one does. Which one is fundamentally different in kind from the other three, and why?__
