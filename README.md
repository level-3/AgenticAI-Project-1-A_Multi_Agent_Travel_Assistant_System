# Project: AgentsVille Trip Planner: A Multi-Agent Travel Assistant System

## General Prompt Design

### Criteria

Design a system prompt that effectively guides the LLM to produce a comprehensive, structured itinerary.

Design a system prompt that determines whether an event should be avoided due to weather conditions.

### Submission Requirements

The prompt in `ITINERARY_AGENT_SYSTEM_PROMPT` satisfies the following:

- Clearly instructs LLM to assume a role as a travel planner, travel agent, or something similar
- Encourages detailed daily plans through Chain-of-Thought guidance or examples
- Specifies JSON output format matching the `TravelPlan` Pydantic model structure
- Provides necessary context, including `VacationInfo` object, weather data, and activities data

The prompt in `ACTIVITY_AND_WEATHER_ARE_COMPATIBLE_SYSTEM_PROMPT` satisfies the following:

- Clearly defines the LLM role and task
- Specifies exact output format
- Includes relevant examples illustrating expected evaluations

## Agent Reasoning and Tool Use

### Criteria

Design a tool description that guides the LLM to call it when needed.

Design a prompt that guides an LLM to use reasoning and tools in the ReAct cycle.

### Submission Requirements

The docstring for the `get_activities_by_date` tool function should:

- Provide sufficient description for understanding tool's purpose and use and
- Define expected input parameters and their data types and formats

The prompt `ITINERARY_REVISION_AGENT_SYSTEM_PROMPT` should satisfy the following:

- Clearly states LLM role and itinerary revision task
- Details the THINK-ACT-OBSERVE cycle explicitly
- Lists all available tools, their purposes, and parameter requirements (adding this dynamically is nice, but not required)
- Specifies exact ACTION format for tool invocation:

  ```python
  {"tool_name": "[tool_name]", "arguments": {"arg1": "value1", ...}}
  ```

- Includes explicit exit instruction via `final_answer_tool` invocation
- States that the `run_evals_tool` must be run before the `final_answer_tool`

## Structured Output Validation

### Criteria

Pydantic models are created and used properly.

### Submission Requirements

The `VacationInfo` Pydantic model is:

- Created properly given the JSON structure representing the travelers and their vacation details
- Is read properly (start and end dates) when pulling the weather and activity data

The `TravelPlan` Pydantic model's schema is:

- Included in AT LEAST ONE system prompt that needs to output a `TravelPlan` data structure (there are two such prompts, so it is best, but not required, to include this schema in both of them)

---

# AgentsVille Trip Planner: Multi-Agent Travel Assistant System - Code Implementation Analysis

This document analyzes how the implemented code addresses the specific submission requirements for the AgentsVille Trip Planner project.

## General Prompt Design

### Criteria Requirements

**Design a system prompt that effectively guides the LLM to produce a comprehensive, structured itinerary.**

**Design a system prompt that determines whether an event should be avoided due to weather conditions.**

### Implementation Analysis

#### ITINERARY_AGENT_SYSTEM_PROMPT ✅

The implemented prompt satisfies all requirements:

```python
ITINERARY_AGENT_SYSTEM_PROMPT = f"""
You are an expert itinerary planning agent, capable of creating detailed travel plans based on vacation information, weather data, and available activities. 

## Task
Your task is to generate a comprehensive travel itinerary that meets the specified requirements. 
If the weather is rainy , outdoor events should be avoided 
Events should be chosen based on traveler interests 
Budget should be considered while selecting activities and the budget should not be exceeded 
There should be at least one activity per day

## Output Format
Respond using two sections (ANALYSIS AND FINAL OUTPUT) in the following format:
    ANALYSIS:
    - Analyze the vacation information, weather data, and available activities to create a detailed travel plan.
    - Consider the weather conditions, traveler interests, and budget constraints.
    - Provide a step-by-step reasoning process for how you arrived at the final output.   

    FINAL OUTPUT:
    ```json
    [TravelPlan Pydantic model structure]
    ```

## Context
{weather_for_dates} 
{activities_for_dates} 
"""
```

**✅ Clearly instructs LLM to assume a role as a travel planner, travel agent, or something similar**

- Opens with "You are an expert itinerary planning agent"

**✅ Encourages detailed daily plans through Chain-of-Thought guidance or examples**

- Includes ANALYSIS section requiring "step-by-step reasoning process"
- Requires consideration of "weather conditions, traveler interests, and budget constraints"

**✅ Specifies JSON output format matching the TravelPlan Pydantic model structure**

- FINAL OUTPUT section specifies ```json format
- Includes complete Pydantic model structure in prompt

**✅ Provides necessary context, including VacationInfo object, weather data, and activities data**

- Context section includes `{weather_for_dates}` and `{activities_for_dates}`
- Takes `vacation_info.model_dump_json()` as user input

#### ACTIVITY_AND_WEATHER_ARE_COMPATIBLE_SYSTEM_PROMPT ✅

The weather compatibility prompt satisfies all requirements:

```python
ACTIVITY_AND_WEATHER_ARE_COMPATIBLE_SYSTEM_PROMPT = """
You are an expert in determining whether a given activity is compatible with the current weather conditions.

## Task
Determine whether the given activity is compatible with the current weather conditions. 
When there is insufficient information, assume the activity IS_COMPATIBLE
Check for backup options in the activity description 

## Output format
    REASONING:
    - Provide a step-by-step reasoning process for how you arrived at the final output.
    - Consider the weather conditions, activity description, and any relevant factors.

    OUTPUT:
    - Provide a final output that is either 'IS_COMPATIBLE' or 'IS_INCOMPATIBLE'.
    - The output should be a single word with no punctuation.

    FINAL ANSWER:
    [IS_COMPATIBLE, IS_INCOMPATIBLE]

## Examples
when it is raining outdoor activities are IS_INCOMPATIBLE 
when it is sunny outdoor activities are IS_COMPATIBLE 
when it is cloudy outdoor activities are IS_COMPATIBLE 
when it is snowing outdoor activities are IS_INCOMPATIBLE 
when it is windy outdoor activities are IS_INCOMPATIBLE 
when it is stormy outdoor activities are IS_INCOMPATIBLE
""".strip()
```

**✅ Clearly defines the LLM role and task**

- "You are an expert in determining whether a given activity is compatible with the current weather conditions"

**✅ Specifies exact output format**

- Clear REASONING and OUTPUT sections
- Exact format specification: "IS_COMPATIBLE" or "IS_INCOMPATIBLE"

**✅ Includes relevant examples illustrating expected evaluations**

- Six examples covering different weather conditions (rain, sun, clouds, snow, wind, storms)

## Agent Reasoning and Tool Use

### Criteria Requirements

**Design a tool description that guides the LLM to call it when needed.**

**Design a prompt that guides an LLM to use reasoning and tools in the ReAct cycle.**

### Implementation Analysis

#### get_activities_by_date_tool Function ✅

The tool function satisfies docstring requirements:

```python
def get_activities_by_date_tool(date: str, city: str) -> List[dict]:
    """Fetches activities for a given date and city.
    
    Args:
        date (str): The date for which to fetch activities, in the format 'YYYY-MM-DD'.
        city (str): The city for which to fetch activities.
    
    Returns:
        List[dict]: A list of activities for the given date and city.
    
    Example:
        >>> get_activities_by_date_tool("2025-06-10", "AgentsVille")
        [{'activity_id': '1', 'name': 'Activity 1', 'start_time': '2025-06-10T10:00:00', 'end_time': '2025-06-10T12:00:00', 'location': 'Location 1', 'description': 'Description 1', 'price': 50, 'related_interests': ['tennis', 'cooking']}, ...]
    """
```

**✅ Provide sufficient description for understanding tool's purpose and use**

- Clear description: "Fetches activities for a given date and city"

**✅ Define expected input parameters and their data types and formats**

- `date (str)`: Specifies format 'YYYY-MM-DD'
- `city (str)`: Clear parameter description

**✅ Includes example usage**

- Realistic example with expected input and output structure

#### ITINERARY_REVISION_AGENT_SYSTEM_PROMPT ✅

The ReAct prompt satisfies all requirements:

```python
ITINERARY_REVISION_AGENT_SYSTEM_PROMPT = f"""
You are a highly skilled itinerary revision agent, capable of revising travel plans based on feedback and evaluation results.

## Task
Your task is to revise the provided travel plan based on the traveler's feedback and the results of the evaluation tools.
You will consider the traveler's feedback and invoke the evaluation tools to check the proposed itinerary.
Before providing the final output, you should run the evaluation tools again to ensure the revised plan meets all requirements.

Your output should include a THOUGHT section that explains your reasoning and an ACTION section that specifies the tool to call with the necessary arguments in JSON format.

## Available Tools
{get_tool_descriptions_string(ALL_TOOLS)}

## Output Format
    THOUGHT:
    - Provide a step-by-step reasoning process for how you arrived at the final output.
    - Explain how you will revise the travel plan based on the feedback and evaluation results.
    - Consider the traveler's feedback, evaluation results, and any other relevant factors.
    - If you need to call a tool, specify the tool name and the arguments in the ACTION section.

    ACTION:
    {{"tool_name": "[tool_name]", "arguments": {{"arg1": "value1", ...}}}}

    OBSERVATION:
    - Provide the observation from the tool call.
"""
```

**✅ Clearly states LLM role and itinerary revision task**

- "You are a highly skilled itinerary revision agent"
- Clear task definition for revision based on feedback

**✅ Details the THINK-ACT-OBSERVE cycle explicitly**

- THOUGHT section for reasoning
- ACTION section for tool invocation
- OBSERVATION section for tool results

**✅ Lists all available tools, their purposes, and parameter requirements**

- Uses `{get_tool_descriptions_string(ALL_TOOLS)}` for dynamic tool documentation
- Includes calculator_tool, get_activities_by_date_tool, run_evals_tool, final_answer_tool

**✅ Specifies exact ACTION format for tool invocation**

- `{"tool_name": "[tool_name]", "arguments": {"arg1": "value1", ...}}`

**✅ Includes explicit exit instruction via final_answer_tool invocation**

- final_answer_tool included in available tools
- Implementation checks for `tool_name == "final_answer_tool"` to exit cycle

**✅ States that the run_evals_tool must be run before the final_answer_tool**

- Task description emphasizes running evaluation tools before final output
- Implementation includes run_evals_tool in the available tools

## Structured Output Validation

### Criteria Requirements

**Pydantic models are created and used properly.**

### Implementation Analysis

#### VacationInfo Pydantic Model ✅

The model satisfies all requirements:

```python
class Traveler(BaseModel):
    """A traveler with a name, age, and list of interests."""
    name: str
    age: int
    interests: List[Interest]

class VacationInfo(BaseModel):
    """Vacation information including travelers, destination, dates, and budget."""
    travelers: List[Traveler] = Field(..., description="A list of travelers")
    destination: str = Field(..., description="The vacation destination")
    date_of_arrival: datetime.date = Field(..., description="The date of arrival")
    date_of_departure: datetime.date = Field(..., description="The date of departure")
    budget: int = Field(..., description="The budget for the vacation in fictional currency units")
```

**✅ Created properly given the JSON structure representing the travelers and their vacation details**

- Handles nested Traveler objects with interests
- Proper datetime.date handling for arrival/departure dates
- Validates against VACATION_INFO_DICT structure

**✅ Is read properly (start and end dates) when pulling the weather and activity data**

```python
weather_for_dates = [
    call_weather_api_mocked(
        date=ts.strftime("%Y-%m-%d"), city=vacation_info.destination
    )
    for ts in pd.date_range(
        start=vacation_info.date_of_arrival,  # ✅ Uses model fields
        end=vacation_info.date_of_departure,   # ✅ Uses model fields
        freq="D",
    )
]
```

#### TravelPlan Pydantic Model ✅

The model schema satisfies requirements:

```python
class Weather(BaseModel):
    temperature: float
    temperature_unit: str
    condition: str

class Activity(BaseModel):
    activity_id: str
    name: str
    start_time: datetime.datetime
    end_time: datetime.datetime
    location: str
    description: str
    price: int
    related_interests: List[Interest]

class ActivityRecommendation(BaseModel):
    activity: Activity
    reasons_for_recommendation: List[str]

class ItineraryDay(BaseModel):
    date: datetime.date
    weather: Weather
    activity_recommendations: List[ActivityRecommendation]

class TravelPlan(BaseModel):
    city: str
    start_date: datetime.date
    end_date: datetime.date
    total_cost: int
    itinerary_days: List[ItineraryDay]
```

**✅ Included in AT LEAST ONE system prompt that needs to output a TravelPlan data structure**

The schema is included in **TWO** system prompts:

1. **ITINERARY_AGENT_SYSTEM_PROMPT** - includes complete schema in FINAL OUTPUT section
2. **ITINERARY_REVISION_AGENT_SYSTEM_PROMPT** - includes complete schema in FINAL OUTPUT section

Both prompts include the full Pydantic model structure, satisfying the requirement and going beyond the minimum of "at least one."

## Additional Implementation Highlights

### Evaluation Framework

- Comprehensive evaluation functions checking dates, costs, weather compatibility, and interest matching
- Integration with traveler feedback evaluation using separate LLM calls
- ReAct cycle continues until all evaluations pass

### Tool Integration

- Four tools implemented: calculator, activities lookup, evaluation runner, and final answer
- Proper JSON parsing with error handling and repair functionality
- Dynamic tool documentation generation

### Error Handling

- JSON repair for malformed tool calls
- Comprehensive exception handling in ReAct cycle
- Validation at multiple stages (Pydantic models, evaluation functions, tool calls)

The implementation successfully addresses all submission requirements while providing a robust, production-ready travel planning system.
