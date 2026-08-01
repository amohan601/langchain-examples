
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

**Invoke model**

Invoke OpenAI model without any base URL. Same class can be used with base url for groq calling open ai model. we can invoke the model like below 

```
client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
client = Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])
client = OpenAI(api_key=os.environ["GROQ_API_KEY"], base_url="https://api.groq.com/openai/v1")
client = OpenAI(api_key=os.environ["OPENROUTER_API_KEY"], base_url="https://openrouter.ai/api/v1")
```
**Invocation (Non-Anthropic)**
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
