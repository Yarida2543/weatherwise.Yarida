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
