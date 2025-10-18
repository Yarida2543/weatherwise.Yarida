# 🌦️ WeatherWise Project Overview

Welcome to the 'WeatherWise by Yarida' This smart weather advisor application that combines **real weather data**, **visualisations**, and **AI-powered analysis**. This notebook helps you build, test, and explain your project step by step — from fetching live data to generating natural language weather reports. 🤖📊  

![Build With AI](https://img.shields.io/badge/Built_with-AI-blueviolet?logo=openai)
![Python](https://img.shields.io/badge/Made_with-Python-3776AB?logo=python)
![Visualisation](https://img.shields.io/badge/Includes-Visualisations-orange?logo=plotly)

---
## 📘 Project Overview
**WeatherWise** demonstrates how Python, APIs, and AI can work together in one notebook.  
You’ll learn to:
- Retrieve real-time weather data from the **OpenWeather API**
- Process and clean data using Python functions
- Visualise daily forecasts with **Matplotlib**
- Parse natural language questions using a simple NLP function
- Generate AI-driven weather answers using the **Hands-On AI API**

## ✨ What It Does
- Fetches **current conditions** and a **5-day forecast** for any city.
- Creates **readable charts** for temperature and precipitation.
- Answers natural questions like *“Will it rain tomorrow in Perth?”* using AI.
- Provides a **menu-driven interface** that’s easy to run in a notebook.

## 🧰 Features

| Category | Description |
| 🧰 Setup and Configuration|Imports packages and sets up API connections|
| 🌤️ Weather Data | Fetches 5-day forecasts and current conditions via OpenWeather |
| 📊 Visualisation | Line chart for temperature trends and bar chart for precipitation |
| 🤖 AI Interaction | Converts your questions (e.g. *“Will it rain tomorrow in Perth?”*) into intelligent answers |
| 🧭 User Interface | Console menu built with `pyinputplus` for smooth navigation |
| 🧪 Testing | Includes test cases for data retrieval and forecast processing |
| 🚀 Run Program | Starts the interactive WeatherWise experience |

## 🧠 How It Works (At a Glance)
1. **Fetch**: call OpenWeather → receive 3-hourly forecasts  
2. **Process**: summarise into daily highs/lows & rainfall  
3. **Visualise**: render charts with Matplotlib  
4. **Understand**: parse user questions (attribute, time, place)  
5. **Answer**: combine parsed intent + forecast → AI reply

🎮 Usage
After setting up and launching the notebook, you can interact with the **WeatherWise** console-based user interface.
When prompted, follow these steps:
1. **Enter a location**  
   Type the name of the city you want to get weather information for (e.g., `Perth` or `London`).
2. **Choose an option from the menu**  
   Select one of the available options displayed in the console:
   - 🌤️ **Get Weather Today** — Shows the current or latest weather forecast.  
   - 🌡️ **Create Temperature Visualisation** — Displays a line chart of max/min temperatures.  
   - 🌧️ **Create Precipitation Visualisation** — Displays a bar chart showing daily rainfall totals.  
   - 🤖 **Ask an AI Question** — Type a natural language question such as:  
     *“Will it rain tomorrow in Sydney?”*  
     The AI will analyze your question and respond based on real forecast data.  
   - 🚪 **Quit the Program** — Exit the console safely when finished.
3. **View the Results**  
   - Data visualisations open directly within your notebook.  
   - AI-generated responses appear in the console output.

   
## 🚀 Getting Started
Follow these steps to run **WeatherWise** locally or in **Jupyter Notebook**.

## 🛠️ Setup and Installation

# Configure API keys
os.environ["OPENWEATHER_API_KEY"] = "your_openweather_api_key"
os.environ["HANDS_ON_AI_SERVER"] = "https://ollama.serveur.au"
os.environ["HANDS_ON_AI_MODEL"] = "llama3.2"
os.environ["HANDS_ON_AI_API_KEY"] = "your_ai_api_key"

# 📦 Setup and Configuration

Import required packages and set up your environment before running the notebook.
Install dependencies
```bash
pip install requests matplotlib pyinputplus fetch-my-weather hands-on-ai

## 🌤️ Weather Data Functions 
These helpers fetch and summarise forecasts from **OpenWeather** for easy use in charts and AI replies.

**What’s included**
- `Get_data_from_Web(city, units="metric")` → raw 5-day / 3-hour forecast JSON.
- `_process_daily_forecast_items(day_items, date_str)` → one-day summary (max/min temp, total rain, description).
- `get_weather_data(city, forecast_days=5)` → structured result

## 📊 Visualisation Functions 
Create clear charts from processed forecast data to help users spot trends quickly.

**What’s included**
- `day_labels(daily)` → weekday/date labels for the x-axis.
- `create_temperature_visualisation(weather_data, output_type="display")`
  - Line chart of **max** and **min** temperatures (per day).
- `create_precipitation_visualisation(weather_data, output_type="display")`
  - Bar chart of **total daily rainfall (mm)**.

## 🤖 Natural Language Processing 
Translate plain-English weather questions into structured intent the app can use.

**What it does**
- Extracts **attribute** (e.g., `temperature`, `precipitation`, `wind`, `condition`)
- Detects **time** (e.g., `today` → `0`, `tomorrow` → `1`, `next 3 days` → `3`)
- Finds **location** (e.g., phrases like “in Perth”, “for Melbourne”, “at Sydney”)
- Returns a small dict your app can pass to the AI + forecast logic

## 🧭 User Interface 
A simple **console menu** (using `pyinputplus`) lets users run WeatherWise without editing code.
**Main function**
- `run_console_menu()`

**What it does**
1. Prompts for a **city/location** (e.g., *Perth*).
2. Shows an **interactive menu**:
   - 🌤️ Get weather today
   - 🌡️ Create Temperature Visualisation
   - 🌧️ Create Precipitation Visualisation
   - 🤖 Ask any question with AI
   - 🚪 Quit program
3. Displays results:
   - Charts render inline in the notebook
   - AI replies print in the console

🧩 Main Application Logic 

Connects user intent, forecast data, and AI to produce a clear answer.

**Main function**
- `generate_weather_response(parsed_question: Dict[str, Any], weather_data: Dict[str, Any]) -> str`

**What it does**
1. Validates inputs (checks for parsing or data errors).
2. Builds a concise **weather context** (current + daily summaries).
3. Combines that context with the **original user question**.
4. Calls the AI (`get_response(...)`) to produce a natural-language repl

🧪 Testing and Examples

Quick checks to validate data fetching, processing, charts, NLP parsing, and AI replies.

##1) API Fetch (OpenWeather)
```python
print("Real city:");  print(Get_data_from_Web("London"))
print("Invalid city:"); print(Get_data_from_Web("InvalidFakeCity123"))
print("Imperial units:"); print(Get_data_from_Web("New York", units="imperial"))

## 🚀 Run Program

Start the interactive **WeatherWise** console from your notebook’s final cell:

```python
run_console_menu()



Enjoy with Weatherwise 💡🌤️
