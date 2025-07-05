# 🧠🌍 Agentic AI Travel Planner

## 📝 Problem Statement

Planning a trip manually is often time-consuming, fragmented, and inefficient. Travelers typically navigate through multiple websites and apps to gather weather information, explore attractions, calculate budgets, book accommodations, and create an itinerary.

This project aims to solve that by developing an **AI Travel Planner** using **Agentic AI** principles. It acts as an intelligent, modular assistant that autonomously uses various tools to gather real-time data, perform reasoning, and generate a travel plan tailored to the user's preferences.

---

## 🎯 Project Objectives

The AI Travel Planner is designed to:

- Fetch real-time **weather information** of selected destinations
- Provide details about **popular attractions and activities**
- Estimate **hotel/accommodation costs**
- Convert currencies using **real-time exchange rates**
- Calculate **total trip expenses**
- Generate a **day-wise itinerary**
- Summarize all information into a **travel document**
- Operate via a **modular, industry-grade structure**
- Run through an intuitive **Streamlit interface**

Agentic AI enables the system to orchestrate tool calls, maintain memory, and reason through steps autonomously, imitating the workflow of a human travel planner.

---

## 🗂️ Project Structure

The project follows a clean, modular architecture and uses the `uv` package to manage dependencies and execution environments.

```bash
AI-Trip-Planner/
│
├── agent/
│   └── agentic_workflow.py           # Defines the autonomous multi-step reasoning workflow
│
├── config/
│   └── config.yaml                   # Configuration file with llm parameters
│
├── exception/
│   └── exception_handler.py          # Centralized error handling
│
├── logger/
│   └── logger.py                     # Logger for tracking events and debugging
│
├── prompt_library/
│   └── prompt.py                     # Predefined prompts for LLM interactions
│
├── tools/
│   ├── arithmetic_op_tool.py         # Tool for basic arithmetic operations
│   ├── currency_conversion_tool.py   # Tool to fetch real-time currency exchange rates
│   ├── expense_calculator_tool.py    # Tool to calculate total expenses
│   ├── place_search_tool.py          # Tool to fetch activities/attractions
│   └── weather_info_tool.py          # Tool to retrieve weather data
│
├── utils/
│   ├── config_loader.py              # Loads configuration settings
│   ├── currency_converter.py         # Utility for currency conversions
│   ├── expense_calculator.py         # Utility to estimate trip costs
│   ├── model_loader.py               # Loads any LLM models or APIs
│   ├── place_info_search.py          # Uses APIs to get attraction info
│   ├── save_to_document.py           # Saves itinerary and summary to PDF/Doc
│   └── weather_info.py               # Queries weather APIs and parses data
│
├── .env                              # Stores environment variables (API keys)
├── main.py                           # Entry point for the backend pipeline
├── requirements.txt                  # Project dependencies
├── setup.py                          # Setup script for packaging
└── streamlit_app.py                  # Streamlit-based UI interface for interaction
```
---

## 📁 Code Explanation

### `agent/agentic_workflow.py`

This module defines the **Agentic Workflow Graph**, which forms the core of the AI Travel Planner. It creates a tool-augmented agent using LangGraph and LangChain that can autonomously reason, call tools, and answer user queries through multiple steps.

#### 🔧 Class: `GraphBuilder`

The `GraphBuilder` class is responsible for initializing the tools, loading the LLM, and building a LangGraph-based agentic pipeline.

##### `__init__(self, model_provider: str = "groq")`

- Loads a language model using `ModelLoader` based on the chosen provider (e.g., `groq`, OpenAI).
- Initializes and aggregates tools for:
  - **Weather information** (`WeatherInfoTool`)
  - **Attraction/place search** (`PlaceSearchTool`)
  - **Expense calculation** (`CalculatorTool`)
  - **Currency conversion** (`CurrencyConverterTool`)
- Binds all tools to the LLM using `.bind_tools()` so it can invoke them autonomously.
- Stores the system prompt from `prompt_library.prompt`.

##### `agent_function(self, state: MessagesState)`

- Acts as the **thinking function** of the agent.
- Takes the current `state["messages"]`, prepends the system prompt, and invokes the bound LLM.
- Returns the new message (LLM response or tool invocation) in the expected format for the next graph step.

##### `build_graph(self)`

This method builds the LangGraph using `StateGraph(MessagesState)`:

1. Adds two main nodes:
   - `"agent"` → runs the main reasoning function
   - `"tools"` → handles tool calls (via `ToolNode`)
2. Creates a loop:
   - Start → agent
   - If a tool is needed, go to `tools`, then back to `agent`
   - Once reasoning is complete, move to `END`

This structure enables **multi-step reasoning and tool execution** in a loop-like workflow.

##### `__call__(self)`

- Allows the class to be directly called like a function.
- Returns the compiled LangGraph graph.

![image](my_graph.png)

### 🧠 Summary

- This file constructs the **central reasoning engine** using Agentic AI principles.
- It enables the planner to:
  - Think
  - Decide if a tool is needed
  - Call the tool
  - Think again
  - Exit when done
- It's built using **LangGraph** to manage agent-tool interaction in a stateful, modular way.

---

### `config/config.yaml`

This is the main configuration file used to define which LLMs (Large Language Models) are available to the system.

It supports multiple model providers and allows dynamic selection based on user preference or runtime settings. This abstraction makes the system **flexible**, **scalable**, and **easy to extend** with additional providers or models.

#### 🛠️ Structure Overview

```yaml
llm:
  openai:
    provider: "openai"
    model_name: "o4-mini"
  groq:
    provider: "groq"
    model_name: "deepseek-r1-distill-llama-70b"
```

---

### 🧰 `tools/` — AI-Callable Utility Tools

This folder contains modular tool wrappers that can be invoked by the agent to perform specific tasks. These tools are bound to the LLM and used dynamically during the agentic reasoning cycle. Each tool is decorated with `@tool` from LangChain to make it usable in the agent's graph.

---

### `tools/arithmetic_op_tool.py`

This file defines simple arithmetic and financial tools:

- **`multiply(a: int, b: int)`** – Returns the product of two integers.
- **`add(a: int, b: int)`** – Returns the sum of two integers.
- **`currency_converter(from_curr, to_curr, value)`** – Converts currency using AlphaVantage API (used in earlier prototypes or fallback use-cases).

Environment variables are managed with `dotenv` for API key handling.

---

### `tools/currency_conversion_tool.py`

This is a more refined and extensible currency conversion tool:

- Uses a utility class `CurrencyConverter` (from `utils/`) to interface with a real-time currency exchange API.
- Dynamically loads the API key from `.env`.
- **Tool:** `convert_currency(amount, from_currency, to_currency)` – Converts the specified amount from one currency to another.

This tool is more production-ready than the one in `arithmetic_op_tool.py`.

---

### `tools/expense_calculator_tool.py`

A set of tools to calculate travel-related expenses using internal utility logic:

- **`estimate_total_hotel_cost(price_per_night, total_days)`** – Multiplies nightly rate with total days to estimate stay cost.
- **`calculate_total_expense(*costs)`** – Adds all major travel costs (transport, food, stay, activities).
- **`calculate_daily_expense_budget(total_cost, days)`** – Divides total budget across the trip duration.

These tools wrap reusable functions from `utils/expense_calculator.py`.

---

### `tools/place_search_tool.py`

Fetches local place-related info using either the **Google Places API** or **Tavily Search** as a fallback.

Tools provided:

- **`search_attractions(place)`** – Lists popular tourist spots.
- **`search_restaurants(place)`** – Lists recommended restaurants.
- **`search_activities(place)`** – Suggests activities and events.
- **`search_transportation(place)`** – Describes local transport modes.

Each function:
- Tries using Google Places API
- Falls back to Tavily in case of failure
- Provides a detailed fallback message if primary source fails

Uses `dotenv` for API key injection.

---

### `tools/weather_info_tool.py`

Fetches weather data using the **OpenWeatherMap API**:

- **`get_current_weather(city)`** – Returns temperature and weather condition for a city.
- **`get_weather_forecast(city)`** – Returns multi-day weather forecast (date, temperature, description).

Backed by `WeatherForecastTool` from the `utils/` layer and loaded with API keys from `.env`.

---

### 🧠 Summary

Each tool module encapsulates a specific functional responsibility:

| Tool File | Purpose |
|-----------|---------|
| `arithmetic_op_tool.py` | Basic math + experimental currency API |
| `currency_conversion_tool.py` | Modern currency conversion |
| `expense_calculator_tool.py` | Hotel, total, and daily budget calculations |
| `place_search_tool.py` | Attractions, food, activities, and transport |
| `weather_info_tool.py` | Real-time and forecasted weather data |

All tools are plug-and-play and registered in the agent graph for autonomous use via LLM instructions.

---

### 🧩 `utils/` — Core Utility Modules

The `utils/` folder contains helper components that abstract the logic for configuration, API integration, model loading, data retrieval, and computations. These are low-level services used by the `tools/` and `agent/` layers to function properly.

---

### `utils/config_loader.py`

- **Purpose:** Loads the YAML config file containing model provider and name info.
- **Function:** `load_config()` – Reads and returns configuration from `config/config.yaml`.

This keeps all model configuration centralized and easy to modify.

---

### `utils/currency_converter.py`

- **Class:** `CurrencyConverter`
- **Purpose:** Interacts with the **ExchangeRate API** to convert currency.
- **Function:** `convert(amount, from_currency, to_currency)` – Fetches exchange rates and returns the converted amount.

This class is used by the `CurrencyConverterTool` to power currency conversion requests.

---

### `utils/expense_calculator.py`

- **Class:** `Calculator`
- **Methods:**
  - `multiply(a, b)` – Multiplies two values (e.g., cost × days)
  - `calculate_total(*x)` – Sums a list of costs
  - `calculate_daily_budget(total, days)` – Divides total cost over days to get per-day budget

Used by `expense_calculator_tool.py` to compute all cost-related values.

---

### `utils/model_loader.py`

Handles the loading of different LLMs based on the config and environment:

- **Class:** `ModelLoader`
- **Uses:** 
  - `ChatGroq` for Groq-hosted LLMs
  - `ChatOpenAI` for OpenAI models
- **Process:**
  - Loads the config from `config.yaml`
  - Fetches API keys from `.env`
  - Initializes the correct LLM with the right provider/model

This module is critical for **plug-and-play LLM integration**.

---

### `utils/place_info_search.py`

Provides two classes to retrieve place-related information:

#### 🔹 `GooglePlaceSearchTool`
- Uses `langchain_google_community` to query Google Places API.
- Functions:
  - `google_search_attractions(place)`
  - `google_search_restaurants(place)`
  - `google_search_activity(place)`
  - `google_search_transportation(place)`

#### 🔹 `TavilyPlaceSearchTool`
- Uses `langchain_tavily` to perform web search as a fallback.
- Functions:
  - `tavily_search_attractions(place)`
  - `tavily_search_restaurants(place)`
  - `tavily_search_activity(place)`
  - `tavily_search_transportation(place)`

Both are used in `place_search_tool.py` to provide reliable and robust search capabilities.

---

### `utils/save_to_document.py`

- **Function:** `save_document(response_text, directory="./output")`
- **Purpose:** Saves the generated travel plan as a **Markdown file**.
- **Features:**
  - Adds headers like generation timestamp and author name
  - Auto-generates filename based on timestamp
  - Saves in UTF-8 format in the `output/` directory

Provides export functionality for final trip summaries or itineraries.

---

### `utils/weather_info.py`

- **Class:** `WeatherForecastTool`
- **Purpose:** Interacts with the **OpenWeatherMap API**.
- **Functions:**
  - `get_current_weather(place)` – Gets live weather info
  - `get_forecast_weather(place)` – Retrieves a 5-day forecast

Used by `weather_info_tool.py` to provide real-time weather data for trip planning.

---

### 🧠 Summary

| File | Role |
|------|------|
| `config_loader.py` | Loads YAML configuration for models |
| `currency_converter.py` | Converts currency via API |
| `expense_calculator.py` | Computes total and daily expenses |
| `model_loader.py` | Loads and initializes the appropriate LLM |
| `place_info_search.py` | Searches attractions using Google and Tavily |
| `save_to_document.py` | Exports trip plans as markdown files |
| `weather_info.py` | Retrieves current and forecasted weather |

These modules are **reusable, extendable**, and designed to decouple logic from the tool/agent layers.

---

### 🚀 `main.py` — Backend API using FastAPI

This script runs a **FastAPI backend** that serves as the API interface to the Agentic Travel Planner. It exposes an endpoint (`/query`) to receive user questions and returns the AI-generated travel plan or response.

---

#### 🔧 Tech Stack & Imports

- **`FastAPI`**: For building RESTful endpoints
- **`GraphBuilder`**: Agentic AI workflow loaded from `agentic_workflow.py`
- **`save_document()`**: Optional utility to persist AI responses as markdown (currently unused in this file)
- **`CORS Middleware`**: Allows frontend (e.g., Streamlit, React) to talk to backend without cross-origin issues
- **`.env loader`**: Loads environment variables like API keys
- **`Pydantic`**: Defines the input schema

---

#### 🌐 Endpoint: `/query` (POST)

```python
@app.post("/query")
async def query_travel_agent(query: QueryRequest):
```

---

### ⚙️ `setup.py` — Project Installer Script

This file defines the **installation and packaging logic** for the AI Travel Planner project using Python’s `setuptools`.

It allows you to install the entire project as a package in a virtual environment using:

```bash
pip install -e .
```
```
setup(
    name="AI-TRAVEL-PLANNER",
    version="0.0.1",
    author="Aniket Malpure",
    author_email="aniketmalpure@ufl.edu",
    packages=find_packages(),
    install_requires=get_requirements()
)
```

---

