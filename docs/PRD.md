# Product Requirements Document (PRD)
## Project: AI-Powered Personalized Travel Itinerary Planner (MVP)

---

### 1. Product Overview & Goal
The **AI-Powered Travel Itinerary Planner** is a focused web application designed to help travelers plan realistic, stress-free, 2 to 5-day city trips.

Instead of generating generic, mathematically broken, or geographically scattered itineraries like general-purpose chatbots often do, this product combines **personalized AI suggestions** with **deterministic quality validation** to guarantee that every plan is budget-aware, geographically sensible, and practically feasible.

---

### 2. The User Problem
When travelers plan a short city vacation, they face three major pain points:
1. **Broken Budgets:** AI tools frequently underestimate costs or make arithmetic errors, suggesting itineraries that blow past what the traveler can afford.
2. **Exhausting Schedules:** General AI tools often recommend places on opposite sides of a city on the same day, forcing travelers to spend their holiday sitting in traffic rather than sightseeing.
3. **Lack of Confidence:** Travelers don't know where AI numbers come from or if recommendations are realistic, forcing them back into hours of manual research.

---

### 3. Target Persona
* **The Practical City Explorer:** A single traveler or group coordinator planning a 2 to 5-day vacation for themselves or their party (solo, couple, friends, or family), seeking a dependable, budget-conscious schedule without the hassle of hours of manual research.

---

### 4. User Inputs (The Request)
The traveler provides the following inputs through a clean, low-friction interface:

| Input Field | Type | Description / Validation |
| :--- | :--- | :--- |
| **Destination City** | Text | Single destination city (e.g., "Barcelona, Spain"). |
| **Start & End Dates** | Date Picker | Determines calendar dates, day count (2–5 days), and night count (`nights = days - 1`). |
| **Total Budget & Currency** | Number + Currency | Total amount the traveler wants to spend (e.g., $800 USD). |
| **Group Size** | Selection | Number and type of travelers (e.g., "1 Solo", "2 Adults", "Family of 3"). |
| **Travel Pace** | Selection | **Relaxed** (2–3 spots/day, slower pace) vs. **Active** (packed schedule, more sights). |
| **Interests & Vibe** | Multi-select / Text | Preferred themes (e.g., Food & Dining, Historical Sites, Art, Nature/Parks, Nightlife). |

---

### 5. Output Experience (The Itinerary)
The generated response presents a structured, human-friendly itinerary:

1. **Trip Overview:**
   * City, dates, total days/nights, and pace.
   * **Budget Status Banner:** Clear indicator showing if the plan is *Within Budget* or if an alert applies.
2. **Day-by-Day Itinerary:**
   * **Daily Geographic Theme:** Identifies the neighborhood/district (e.g., *"Day 1: Gothic Quarter & El Born"*), ensuring activities are clustered close together.
   * **Time Blocks:** Morning, Afternoon, and Evening activities.
   * **Activity Cards:** Each card displays Activity Name, Why it was picked, Estimated Cost, and Estimated Duration.
3. **Transparent Budget Breakdown:**
   * Itemized cost categories: **Accommodation** (Nights $\times$ Nightly Estimate) + **Food & Dining** + **Activities & Admissions** + **Local Transit**.
   * Total Estimated Cost vs. User Budget, including any remaining leftover savings.
4. **"Worth the Splurge" Section:**
   * 2 to 3 curated, popular activities in that city that exceed the current budget.
   * Clearly displays estimated extra cost and why it's worth considering if the traveler decides to expand their budget.

---

### 6. Quality & Deterministic Validation Rules
Before displaying an itinerary to the user, the application runs deterministic validation checks:

* **Date & Duration Check:** Verifies `End Date > Start Date` and duration is strictly between 2 and 5 days.
* **Structural Completeness:** Ensures every single day has Morning, Afternoon, and Evening entries with non-empty descriptions.
* **Geographic Clustering:** Verifies that each day has an assigned neighborhood/zone tag.
* **Math Integrity Check:** Deterministically verifies that:
  $$\text{Total Cost} = \text{Accommodation} + \text{Food} + \text{Activities} + \text{Transit}$$
* **Budget Conformance Check:** Checks if $\text{Total Cost} \le \text{User Budget}$.

---

### 7. Failure Handling & Self-Correction (Approach C)
If an itinerary fails the budget check ($\text{Total Cost} > \text{User Budget}$):
1. **Automated Self-Correction (Retry 1):** The system passes the validation error back to the AI model behind the scenes (*"Plan is over budget by $X. Swap expensive activities for free/budget alternatives"*).
2. **Graceful Degradation (Fallback):** If the plan still exceeds budget after 1 retry, the application displays the plan with a prominent **Transparent Warning Badge** detailing where the budget was exceeded, avoiding long loading times or system crashes.

---

### 8. MVP Boundaries

| In-Scope (MVP) | Out-of-Scope (Future Enhancements) |
| :--- | :--- |
| Single-city itineraries (2–5 days) | Multi-city flight and train routing |
| Single-planner input flow | Multi-user collaborative voting/invites |
| Deterministic budget & schema validation | Live booking, ticketing, or payment processing |
| Self-correction retry loop (max 1 retry) | Real-time live flight/hotel inventory APIs |
| Automated test suite for AI quality | Complex drag-and-drop calendar editors |

---

### 9. Success Criteria (AI Quality Engineering Focus)
1. **Schema Compliance:** 100% of outputs conform to the structured data contract.
2. **Arithmetic Accuracy:** 100% mathematical accuracy in budget breakdowns.
3. **Resilience:** Invalid inputs rejected before calling the LLM; API/model failures handled with friendly user messaging.
4. **Automated Test Coverage:** Unit and integration tests covering valid inputs, invalid inputs, self-correction triggers, and edge-case budgets.
