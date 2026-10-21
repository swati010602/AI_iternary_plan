# Architecture & Technical Solution Design
## Project: AI-Powered Personalized Travel Itinerary Planner

---

## 1. System Overview & Dual-Stack Architecture

The system uses a decoupled client-server architecture combining a **Python FastAPI backend** with a **TypeScript React frontend**.

```
┌────────────────────────────────────────────────────────┐
│               Frontend (TypeScript / React)            │
│  - User Input Form (City, Dates, Budget, Travelers)    │
│  - Loading State with Travel Advice Tips               │
│  - Interactive Day-by-Day Itinerary Cards              │
│  - Transparent Budget Breakdown & Splurge Suggestions  │
└───────────────────────────┬────────────────────────────┘
                            │ HTTP POST /api/itinerary/generate
                            ▼
┌────────────────────────────────────────────────────────┐
│               Backend (Python / FastAPI)               │
│                                                        │
│  1. Input Validator (Pydantic Request Schema)          │
│     └── Fast fail on invalid dates, negative budgets   │
│                                                        │
│  2. Gemini AI Service (google-genai SDK)               │
│     └── Structured Output via Pydantic response schema │
│                                                        │
│  3. Deterministic Validation Engine (validator.py)     │
│     ├── Math Check: Itemized costs == Total Cost       │
│     ├── Budget Check: Total Cost <= User Budget        │
│     ├── Completeness: Exactly N days with Time Blocks  │
│     └── Geographic Clustering: Neighborhood tags       │
│                                                        │
│  4. Self-Correction Loop (Approach C)                  │
│     ├── Retry 1: Feedback loop with exact error delta  │
│     └── Fallback: Graceful degradation + warning badge │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│           Automated Test Suite (pytest)                │
│  - Unit tests for deterministic validation rules       │
│  - Self-correction trigger & recovery tests            │
│  - API integration tests (valid, edge-case, invalid)   │
└────────────────────────────────────────────────────────┘
```

---

## 2. Technology Choices & Rationale

| Component | Technology | Rationale |
| :--- | :--- | :--- |
| **Backend Framework** | **Python / FastAPI** | High-performance asynchronous API framework with native OpenAPI documentation and first-class Pydantic integration. |
| **Data Validation** | **Pydantic v2** | Gold standard for strict type enforcement, numerical constraints, and LLM structured outputs. |
| **AI Provider** | **Google Gemini (`gemini-1.5-flash` / `gemini-2.0-flash`)** | High speed, generous free tier via Google AI Studio, native JSON schema enforcement. |
| **Testing Framework** | **Pytest** | Standard in AI QE and Python engineering; tests deterministic business logic, math invariants, and mock LLM pipelines. |
| **Frontend Framework** | **TypeScript / React (Vite)** | Type-safe, modern web UI. Provides the target for future **Project 3 (Playwright UI Automation)**. |

---

## 3. Data Contracts (Schemas)

### 3.1 Request Schema (`ItineraryRequest`)
```python
from pydantic import BaseModel, Field
from datetime import date
from typing import List, Literal

class ItineraryRequest(BaseModel):
    destination: str = Field(..., min_length=2, description="Target city and country")
    start_date: date = Field(..., description="Trip start date (YYYY-MM-DD)")
    end_date: date = Field(..., description="Trip end date (YYYY-MM-DD)")
    total_budget: float = Field(..., gt=0, description="Total budget amount")
    currency: str = Field(default="USD", min_length=3, max_length=3, description="Currency code (e.g. USD, EUR, INR)")
    traveler_count: int = Field(default=1, ge=1, le=6, description="Number of travelers (1-6)")
    travel_pace: Literal["relaxed", "active"] = Field(default="relaxed", description="Pacing style")
    interests: List[str] = Field(default=["general"], description="List of traveler interest tags")
```

### 3.2 Response Schema (`ItineraryResponse`)
```python
class ActivityItem(BaseModel):
    time_block: Literal["morning", "afternoon", "evening"]
    name: str
    why_recommended: str
    estimated_cost: float
    estimated_duration_hours: float

class DayPlan(BaseModel):
    day_number: int
    date: str
    neighborhood_theme: str
    activities: List[ActivityItem]

class BudgetBreakdown(BaseModel):
    accommodation_cost: float  # (nights * nightly_rate)
    food_dining_cost: float
    activities_cost: float
    local_transit_cost: float
    total_calculated_cost: float

class SplurgeItem(BaseModel):
    name: str
    why_worth_it: str
    estimated_extra_cost: float

class TripSummary(BaseModel):
    destination: str
    duration_days: int
    duration_nights: int
    total_estimated_cost: float
    remaining_budget: float
    budget_status: Literal["within_budget", "budget_exceeded"]
    warnings: List[str] = []

class ItineraryResponse(BaseModel):
    trip_summary: TripSummary
    days: List[DayPlan]
    budget_breakdown: BudgetBreakdown
    worth_the_splurge: List[SplurgeItem]
```

---

## 4. Deterministic Validation Engine & Self-Correction (Approach C)

### 4.1 Invariant Rules Checked
1. **Math Integrity Check:**
   $$\text{total\_calculated\_cost} == \text{accommodation} + \text{food} + \text{activities} + \text{transit}$$
2. **Budget Compliance Check:**
   $$\text{total\_calculated\_cost} \le \text{total\_budget}$$
3. **Duration Invariant:**
   $$\text{len}(\text{days}) == (\text{end\_date} - \text{start\_date}).\text{days} + 1$$
4. **Time-Block Completeness:**
   Every day must contain non-empty `morning`, `afternoon`, and `evening` activities.
5. **Geographic Clustering:**
   Every day must define a non-empty `neighborhood_theme`.

### 4.2 Self-Correction Loop
If validation catches an arithmetic or budget failure:
* **Retry 1 (Self-Correction):** The validator constructs a targeted error prompt:
  > *"Validation Failure: The generated itinerary total is $680, which exceeds the user budget of $600 by $80. Replace high-cost activities with free or budget-friendly alternatives to bring the total under $600."*
* **Fallback (Graceful Degradation):** If the re-prompted plan still exceeds budget after 1 retry:
  - The plan is returned to the user.
  - `budget_status` is set to `"budget_exceeded"`.
  - Specific warning messages are attached to `trip_summary.warnings` for transparent display in the UI.

---

## 5. Project Directory Structure

```text
AI_iternary_plan/
├── docs/
│   ├── PRD.md                  # Approved Requirements Document
│   ├── DECISION_LOG.md         # Running Architecture & Product Decisions
│   └── ARCHITECTURE.md         # Solution Architecture & Schema Design
│
├── backend/                    # Python / FastAPI
│   ├── app/
│   │   ├── api/
│   │   │   └── routes.py       # API endpoints (/itinerary/generate)
│   │   ├── models/
│   │   │   └── schemas.py      # Pydantic schemas (Request & Response)
│   │   ├── services/
│   │   │   ├── gemini.py       # Gemini API client & prompt templates
│   │   │   └── validator.py    # Deterministic validation & repair loop
│   │   └── config.py           # Configuration & environment variables
│   ├── tests/                  # Pytest automated test suite
│   │   ├── test_validator.py   # Unit tests for deterministic validation rules
│   │   └── test_api.py         # End-to-end integration tests
│   ├── requirements.txt        # Python backend dependencies
│   └── main.py                 # FastAPI application entrypoint
│
├── frontend/                   # TypeScript / React / Vite
│   ├── src/
│   │   ├── components/         # Form, Itinerary Cards, Budget Summary
│   │   ├── types/              # TypeScript types mirroring Pydantic models
│   │   └── App.tsx             # Main application layout
│   └── package.json
│
├── .gitignore                  # Git hygiene (ignoring .env, node_modules, pycache)
└── README.md                   # Project overview & quickstart
```
