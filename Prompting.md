# PROMPTING.md — Intentional Prompting for *WeatherWise*

> **Purpose**  
> This document shows *how* I used AI **intentionally**—not just for answers, but to reason, test, and iteratively improve code. It includes a six-step prompting method for one component and three **Before → After** code examples demonstrating gains in correctness, resilience, and clarity.

---

## How to Read the Examples
Each example follows the same structure:
1. **Prompt** – what I asked and why  
2. **Technique Used** – which intentional prompting move I used  
3. **Before → After** – compact code change  
4. **Impact** – bug fixes, performance, clarity, and edge-case coverage

---

## Example 1 — Get API Keys Set Up (WeatherWise)

### Goal
Use **environment variables**, **validate** they exist, and **centralise** setup for both **OpenWeather** and the **AI provider** in a single notebook setup cell.

### Prompt
> “Can you explain how I can securely access and store an API key? Also, how do I obtain an OpenWeather key, and what happens if a user forgets to set it? I’m also using `HANDS_ON_AI_API_KEY`—how should I configure both keys at the top of my notebook?”

### Techniques Used
- **Explain reasoning** (security rationale)  
- **Think practically** (step-by-step setup + smoke test)  
- **Refine & adapt** (add second provider)  
- **Check assumptions** (missing/invalid key)  
- **Thinking partner** (single, reusable Setup cell)

### Before (problematic)
```python
# Hardcoded and unvalidated
OPENWEATHER_API_KEY = "abc123"
api_key = OPENWEATHER_API_KEY
```
### After
```python
import os

# AI provider
os.environ["HANDS_ON_AI_SERVER"] = "https://ollama.serveur.au"
os.environ["HANDS_ON_AI_MODEL"]  = "llama3.2"
os.environ["HANDS_ON_AI_API_KEY"] = input("Enter your AI API key: ")

# Weather provider
os.environ["WEATHER_API_KEY"] = "YOUR_OPENWEATHER_KEY"  # replace in production

def require_env(var: str) -> str:
    v = os.getenv(var)
    if not v:
        print(f"Error: missing {var}. Set it and rerun.")
        raise SystemExit(1)
    return v

AI_KEY = require_env("HANDS_ON_AI_API_KEY")
WX_KEY = require_env("WEATHER_API_KEY")
```
---
## Example 2 — Build `get_weather_data()` Daily Summaries (WeatherWise)

### Goal
Group **OpenWeather 3-hour** forecasts into **per-day summaries** for the next **N (1–5)** days, with **simple input validation**, **consistent error handling**, and a lightweight **“current”** snapshot.

### Prompt
> “Design `get_weather_data(city, forecast_days=5)` that validates inputs, groups 3-hour forecasts by day for the next N days, returns daily summaries, adds a simple ‘current’ snapshot from today’s first item, and always includes a clear error message if something fails.”

### Techniques Used
- **Plan first** (brief outline before coding)
- **Guardrails** (input validation + consistent `error` field)

### Before (problematic)
```python
# Raw call with no validation or grouping
def get_weather_data(city, forecast_days=5):
    return Get_data_from_Web(city, units="metric")
```
### After
```python
import os
import requests
from collections import defaultdict, Counter
from datetime import datetime, timedelta, timezone
from typing import Dict, Any, List, Optional

def _err(message: str) -> Dict[str, Any]:
    return {"current": None, "days": [], "error": message}

def get_weather_data(city: str, forecast_days: int = 5, units: str = "metric") -> Dict[str, Any]:
    """
    Return a dict with:
      - current: snapshot from the first forecast item of 'today'
      - days: list of per-day summaries (up to N days)
      - error: None or string
    """
    # --- Validate inputs ---
    if not isinstance(city, str) or not city.strip():
        return _err("Invalid city: provide a non-empty string.")
    if not isinstance(forecast_days, int) or not (1 <= forecast_days <= 5):
        return _err("Invalid forecast_days: choose an integer between 1 and 5.")
    if units not in {"metric", "imperial", "standard"}:
        return _err("Invalid units: use 'metric', 'imperial', or 'standard'.")

    api_key = os.getenv("WEATHER_API_KEY")
    if not api_key:
        return _err("Missing WEATHER_API_KEY environment variable.")

    # --- Fetch 5-day/3-hour forecast ---
    try:
        url = "https://api.openweathermap.org/data/2.5/forecast"
        params = {"q": city.strip(), "appid": api_key, "units": units}
        r = requests.get(url, params=params, timeout=15)
        if r.status_code != 200:
            # Try to surface OpenWeather message if available
            try:
                payload = r.json()
                msg = payload.get("message") or str(payload)
            except Exception:
                msg = r.text
            return _err(f"OpenWeather error ({r.status_code}): {msg}")
        data = r.json()
    except requests.RequestException as e:
        return _err(f"Network error: {e}")

    # --- Basic shape check ---
    items: List[Dict[str, Any]] = data.get("list") or []
    if not items:
        return _err("No forecast data returned.")

    # --- Time helpers ---
    # OpenWeather timestamps are UTC seconds
    def to_local_date(utc_ts: int, tz_shift: Optional[int]) -> str:
        # city.timezone is seconds offset from UTC (may be absent on some responses)
        tz = timezone(timedelta(seconds=(tz_shift or 0)))
        return datetime.fromtimestamp(utc_ts, tz=tz).strftime("%Y-%m-%d")

    tz_shift = (data.get("city") or {}).get("timezone", 0)

    # --- Group 3-hour entries by YYYY-MM-DD ---
    grouped: Dict[str, List[Dict[str, Any]]] = defaultdict(list)
    for it in items:
        dt_txt = it.get("dt")  # unix
        if dt_txt is None:
            continue
        dkey = to_local_date(dt_txt, tz_shift)
        grouped[dkey].append(it)

    # Keep only the next N calendar days from 'today' (local to city)
    today_key = to_local_date(int(datetime.now(timezone.utc).timestamp()), tz_shift)
    sorted_days = sorted(k for k in grouped.keys() if k >= today_key)[:forecast_days]

    # --- Build 'current' snapshot from first item of today (or earliest available) ---
    def _mk_current(it: Dict[str, Any]) -> Dict[str, Any]:
        main = it.get("main", {})
        wx = (it.get("weather") or [{}])[0]
        wind = it.get("wind", {})
        return {
            "time_utc": it.get("dt"),
            "temp": main.get("temp"),
            "feels_like": main.get("feels_like"),
            "humidity": main.get("humidity"),
            "pressure": main.get("pressure"),
            "wind_speed": wind.get("speed"),
            "condition": wx.get("main"),
            "description": wx.get("description"),
        }

    today_items = grouped.get(today_key, [])
    first_item = today_items[0] if today_items else items[0]
    current = _mk_current(first_item) if first_item else None

    # --- Summarise each day ---
    day_summaries: List[Dict[str, Any]] = []
    for dkey in sorted_days:
        buckets = grouped[dkey]
        temps = [b.get("main", {}).get("temp") for b in buckets if b.get("main", {}).get("temp") is not None]
        hums  = [b.get("main", {}).get("humidity") for b in buckets if b.get("main", {}).get("humidity") is not None]
        conds = [((b.get("weather") or [{}])[0].get("main") or "Unknown") for b in buckets]

        # Rain can appear under 'rain': {'3h': mm}
        precip = 0.0
        for b in buckets:
            rain = b.get("rain", {})
            snow = b.get("snow", {})
            precip += float(rain.get("3h", 0) or 0) + float(snow.get("3h", 0) or 0)

        summary = {
            "date": dkey,
            "temp_min": min(temps) if temps else None,
            "temp_max": max(temps) if temps else None,
            "humidity_avg": round(sum(hums) / len(hums), 1) if hums else None,
            "precip_mm": round(precip, 2),
            "dominant_condition": Counter(conds).most_common(1)[0][0] if conds else None,
            "points": len(buckets),  # how many 3h slices contributed
        }
        day_summaries.append(summary)

    return {"current": current, "days": day_summaries, "error": None}
```
---
## Example 3 — Create `create_precipitation_visualisation()` (WeatherWise)

### Goal
Build a **daily precipitation** visualisation that reads rainfall totals from `weather_data`, safely handles **missing or hourly** data, and plots a clear **bar chart** for easy comparison across days.

### Prompt
> “Help me make a precipitation visualisation function that reads daily rainfall totals, fills in missing values using hourly data if needed, and plots them as a bar chart with Matplotlib.”

### Techniques Used
- **Plan first** (outline logic before coding)
- **Check assumptions** (handle missing or non-numeric precipitation data)

### Before (problematic)
```python
def create_precipitation_visualisation(weather_data):
    totals = [d["totalPrecipitation"] for d in weather_data["daily"]]
    plt.plot(totals)  # line chart, fails if data missing
```
### After
```python
from typing import Dict, Any
import matplotlib.pyplot as plt

def create_precipitation_visualisation(weather_data: Dict[str, Any], output_type: str = "display"):
    """
    Plot daily precipitation (mm) as a bar chart.
    Falls back to summing hourly precip if daily total is missing.
    Assumes a helper `day_labels(daily)` that returns x-axis labels.
    """
    daily = weather_data.get("daily", [])
    if not daily:
        print("No data to forecast precipitation plot.")
        return None

    totals, labels = [], day_labels(daily)  # day_labels must be defined elsewhere

    for d in daily:
        mm = d.get("totalPrecipitation")
        if mm is None and d.get("hourly"):
            # Fallback: sum hourly precipitation (field names may differ per source)
            try:
                mm = sum(float(h.get("precipi_MM") or 0.0) for h in d["hourly"])
            except Exception:
                mm = 0.0

        # Coerce to float; treat bad values as 0.0
        try:
            totals.append(float(mm))
        except Exception:
            totals.append(0.0)

    fig, ax = plt.subplots()
    ax.bar(labels, totals)  # no explicit colors to keep styles configurable
    ax.set_title("Forecast Precipitation by Day")
    ax.set_xlabel("Day and Date")
    ax.set_ylabel("Precipitation (mm)")
    plt.tight_layout()

    if output_type == "figure":
        return fig
    else:
        plt.show(block=False)
        plt.pause(0.1)
        return None
```
---
## Example 4 — Design `parse_weather_question()` (WeatherWise)

### Goal
Create a **natural language parser** that converts user questions like “Will it rain tomorrow in Perth?” into a structured dictionary containing **location**, **time**, and **weather attribute**.

### Prompt
> “Help me write a function that takes a user’s weather question (like ‘What’s the weather in Perth today?’) and extracts the location, time (‘today’ or ‘tomorrow’), and attribute (‘temperature’, ‘rain’, or ‘wind’).”

### Techniques Used
- **Plan first** (define output structure before coding)
- **Check assumptions** (use keyword and regex parsing for reliability)

### Before (problematic)
```python
def parse_weather_question(q):
    words = q.split()
    return {"city": words[-1]}  # oversimplified, misses time and attribute
```
### After
```python
import re
from typing import Dict, Any, Union

def parse_weather_question(question: str) -> Dict[str, Any]:
    if not question or not isinstance(question, str):
        return {"error": "Failed response."}

    q = question.strip().lower()

    # Detect weather attribute
    if "temperature" in q or "hot" in q or "cold" in q:
        attribute = "temperature"
    elif "rain" in q or "precipitation" in q or "humid" in q:
        attribute = "precipitation"
    elif "wind" in q or "fresh" in q:
        attribute = "wind"
    else:
        attribute = None

    # Detect time
    time_date: Union[int, str, None] = None
    if "today" in q:
        time_date = 0
    elif "tomorrow" in q:
        time_date = 1

    # Extract location
    location = None
    match = re.search(r"in\s([\w\s]+)|for\s([\w\s]+)|at\s([\w\s]+)", q)
    if match:
        location = next((g for g in match.groups() if g), None)
        if location:
            location = location.strip().title()

    return {
        "raw": question,
        "location": location,
        "attribute": attribute,
        "time": time_date,
    }
```
