
# KrishNaik AgenticAI 3.0 notes

## Langchain

### Basics
---

Model - you ask question you get a answer

Chatbot - you pass all previous questions and answers and ask a new question. agent answers the new question keeping in mind the past coversation. conversation with some memory

Agent - you can ask question and get answer, pass conversation, and also pass some tools for the agent to utilize for generating the answers. 

**Types of Models**

You can use AI models to ask question. 

Closed source models - OpenAI,Anthropic - You have to buy credits

Open/closed source models - Groq,Openrouter - some free models and some paid models

OpenAI came first. So other models api and contract are OpenAI

**API Keys**

Different providers expect their API KEY to be set up in the environment variable before they can be invoked
* OPENAI_API_KEY
* ANTHROPIC_API_KEY
* GROQ_API_KEY
* OPENROUTER_API_KEY

**Invoke model**

Invoke OpenAI model without any base URL. Same class can be used with base url for groq calling open ai model. we can invoke the model like below 

```
client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
client = Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])
client = OpenAI(api_key=os.environ["GROQ_API_KEY"], base_url="https://api.groq.com/openai/v1")
client = OpenAI(api_key=os.environ["OPENROUTER_API_KEY"], base_url="https://openrouter.ai/api/v1")
```
**Invocation (Non-Anthropic)**

Every question we ask the model can be user or assistant question defined by the "role".

```
response = client.chat.completions.create(
    model="gpt-4o-mini", **change to the model supported by respective provider**
    max_tokens=200,
    messages=[{"role": "user", "content": question}],
)
```
**Output**
```
response.choices[0].message.content
```
**Anthropic Invocation**
```
response = client.messages.create(
        model="claude-3-5-haiku-20241022",
        max_tokens=200,
        messages=[{"role": "user", "content": question}],
    )
```
**Output answer**
```
response.content[0].text
```
**Setting up tool and tool schema for model**
```
response = client.chat.completions.create(
        model=<model to be used>,
        max_tokens=300,
        messages=[{"role": "user", "content": question}],
        tools=[get_weather_schema, get_tool_schema_for_llm],
    )
```
**weather schema**
```
get_weather_schema = {
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get the current weather for a city. Use this whenever "
                        "the user asks about weather, temperature, or conditions "
                        "in a specific place. Don't use it for AQI"
                        },
        "parameters": {
            "type": "object",
            "properties": {
                "city": {"type": "string", "description": "The city name, e.g. 'Tokyo'."}
            }
                            },
}
```

**tools schema**
```
get_tool_schema_for_llm={
    "type": "function",
    "function": {
        "name": "get_tool_schema_for_llm",
        "description": "Get the schema of a tool by its name. Use this whenever " 
                        "the user asks for the schema of a specific tool.",     
        "parameters": {
            "type": "object",
            "properties": {
                "tool_name": {"type": "string", "description": "The name of the tool, e.g. 'get_weather'."}
            },
            "required": ["tool_name"],
        },
    },
}
```
### Transitioning from Model to Agent using tool call

**model response with tool schema**
```
If the question does not include asking about weather then model does not include tool_calls. 
Otherwise it include tool_calls which tells us what tool we should call

Example - 
Hi How are you ?

Model's raw reply: 
ChatCompletionMessage(content="I'm just a computer program, 
so I don't have feelings, but I'm here and ready to help you! How can I assist you today?", 
refusal=None, role='assistant', 
annotations=[], 
audio=None, function_call=None, 
tool_calls=None)


Example - 
what is the weather like in Tokyo right now?

Model's raw reply: 
ChatCompletionMessage(content=None, refusal=None, 
role='assistant', annotations=[], audio=None, 
function_call=None, 
tool_calls=
[ChatCompletionMessageToolCall(id='call_x4l82uTQXd3UR6Nw4Nq8wy48', 
function=Function(arguments='{"city":"Tokyo"}', 
name='get_weather'), type='function')]
)

```
Once above response is recieved using that tool_name 'get_weather' we have to make
actual python function call to get the tool response. Agent performs this tool call in a loop until there are no more tool calls needed. Depending on how many tools we provide and how many tools are needed to answer the question model may decide to include one or more tool details in the tool_calls response. 
```
def get_weather(city: str) ->str:
    return f'Weather in {city} is Sunny 22F'
  
tools = {'get_weather': get_weather}
for msg in tool_calls:
    tool_fun = tools[msg.function.name]
    arguments = msg.function.arguments
    tool_fun(**arguments)
```
For each response from tool_call we can pass that back to model as a assistant role question.
```
1st pass
---------
message (to model) -> [{'role': 'user', 'content': 'weather in tokyo'}] --> message1

response (from model) -> 
ChatCompletionMessage(
  content=None, refusal=None, 
  role='assistant', annotations=[], audio=None, 
function_call=None, 
tool_calls=[ChatCompletionMessageFunctionToolCall(id='call_DLFkn8JhjBQdGb1AzNSre4cH', function=Function(arguments='{"city":"Tokyo"}', name='get_weather'), 
type='function')])

**<perform tool call get_weather to get information from the tool>**

2nd pass
---------

messages (to model)-> [
{'role': 'user', 'content': 'weather in tokyo'}, -->message1
{'role': 'assistant', 'content': None,  'tool_calls': [{'id': 'call_DLFkn8JhjBQdGb1AzNSre4cH', 'type': 'function', 'function': {'name': 'get_weather', 'arguments': '{"city":"Tokyo"}'}}]}, --->message2
 {'role': 'tool', 'tool_call_id': 'call_DLFkn8JhjBQdGb1AzNSre4cH', 'content': 'Tokyo: 22C, partly cloudy'} --->message3
 ]

<!- Model uses tool call response to general natural language answer ->
response  (from model) -> ChatCompletionMessage(
content='The current weather in Tokyo is 22°C and partly cloudy.', 
refusal=None, 
role='assistant', annotations=[], 
audio=None, function_call=None, 
tool_calls=None)

**<no more tool calls agent stops and respond back with results >**

Final answer: 
Agent: The current weather in Tokyo is 22°C and partly cloudy.

```
## Agents

Agent = LLM  + Tools + Memory

### Agent frameworks
* Langchain
* Langgraph
* OpenAI Agent SDK
* Google ADK
* AWS Strands
* CrewAI
* Llama Index
* n8n
* Langflow

### Langchain Family
Langchain is an Agent development framework. Three things offered in langchain family are
* Langchain - create agent with some base setup
* Langgraph - foundation only create agents on your own
* Deep agents - ready to use agents
* Langsmith - monitoring,tracing, and tracking agents

#### langchain

  * latest version 1.3.13 v1 version or the latest version)
  * langchain-classic is the older version package (prior to langchain 1.0)

**What is Agent ? What is Harness ?**

Agent = Model + Harness. Langchain provides the create_agent method that can help to create an agent that is minimal work but highly configurable. We can pass in the model, tools, memory and middleware the that shape the **create_agent** and agent will perform tool calling, looping and middleware execution based on this harness. 
Harness is everything around the model loop - the prompt, the tools, the middleware and anything that define the agent behavior. We can define the power of model better using this harness thus making it a "Agent". 

https://docs.langchain.com/oss/python/langchain/agents

TBD: Langgraph and Deep agents to be explained later,

Example of how differnt products are different 

Deep agent - is like swiggy, you cannot do any deep control of food creation
Langchain agent - home cooked meal some control is possible in recipe
langgraph - just vegetables shared you get to cook everything more control 

**Create Agent in Langgraph**

It internally invokes the model with tool to get the tool call suggestion then it loops through the tool result to make tool call as python code and then it pass that result to model to form a natural language based result from the tool call response. 
```
from langchain.agents import create_agent
from langchain.tools import tool
from langchain.chat_models import init_chat_model

@tool
def get_weather(city: str) -> str:
    """Define the weather for a city
    """
    return "It is sunny and 22F"

model = init_chat_model(
    "openai:gpt-5.5",
    temperature=0.5,
    timeout=300,
    max_tokens=25000,
)

agent = create_agent(
    model = model,
    tools =[get_weather], 
    system_prompt = 'you are a helpful weather assistant' )

result = agent.invoke({"messages": [{"role": "user", 
                                    "content": "what is the weather in tokyo? "}]})

 print(result["messages"][-1].content_blocks)                                   
```
* *Make sure to have the respective provider of the model OPENAI_API_KEY be set up as environment variable loaded using load_env* <br>
* *Make sure to have respecticve provider library for langchain is also installed eg: langchain-openai* <br>
* *Different providers will have slightly different format for init_chat_model creation*

Result will have below <br>
* HumanMessage <br>
* AIMessage(with tool call info) <br>
* ToolMessage(with tool call result)<br>
* AIMessage(final message from model using tool call result)<br>

### Setting up virtual environment
Open a terminal  and run:
```bash
pip install uv (if uv does not exist)
uv init langchain-course
cd langchain-course
uv add langchain langchain-openai langchain-community langgraph python-dotenv
uv add langchain-mcp-adapters langchain-chroma chromadb pypdf
```
It creates .venv file, .env file, pyproject.toml file, README.md, and uv.lock files 




### AI Model types
#### Free
* OpenRouter (contain free and paid models from different providers) (langchain-openrouter library)
* Groq
#### Paid
* OpenAI
* Anthropic
* Gemini

#### Closed source
We dont have information about weights, tuning etc
* openrouter (has closed source models for free and paid)

#### Open source
We have information about details of model, we can download and run within our infra. 
* Ollama (multiple models provided by Ollama)
* Deepseek
* LMStudio

##### openrouter selecting a free model

Below syntax will ensure openrouter will route this request to one of its free models. 
```
model = init_chat_model(
    "openrouter:free",
    model_provider="openrouter",
)
```

### Models
---

```
from langchain.chat_models import init_chat_model

model = init_chat_model(
    "claude-sonnet-4-6",
    # Kwargs passed to the model:
    temperature=0.7,
    timeout=30,
    max_tokens=1000,  --> define the max tokens we can receive in response
    max_retries=6,  # Default; increase for unreliable networks
)
```
#### HumanMessage, SystemMessage to model 
```
from langchain_core.messages import SystemMessage,HumanMessage

agent.invoke(messages = [
    SystemMessage(content = 'You are a pirate. Answer in pirate language'),
    HumanMessage (content = 'what is the capital of france? ')
])
```
Other variables with in response
```
print(response.content)
print("text :                    ",response.text)
print("content_blocks            ", response.content_blocks)
print("id:                       ", response.id)
print("tool_calls                ", response.tool_calls)
```
Output
```
Arrr, the capital o' France be Paris, matey!
text :                     Arrr, the capital o' France be Paris, matey!
content_blocks             [{'type': 'text', 'text': "Arrr, the capital o' France be Paris, matey!"}]
id:                        lc_run--019f7395-602d-7e71-8ed7-89ec75573389-0
tool_calls                 []
```

#### Fewshot prompting to model 
Give information to agent with examples in the form of system message, human message and ai message. Then Agent knows how to respond based on this example. 
```
from langchain_core.messages import HumanMessage,SystemMessage,AIMessage

messages = [
    SystemMessage("You are a helpful assistant"),
    HumanMessage("Can you help me?"),
    AIMessage("I'd be happy to help you with that question!"),

    HumanMessage("Great! What's 2+2?"), --> This is my actual question.
]
response = model.invoke(messages)
print(response.content)
```
Another way to give same thing

```
conversation = [
    {"role": "system", "content": "You are a helpful assistant that translates English to French."}, --> SystemMessage
    {"role": "user", "content": "Translate: I love programming."}, -->HumanMessage
    {"role": "assistant", "content": "J'adore la programmation."}, -->AIMessage
    {"role": "user", "content": "Translate: I love building applications."} -->HumanMessage (My question)
]

response = model.invoke(conversation)
print(response)  # AIMessage("J'adore créer des applications.")

```
### Invoking the model
* invoke (we already saw this - AIMessage output)
* stream. (AIMessageChunk output)
* batch (AIMessage output)

#### Streaming with models 
Stream the output content of a model that takes time to generate the full response so that it gives better user experience and when output is long or takes time to generate fully. if the model supports streaming, the output can be streamed.
 model.invoke(), returns a single **AIMessage** after the model has finished generating its full response, model.stream() returns multiple **AIMessageChunk** objects, each containing a portion of the output text. 
```
for chunk in model.stream("Why do parrots have colorful feathers?"):
    print(chunk.text, end="|", flush=True)
```

#### Batching with model
Send multiple request to model in one batch to improve performance and reduce cost , as the processing can be done in parallel.
Call the model using **model.batch()**
```
q1 = 'hi, how are you ?'
q2 = 'tell about AI in less than 100 words?'
q3 = 'Explain agents in 30 words ?'

responses = openai_model.batch([
    q1,
    q2,
    q3
])

for response in responses:
  print(response)

```

#### Stream while batch is executed
You can print the response while batch is generating output instead of waiting till the end.

```
 
for response in openai_model.batch_as_completed([
    "Why do parrots have colorful feathers?",
    "How do airplanes fly?",
    "What is quantum computing?"
]):
    print(response)
```

#### Binding tools to model
When a model decides it needs to call a tool, that request sits in the exact `.tool_calls`
field from section 3.3 -- nothing runs yet, this is purely a *request*.
Use **model.bind_tools** to bind these tools. The response is an AIMessage with .tool_calls section that suggest what tool we should call from the list of tools we provided. 
```
def get_weather(location: str) -> str:
    """Get the weather at a location."""
    return f"Sunny in {location}"

def set_password(new_pass: str) -> str:
    """Set a new password."""
    return "Password changed"

model_with_tools = model.bind_tools([get_weather, set_password])
response = model_with_tools.invoke("Set the password to admin123")

for tool_call in response.tool_calls:
    print(f"Tool: {tool_call['name']}")
    print(f"Args: {tool_call['args']}")
    print(f"ID:   {tool_call['id']}")
```

#### Sending message to model (another way)
```
result = model.invoke([
    SystemMessage("You are a helpful assistant"),
    HumanMessage("Can you help me?"),
    AIMessage("I'd be happy to help you with that question!"),
    ToolMessage(content=<result_message_from_python_tool_call>, tool_call_id="call_123")
])
```

## Middleware (callbacks or hooks)
---
* Helps to understand and control more tightly what happens inside the agent. 
* Tracking agent behavior with logging, analytics, and debugging 
* transforming prompts, tool selections, output formatting
* adding retries, fallbacks, early termination logic
* apply rate limit, guard rails, pii detection
* we have option to do before_agent, before_model, after_agent, after_model setup, wrap_tool_call(before tool call), wrap_model_call
* middle ware is one way to apply guard rails

![Middleware diagram.](/middleware1.png "Middleware")
Code for this section 
<a href="langchain-middleware.ipynb">langchain-middleware.ipynb</a>
* Pre-Built in middleware
* Custom middleware

Example of Pre-Built middleware from langchain (Called before tool call)

#### Summarization middleware
* **Summarization** - 	Automatically summarize conversation history when approaching token limits. <a href="https://docs.langchain.com/oss/python/langchain/middleware/built-in#summarization">Summarization docs</a>


```
from langchain_core.tools import tool
from langchain.agents import create_agent
from langchain.messages import SystemMessage,HumanMessage
from langchain.agents.middleware import SummarizationMiddleware

@tool
def book_movie(movie:str) -> str:
    """book a movie """
    print('[tool] Inside book_movie')
    return f"the movie {movie} is booked"

@tool
def cancel_booking(movie:str) -> str:
    """cancel a movie """
    print('[tool] Inside cancel_booking')
    return f"the movie {movie} is cancelled"
    
@tool
def check_showtimes(movie_title:str) -> str:
  """Check available showtimes for a movie at the cinema.

  Args:
      movie_title: The exact title of the movie to check
  """
  print('[tool] Inside check_showtimes')
  fake_showtimes = {
      "interstellar": "7:00 PM and 10:15 PM",
      "dune part two": "9:30 PM only",
      "oppenheimer": "Sold out for tonight",
  }
  return fake_showtimes.get(movie_title.lower(), "No showtimes found for that title.")

agent = create_agent(
    model='openai:gpt-5-mini',
    tools=[book_movie,cancel_booking,check_showtimes],
     middleware=[
        SummarizationMiddleware(
            model="gpt-5.4-mini",
            trigger=("tokens", 50),
            keep=("messages", 1),
        )
    ],
    )

print(agent.invoke({
    "messages": [
        SystemMessage(content="You are a cine bot assistant"),
        HumanMessage(content="what time does interstellar run? can u book one movie for me?")
    ]
}))        
```

when agent is invoked with a question, depending on the trigger summarization kicks in. It summarizes the input to the model and sends the summarized HumanMessage to the model. The lc_source="summarization" in the HumanMessage indicates that summarization middleware was called. 
Summarization Middleware is checked everytime before a model is called. It only kicks in if the trigger condition is met though. 

```
print(agent.invoke({
    "messages": [
        SystemMessage(content="You are a cine bot assistant"),
        HumanMessage(content="what time does interstellar run? can u book one movie for me?")
    ]
}))
```
```
Original conversation
        ↓
Summarization Middleware
        ↓
Compressed representation
        ↓
HumanMessage(lc_source="summarization")
        ↓
Model
```
Original message was 
```
[
    SystemMessage(...),
    HumanMessage("what time does interstellar run?..."),
    AIMessage(...),
    ToolMessage(...),
    AIMessage(...)
]
```
But what actually got is below
```[
    HumanMessage(
        "Here is a summary of the conversation..." --> created with summarization middleware
    ),

    AIMessage(
        content="",
        tool_calls=[...] --> model response with suggestion about tools to call 
    ),

    ToolMessage(...),  --> tool response from the tool call

    AIMessage(
        content="Do you mean the movie's runtime..." --> models final response using the tool result
    )
]

```

The output of summarization middleware looks like below
```
SESSION INTENT
The user wants to know the runtime of Interstellar
and asks to book a movie ticket.

SUMMARY
The user asked:
"what time does interstellar run?
 can u book one movie for me?"

Missing:
- theater/location
- date
- showtime
- booking preferences

NEXT STEPS
Ask for the missing booking details.

Also clarify whether "what time" means:
- movie runtime
- movie showtimes
```

#### HITL middleware
* Human-In-The-Loop - HITL Middleware
It allows us to interrupt and get the user involved before the tool is called. 
<a href="https://reference.langchain.com/python/langchain/agents/middleware/human_in_the_loop/HumanInTheLoopMiddleware">HITL Docs</a>
```
from langchain.agents import create_agent
from langchain.agents.middleware import HumanInTheLoopMiddleware
from langgraph.checkpoint.memory import InMemorySaver


def your_read_email_tool(email_id: str) -> str:
    """Mock function to read an email by its ID."""
    return f"Email content for ID: {email_id}"

def your_send_email_tool(recipient: str, subject: str, body: str) -> str:
    """Mock function to send an email."""
    return f"Email sent to {recipient} with subject '{subject}'"

agent = create_agent(
    model="gpt-5.5",
    tools=[your_read_email_tool, your_send_email_tool],
    checkpointer=InMemorySaver(),
    middleware=[
        HumanInTheLoopMiddleware(
            interrupt_on={
                "your_send_email_tool": {
                    "allowed_decisions": ["approve", "edit", "reject"],
                },
                "your_read_email_tool": False,
            }
        ),
    ],
)

config = {'configurable':{"thread_id":"hitl"}}
result = agent.invoke({"messages" : [
    ("user", "First call the Read email aj123@gmail.com and tell me the subject. Do not call send email until read email is completed. Next, Send an email to my manager on aj1234@gmail.com, asking for a leave, dont include any reason. its for today.")
]},config = config)
```
The HITL middleware is invoked right before the tool is called and it generates an interrupt that we can see in the response AIMessage. 

We can resume the interrupt using below. Note that since continiuty is needed in this conversation to resume after interrupt inmemorycheckpointer is needed. 
```
resumed_result = agent.invoke(Command(resume={"decisions":[{"type":"approve"}]}),config=config)
```
In the above example it will first give HumanMessage for our input request. Then it gives 
AIMessage -> indicating read email tool call is needed. Then it gives ToolMessage for that call. Then it gives AIMessage for send email tool call needed. However this step also includes interrupt since HITL is specified for this tool. Once the above command is given to resume the full response will have ToolMessage for this tool call followed by the final message for send email tool call as AIMessage. 

#### Model Call Limit
* Model call limit - Limit the number of model calls to prevent excessive costs.

ModelCallLimitMiddleware uses thread_limit which is across all conversations how many invoke(or model call) and run_limit is for a single conversation how many times invoke is allowed 
```
from langchain.agents.middleware import ModelCallLimitMiddleware
call_limited_agent = create_agent(
    model="openai:gpt-5-mini",
    tools=cinebot_tools,
    checkpointer=InMemorySaver(),  # required for thread_limit to persist across calls
    middleware=[
        ModelCallLimitMiddleware( # low values taken just for example
            thread_limit=5,   # across the WHOLE conversation
            run_limit=2,       # per single .invoke() call
            exit_behavior="end",  # graceful stop, not an exception
        ),
    ],
)
```

### Model fallback middleware

If the existing model invocation fails due to some error then we can plugin in model fallback middleware that can call another model to do the work. 

```
from langchain.agents.middleware import ModelFallbackMiddleware
try:    
    model_fallback_agent = create_agent(
        model="openai:gpt-5-mini-test", --> this will fail
        tools=[tell_story_movie,tell_story_book],
    middleware=[
        ModelFallbackMiddleware(
            "openai:gpt-5-mini" -->the call will fallback to this model
        ),
    ],
    )
    #result = guarded_agent.invoke({"messages": [("user", "Please cancel booking BK1042")]}, config=config)
    result = model_fallback_agent.invoke({"messages": [HumanMessage('Tell me the story of movie Dunn.')]})
    print(result)
except:
    print('error')
```
This is helpful especially when your provider/model has an outage or even intermittent issues this fallback is helpful. The context and everything is automatically passed on internally to the fallback model. You can provide a list of fallback models to ModelFallbackMiddleware so that it can be chained and whichever first successful fallback is used. 

### TollCallLimit Middleware
* Tool call limit - Control tool execution by limiting call counts.
some usecases are not allow many database queries - restrict it with a limit, or prevent certain actions in a flow after certain number of tries etc. 
You specify this limit for all or specific tools. You set it up with either run limit or thread limit or both. 
```
@tool
def create_booking(movie:str) ->str:
    """create booking for a movie"""
    return f"movie {movie} booked"

@tool
def cancel_booking(movie:str) ->str:
    """cancel booking for a movie"""
    return f"movie {movie} booking cancelled"

tool_call_limit_agent = create_agent(
    model="openai:gpt-5-mini",
    checkpointer=InMemorySaver(),
    tools=[create_booking,cancel_booking],
    middleware=[
        ToolCallLimitMiddleware(run_limit=2,thread_limit=4), -> applicable for all tools 
        ToolCallLimitMiddleware(tool_name='cancel_booking', run_limit=1,thread_limit=1), --> applicable only for cancellation tool
    ],
)
config = {"configurable": {"thread_id": "tool-limit-demo"}}
tasks = ['create','cancel','cancel','create']
from rich import print
result = None
for index,item in enumerate(tasks):
    print('------------')
    print(index,item)
    result = tool_call_limit_agent.invoke({"messages" : [HumanMessage(content=f'{item} booking for movie with id B{index}')]},config = config)
print(result)
```
* booking b0 to 'create' booking is called.
* booking b1 to 'cancel' booking is called
* booking b2 to 'cancel' booking is rejected because cancel_booking tool call has thread_limit = 1
* booking b3 to 'create' booking is called. 

#### PII Middleware

we have multiple ways to implement guard rails. one of the ways is using pii middleware. using good system prompt is another way.

```
pii_agent = create_agent(
    model="openai:gpt-5-mini",
    tools=[create_booking,cancel_booking],
    middleware=[
        PIIMiddleware("email", strategy="redact", apply_to_input=True),
        PIIMiddleware("credit_card", strategy="mask", apply_to_input=True),    ],
)
result = pii_agent.invoke({"messages" : [HumanMessage(content='Here is my booking id: b123. My email is aj123@gmail.com. here is my credit card number 4111-1111-1111-1234. please create booking for movie dunn')]})
print(result)

```
it can apply that pii rule to the input data. only the applied/modified data goes into the model. 

you can also apply custom masking for custom fields. in this example the custom id is masked based on regex rules. BK**** only will go into the model and tools. 

```
def detect_booking_code(content: str) -> list[dict]:
    """Detect CineBot's own booking code format: BK followed by 4 digits."""
    matches = []
    for match in re.finditer(r"BK\d{4}", content):
        matches.append({"text": match.group(0), "start": match.start(), "end": match.end()})
    return matches
    
custom_pii_agent = create_agent(
    model="openai:gpt-5-mini",
    tools=[check_booking_status],
    middleware=[PIIMiddleware("booking_code", detector=detect_booking_code, strategy="mask")],
)

result = custom_pii_agent.invoke({
    "messages": [("user", "Can you check the status of my booking BK1044 for me?")]
})
```

#### To-do list middleware
Helps to plan complex task. It gives a to-do list at the end. it creates a todo list as the output. 
```
@tool
def check_show_timing() ->str:
    """check show timing for all movies"""
    return f"movie dunn is at 11:00, movie fearnot is at 5:00, movie interstellar is at 9:00"

@tool
def create_booking(movie:str, tickets: int) ->str:
    """create booking for a movie"""
    return f"movie {movie} is booked for {tickets} people"
    
@tool
def cancel_booking(movie:str) ->str:
    """cancel booking for a movie"""
    return f"movie {movie} booking cancelled"

    
todo_agent = create_agent(
    model="openai:gpt-5-mini",
    tools=[check_show_timing, create_booking, cancel_booking],
    middleware=[TodoListMiddleware()]
)
result = todo_agent.invoke({
    "messages": [("user", "I want to plan a movie night: check what's showing, pick something good science related, and book 2 seats.")]
})
from rich import print
print(result)
```

output has todo list like below. it will use tools and call tools as needed to prepare the list and the status. In this example it called check_show_timing and selected the movie from that info, then called create_booking and send that movie to book. we can see that in toolsmessage it passed the correct movie name and number of tickets to book in the create_booking call. 
```
'todos': [
        {'content': 'Check show timings for all movies', 'status': 'completed'},
        {'content': 'Select a science-related movie to watch (Interstellar at 9:00)', 'status': 'completed'},
        {'content': 'Book 2 seats for chosen movie (Interstellar)', 'status': 'completed'},
        {'content': 'Confirm booking details with user', 'status': 'in_progress'}
    ]
```
### LLMToolSelectorMiddleware
Uses an LLM to select relevant tools before calling the main model.
we can define the max tool that it can send in addition to the always_include tools. 
The tool is selected by the middleware based on the message in the invoke. 
```
tool_selector_agent = create_agent(
    model="openai:gpt-5-mini",
    tools=[check_show_timing, create_booking, cancel_booking,refund_booking,confirm_booking,confirm_cancellation],
    middleware=[LLMToolSelectorMiddleware( 
            model="openai:gpt-5-mini",     # can be a CHEAPER model than the main agent
            max_tools=2, #only give two tools at any given time
            always_include=["check_show_timing"]),
               show_tools
            ]
)
result = tool_selector_agent.invoke({
    "messages": [HumanMessage(content="Can you cancel my booking with ID B1234?")]
})
```
The model was sent first with 3 tools and model actually called cancel and confirm cancel tools. Once tools response was recieved from both tools using ToolsMessage then model formed the final response for that also middleware sent the 3 tools cancel, confirm cancel, check show timing. 
TOOLS SENT TO MODEL:
['cancel_booking', 'confirm_cancellation', 'check_show_timing']

TOOLS SENT TO MODEL:
['cancel_booking', 'confirm_cancellation', 'check_show_timing']

### ToolErrorMiddleware 
<a href="https://docs.langchain.com/oss/python/langchain/middleware/built-in#tool-error-full-example">ToolError</a>
Toolerror middleware can act on errors using the on_error method. if that method is called when tool throws exception, and it returns some result then agent.invoke will not throw any additional exception for the tool error. The response from on_error method is what becomes the ToolMessage for the tool call that failed, instead of agent throwing an exception. 

```
@tool
def divide_into_half(num: int) ->int:
    """this is a divide into half tool"""
    print(f'[tool call] divide_into_half {num}')
    return num/0
    


def on_error(exc: Exception, request: ToolCallRequest) -> str | None:
    print(f'on_error {request.tool_call['name']}, failed with {type(exc).__name__}. ')
    return f"`{request.tool_call['name']}` failed with {type(exc).__name__}."
    # this returns helps to ensure the agent.invoke does not throw exception out of it and it exits gracefully. 


agent = create_agent(
    model="openai:gpt-5-mini",
    tools=[divide_into_half],
    middleware=[ToolErrorMiddleware(on_error)],
)
try:
    result = agent.invoke({"messages": [HumanMessage(content="Divide 5 into half")]})
    from rich import print
    print(result)
except Exception as e:
    print('error')
```

#### ToolRetryMiddleware
Retry tool with exponential backoff and max retries. 

Retry a tool call when failure happens. Can help to retry when there are network failures that causes the tool call to fail. 

When tool call fails first time, it will go and do 3 retries based on ToolRetryMiddleware configuration. Upon exhausting all retries it will go to "error" state(on_failure ="error") and then it will throw exception causing ToolErrorMiddleware to kick in and capture the error inside on_error method. 
```
@tool
def divide_into_half(num: int) ->int:
    """this is a divide into half tool"""
    print(f'[tool call] divide_into_half {num}')
    if num == 0:
        raise ValueError('number is zero')
    return num/0
    

def on_error(exc: Exception, request: ToolCallRequest) -> str | None:
    print(f'on_error {request.tool_call['name']}, failed with {type(exc).__name__}. ')
    return "Make sure to give a non zero number" 
    # propagate everything else

agent = create_agent(
    model="openai:gpt-5-mini",
    tools=[divide_into_half],
    middleware=[ToolErrorMiddleware(on_error),
                ToolRetryMiddleware(max_retries=3, on_failure="error")
               ],
)
try:
    result = agent.invoke({"messages": [HumanMessage(content="Divide 0 into half")]})
    from rich import print
    print(result)
except Exception as e:
    print('error')
```
Note the tool message produced. 
```
ToolMessage(
            content='Make sure to give a non zero number',
            name='divide_into_half',
            id='27e232d0-cf87-40e7-aa57-ad95078a4899',
            tool_call_id='call_ajxRaDheeJQUBoxI9Fcy0jlh',
            status='error'
        ),
```
Below output is produced. \
 divide_into_half 0 \
 divide_into_half 0 >>>retries \  
 divide_into_half 0 >>>retries \
 divide_into_half 0 >>>retries \
on_error divide_into_half, failed with ValueError. >> got inside on_error method  

Comparison of how order matters for toolerror and toolretry
```
count: int = 0;
@tool
def divide_into_half(num: int) ->int:
    """this is a divide into half tool"""
    global count
    count+=1
    print(f'[tool call] divide_into_half num = {num},  execution count = {count}')
    
    if num == 0:
        raise ValueError('number is zero')
    return num/0
    

def on_error(exc: Exception, request: ToolCallRequest) -> str | None:
    print(f'on_error {request.tool_call['name']}, failed with {type(exc).__name__}. ')
    return "Make sure to give a non zero number" 
    # propagate everything else

def on_error2(exc: Exception, request: ToolCallRequest) -> str | None:
    print(f'on_error2 {request.tool_call['name']}, failed with {type(exc).__name__}. ')
    raise ValueError('number is zero')
    
print('testing out toolerror and then toolretry middleware both together in that order')
agent1 = create_agent(
    model="openai:gpt-5-mini",
    tools=[divide_into_half],
    middleware=[
                ToolErrorMiddleware(on_error),
                ToolRetryMiddleware(max_retries=5, on_failure="error")
               ],
)
try:
    result = agent1.invoke({"messages": [HumanMessage(content="Divide 0 into half")]})
    from rich import print
    #print(result)
except Exception as e:
    print('error')

print('testing out toolretry and then toolerror middleware both together in that order')
count = 0

agent2 = create_agent(
    model="openai:gpt-5-mini",
    tools=[divide_into_half],
    middleware=[
                ToolRetryMiddleware(max_retries=5, backoff_factor=2.0, initial_delay=1.0, on_failure="error"),
                ToolErrorMiddleware(on_error),
               ],
)
try:
    result = agent2.invoke({"messages": [HumanMessage(content="Divide 0 into half.")]})
    from rich import print
    #print(result)
except Exception as e:
    print('error')

print('testing out toolretry and then toolerror middleware both together in that order without error handling')
count = 0

agent3 = create_agent(
    model="openai:gpt-5-mini",
    tools=[divide_into_half],
    middleware=[
                ToolRetryMiddleware(max_retries=5, backoff_factor=2.0, initial_delay=1.0, on_failure="error"),
                ToolErrorMiddleware(on_error2),
               ],
)
try:
    result = agent3.invoke({"messages": [HumanMessage(content="Divide 0 into half.")]})
    from rich import print
    #print(result)
except Exception as e:
    print('error')
```

case 1 is like below
```
ToolErrorMiddleware
    │
    ▼
ToolRetryMiddleware ->>> this catches the error and retries until 
it exhausts the limit before it gives the control back to 
toolerror middleware
    │
    ▼
divide_into_half
```
case 2 is like below
```
ToolRetryMiddleware > 
    │
    ▼
ToolErrorMiddleware >>>> when the tool executes first time, this middleware 
capture the error and process it gracefully and exits. So there will not be any retries. 
The error is no longer propagated upward. Instead, the error middleware has 
transformed the exception into a successful tool result. if the error is not handled and thrown then it sends the control back to retry tool and it goes to next retry. 

So from the retry middleware's perspective: 
    │
    ▼
divide_into_half
```
#### LLMEmulatorMiddleware
Emulate tool execution using an LLM for testing purposes, replacing actual tool calls with AI-generated responses. Default model is used by LLM Emulator internally, and we can specify which tools to be emulated. 

```
@tool
def get_weather(location: str) -> str:
    """Get the current weather for a location."""
    print(f'[tool] get_weather {location}')
    return f"Weather in {location}"

@tool
def send_email(to: str, subject: str, body: str) -> str:
    """Send an email."""
    print(f'[tool] send_email {to} {subject}  {body}')
    return "Email sent"


# Emulate all tools (default behavior)
emulator_agent = create_agent(
    model="openai:gpt-5-mini",
    tools=[get_weather, send_email],
    middleware=[LLMToolEmulator(model="openai:gpt-5-mini")]
)

# Emulate specific tools only
emulator_agent2 = create_agent(
    model="openai:gpt-5-mini",
    tools=[get_weather, send_email],
    middleware=[LLMToolEmulator(model="openai:gpt-5-mini", tools=["get_weather"])],
)
result = emulator_agent.invoke({"messages": [HumanMessage(content="what is the weather in tokyo?.  send an email to aj123@gmail.com with subject langchain and body here is what we learned about middleware")]})
from rich import print
print(result)
result = emulator_agent2.invoke({"messages": [HumanMessage(content="what is the weather in tokyo?.  send an email to aj123@gmail.com with subject langchain and body here is what we learned about middleware")]})
print(result)
```
For the first agent, it emulates weather call tool. The send email tool is not emulated and actual call happens there. 
