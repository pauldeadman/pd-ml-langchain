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


```
def call_model(
    state: AgentState,
    config: RunnableConfig,
    ):
    response = model.invoke(state["messages"], config)
    return {"messages": [response]}
```


