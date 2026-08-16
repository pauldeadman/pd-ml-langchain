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

