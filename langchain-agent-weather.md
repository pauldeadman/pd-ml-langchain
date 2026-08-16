# Create Weather Search Agent

## Create location to latitude/longitude

```
from geopy.geocoders import Nominatim
geolocator = Nominatim(user_agent="weather-app")
```

## Create Tool

```
@tool("get_weather_forecast", args_schema=SearchInput, return_direct=True)
def get_weather_forecast(location: str, date: str):
    location = geolocator.geocode(location)
    if location:
        try:
            response = requests.get(f"https://api.open-meteo.com/v1/forecast?latitude={location.latitude}&longitude={location.longitude}&hourly=temperature_2m&start_date={date}&end_date={date}")
            data = response.json()
            return dict(zip(data["hourly"]["time"], data["hourly"]["temperature_2m"]))
        except Exception as e:
            return {"error": str(e)}
    else:
        return {"error": "Location not found"}

tools = [get_weather_forecast]
```

## Create interface to Gemini

```
from langchain_google_genai import ChatGoogleGenerativeAI

# Create LLM class
llm = ChatGoogleGenerativeAI(
    model= "gemini-3-flash-preview",
    temperature=1.0,
    max_retries=2,
    google_api_key=api_key,
)
```

## Test the model

```
model = llm.bind_tools([get_weather_forecast])
res=model.invoke(f"What is the weather in Berlin on {datetime.today()}?")
print(res)
```

## Workflow Create State Data

```
from langchain_core.messages import BaseMessage
from langgraph.graph.message import add_messages  # helper function to add messages to the state
class AgentState(TypedDict):
    """The state of the agent."""
    messages: Annotated[Sequence[BaseMessage], add_messages]
    number_of_steps: int
```

## Workflow Create Basic Model

To make it easier to understand (top down description) we will define the call_model and call_tool later

```
from langgraph.graph import StateGraph, END

workflow = StateGraph(AgentState)
workflow.add_node("llm", call_model)
workflow.add_node("tools",  call_tool)
```

Define the "START"
```
workflow.set_entry_point("llm")
```

## Workflow Now add an IF statement

A function should_continue will return "continue" if last message is a tool call or "end".

```
workflow.add_conditional_edges("llm",should_continue,{"continue": "tools","end": END,})
```

Link node "tools" back to "llm"

```
workflow.add_edge("tools", "llm")
```

# Workflow Compile

```
graph = workflow.compile()
```

## Workflow Helpers

This is the easy bit where call_model calls the LLM and returns the full list of messages.

```
def call_model(
    state: AgentState,
    config: RunnableConfig,
    ):
    response = model.invoke(state["messages"], config)
    return {"messages": [response]}
```

Construct a data model for the tools

```
tools_by_name = {tool.name: tool for tool in tools}
```

To iterate over the tools in "last message", using invoke to call the tools with arguments. ToolMessage is a helper from langchain to stitch together the message data. 

```
def call_tool(state: AgentState):
    outputs = []
    for tool_call in state["messages"][-1].tool_calls:
        tool_result = tools_by_name[tool_call["name"]].invoke(tool_call["args"])
        outputs.append(
            ToolMessage(
                content=tool_result,
                name=tool_call["name"],
                tool_call_id=tool_call["id"],
            )
        )
    return {"messages": outputs}
```

And finally check last message for a "tools_call" in the last message.

```
def should_continue(state: AgentState):
    messages = state["messages"]
    if not messages[-1].tool_calls:
        return "end"
    return "continue"
```

## Full Implementation

[Open Meteo Weather Agent](https://github.com/pauldeadman/pd-ml-langchain/blob/main/LLM-Gemini-Agent-Weather-pub.ipynb)

