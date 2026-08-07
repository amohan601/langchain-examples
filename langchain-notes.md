
# KrishNaik AgenticAI 3.0 notes

### Langchain

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


### HumanMessage, SystemMessage
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

### AI Model providers
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
