# PROMPTING.md — Intentional Prompting for WeatherWise

This document explains **how I used AI deliberately** to design, debug, and refine the WeatherWise project. It is not just a chat log—it shows **why** I asked each question, **what improved**, and **how the code evolved**. You’ll find  a six-step prompting method applied to one component, and **three before/after code examples** that demonstrate measurable gains in correctness, resilience, and clarity.

## How the examples are formatted
Each example shows:
- **Prompt** (what I asked and why),
- **Technique used** (e.g., check assumptions),
- **Before → After** code (short diff),
- **Impact** (bug fixed, performance, clarity, edge-case coverage).

# TOPIC: Get API Keys Set Up (WeatherWise)

## What I want
Use environment variables, validate they exist, and centralise setup for both OpenWeather and the AI provider in one notebook cell.

## Prompt
“Can you explain how I can securely access and store an API key? Also, how do I obtain an OpenWeather key, and what happens if a user forgets to set it? I’m also using HANDS_ON_AI_API_KEY—how should I configure both keys at the top of my notebook?”

## Technique prompt I use
- Explain reasoning  
- Think practically (step-by-step)  
- Refine & adapt (add second provider)  
- Check assumptions (missing key)  
- Thinking partner (single Setup cell)

## Before code
```python
# Hardcoded and unvalidated
OPENWEATHER_API_KEY = "abc123"
api_key = OPENWEATHER_API_KEY
```
