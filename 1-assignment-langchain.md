
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

#### A3. Structured Output (Medium-Hard)


__12. Explain, in your own words, why asking a model to "please respond in JSON" via plain prompt instructions is fundamentally less reliable than using with_structured_output().__

Model is fuzzy and it can choose not to product a json or produce something close enough. with pydantic models using with_structured_output, model is forced to produce the output json that is validated to pydantic model standards. 

__13. What is model.profile, and how is it actually used internally when you call with_structured_output() without specifying a strategy explicitly?__

model.profile is a capability dictionary attached to a LangChain chat model. LangChain uses it to know what the model supports, such as tool calling, multimodal input, context size, and native structured output. Langchain docs follow below check in order. 

if schema is provided corrected, then it checks model's profile capabilities. if native sturctured output is supported, then provider strategy is used, otherwise tool strategy is used.


__14.A teammate writes: model.with_structured_output(BookingRequest, strategy=ProviderStrategy(BookingRequest)) and asks you to code review it. What's wrong with this line, and what should it be instead?__
strategy= is not an option with with_structured_output. it get silently absorbed into kwargs and still fallback to default strategy option. Instead one should use model.with_structured_output(ProviderStrategy(BookingRequest)).

__15.What's the difference between structured output at the model level (with_structured_output) and at the agent level (response_format on create_agent)? Why does almost everything from Part 7 onward use the agent-level version? __
Model-level (with_structured_output) has no awareness of tools or a tool-calling loop — it just returns a parsed object directly. Agent-level (response_format on create_agent) coexists with tools, returning result["structured_response"] alongside the normal message trace. Almost everything from Part 7 onward uses create_agent, so the agent-level version is what's actually needed in practice.

__16.You have a schema response_format=ToolStrategy(Union[NewReservation, CancelReservation]). Explain what happens internally when the model receives a message that's genuinely ambiguous between the two schemas.__
Model tries to generate response with a given format based on its understanding of the context. It may generate the response in one call or may get ambiguous and may take multiple turns until it settle in on a specifc format. 

__17.A schema field is defined as party_size: int = Field(ge=1, le=20), and a customer message says "table for 50 please." Walk through, step by step, what happens inside the agent loop from the moment the model first proposes party_size=50 to the moment a valid final answer is produced.__

Imagine ToolStrategy is used in response_format. The important part is that ToolStrategy validates the model's proposed structured output against the schema and, by default, catches structured-output errors and retries.

User: "table for 50 please"
        ↓
Model proposes:
BookingRequest(party_size=50)
        ↓
LangChain validates against Pydantic schema
        ↓
Validation fails: 50 > 20
        ↓
StructuredOutputValidationError
        ↓
LangChain turns that failure into feedback for the model
        ↓
Model gets another turn
        ↓
Model produces a corrected response
        ↓
Schema validates
        ↓
structured_response returned


if provider strategy was used instead - \
User
 │
 │ "table for 50 please"
 ▼
Agent
 │
 ▼
Model
 │
 │ requests structured output:
 │ party_size = 50
 ▼
Provider's structured-output validation (provider is validating and not langchain)
 │
 │ 50 violates le=20
 ▼
Provider rejects the response

With ToolStrategy, LangChain can detect the validation failure and feed the error back to the model for another attempt. With ProviderStrategy, schema enforcement is handled natively by the model provider, so LangChain doesn't use the same tool-call validation/retry loop. The provider either produces a schema-valid result or the API may fail, depending on the provider's behavior.

The model first proposes party_size=50. Pydantic validates it against ge=1, le=20 and rejects it. That validation error — naming the exact constraint violated — is fed back to the model as a message. The standard agent loop (unmodified) lets the model see that error and retry with a corrected value, repeating until a value satisfies the constraint or handle_errors gives up according to its configured behavior.

#### A4. Tools (Hard)
__18.A tool's docstring is described as "the tool's entire pitch to the model," not documentation for humans. Defend or challenge this claim — is there ever a case where the docstring genuinely doesn't matter much?__

Largely defensible — the docstring genuinely is what the model reads to decide whether and when to call a tool. A case where it matters less: a tool that's the ONLY option available and always needed on every turn — but even then, a bad docstring risks the model second-guessing whether to call it at all, so it's rarely truly irrelevant.

__19. Explain what ToolRuntime actually hides from the model, and how LangChain knows to hide it (i.e., what specifically triggers this behavior)?__
The model doesn't know how to construct a ToolRuntime. LangChain supplies it internally when the tool executes. ToolRuntime is not treated as regular argument and model does not see it automatically. ToolRuntime contains state, contet, stream_write, execution_info and so that is required for the runtime execution. langchain identify this argument using the ToolRuntime type in the argument list. 
ToolRuntime hides state, context, store, execution_info, and server_info from the model — none of these appear in the tool's schema. LangChain detects this by recognizing the ToolRuntime TYPE ANNOTATION on the parameter named runtime, and automatically excludes it when building what the model sees.

__20.Compare runtime.state, runtime.context, and runtime.store. For each one, state: (a) how long the data persists, and (b) one concrete example of information that belongs there.__

runtime.state - contains all state informations for a given thread id. if same thread is used across invocations, it can The retrieve the state. state can be modified between invocations. 
runtime.context - contains information that cannot be modified. it will need to be passed down to every invocation as a fresh value. It is not retained between invocations. 

rumtime.store - contains long term memory information stored using InMemoryStore or other kinds of store where information can be persisted and pulled out for reference. It is not tied to a specific thread or invocation. 

__21.A tool accidentally declares a parameter named config. What actually happens when the agent tries to call it, and why is this a genuinely easy mistake to make by accident?__

config is reserved internally by the framework for RunnableConfig. Using it as a regular argument name causes the framework to intercept it before the function body ever runs, typically producing a TypeError about a missing required argument. Easy to hit by accident because config is an unremarkable, common variable name with no obvious reason to expect it's reserved.

__22 Explain the difference between a tool returning a plain string versus returning a Command. Give an original example (not from any notebook you've seen) of a situation that specifically requires Command, and explain why a plain string return wouldn't work for that case.__

A plain string only ever ANSWERS a question. Command can write directly to agent STATE. Original example: a tool that upgrades a customer's loyalty tier after a large order — it needs to both confirm the upgrade AND persist the new tier into state for later tools/turns to read; a plain string return has no mechanism to change anything beyond the current reply.

__23. Describe, precisely, why wrap_model_call-based tool gating (making a tool invisible) is a stronger guarantee than instructing the model in the system prompt not to use a tool.__

With system prompt it is left to the model to decide what tool to call. With wrap style model, we can customize the model logic to remove the rool from the request itself thus making the tool invisible to the model. 

__24.What is a headless tool, and how is its execution model fundamentally different from every other tool pattern covered in this course? Name one realistic capability that could only be implemented this way.__
A headless tool has its schema registered on the server (so the model can call it normally), but its actual EXECUTION happens on the CLIENT — typically a browser — via an interrupt/resume handshake, rather than running in the Python process. Example: reading the user's real-time geolocation from the browser's Geolocation API — a Python server has no way to access that directly; only client-side code can.
