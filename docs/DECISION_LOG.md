# Architecture & Product Decision Log

This log tracks architectural and product decisions made collaboratively, recording the rationale, trade-offs, and settled milestones.

---

## Log Entry 1: Project Selection & Primary Goal
* **Date:** 2026-10-17
* **Status:** Finalized
* **Decision:** Selected **Project 1: AI-Powered Travel Itinerary Planner** as the starting portfolio project.
* **Objective:** Showcase AI Engineering and AI Quality Engineering (AI QE) capabilities—specifically structured outputs, deterministic constraint validation, and handling non-deterministic LLM edge cases.

---

## Log Entry 2: Scope & Boundary Definition (Lean MVP)
* **Date:** 2026-10-17
* **Status:** Finalized
* **Decision:**
  * **In-Scope:** Single-city trips (2–5 days), single planner managing inputs for their party, deterministic budget validation, automated AI test suite.
  * **Out-of-Scope (Cut to prevent distraction):** Multi-user logins/sessions/voting, relational databases, live booking/payment APIs, multi-city flights, and complex calendar drag-and-drop UIs.
* **Rationale:** Maximizes resume impact by focusing 100% of effort on the AI reliability pipeline and test engineering rather than generic CRUD boilerplate.

---

## Log Entry 3: User Inputs & Accommodation Logic
* **Date:** 2026-10-17
* **Status:** Finalized
* **Decision:** Use **Start Date & End Date** instead of a raw "number of days".
* **Rationale:**
  1. Accurately calculates hotel nights (`nights = days - 1`), allowing realistic accommodation cost estimation (`nights * nightly_rate`).
  2. Provides day-of-week context (attraction opening hours and weekend surges).
  3. Enables deterministic date range validation (`end_date > start_date` and `duration <= 5`).

---

## Log Entry 4: "Worth the Splurge" Section
* **Date:** 2026-10-17
* **Status:** Finalized
* **Decision:** Keep the primary itinerary strictly within budget, but append an optional "Worth the Splurge" section with 2–3 popular activities exceeding the budget.
* **Rationale:** Solves traveler FOMO (Fear Of Missing Out) while keeping core budget constraints intact and mathematically sound.

---

## Log Entry 5: Validation Failure & Self-Correction Strategy
* **Date:** 2026-10-21
* **Status:** Finalized
* **Decision:** Adopted **Approach C (Hybrid: 1 Automated Self-Correction Retry, followed by Graceful Warning Badge)**.
* **Rationale:**
  * Self-correction loop: Passes validation errors back to the model for an automated fix.
  * Cap at 1 retry: Prevents runaway token costs and excessive latency.
  * Graceful degradation: Displays the plan with transparent alerts if it still exceeds budget, ensuring a reliable user experience without crashing.

---

## Log Entry 6: Documentation & Repository Strategy
* **Date:** 2026-10-21
* **Status:** Finalized
* **Decision:** Store formal PRD and Decision Log directly inside the repository under `docs/`.
* **Rationale:** Demonstrates professional engineering lifecycle and product clarity directly to technical recruiters and hiring managers inspecting GitHub.

---

## Log Entry 7: Tech Stack Selection (Dual-Stack Architecture)
* **Date:** 2026-10-22
* **Status:** Finalized
* **Decision:** Decoupled architecture using **Python (FastAPI + Pydantic + Pytest)** on the backend and **TypeScript (React / Vite)** on the frontend.
* **Rationale:**
  1. Python is the industry standard for AI Engineering & AI QE, seamlessly connecting to **Project 2 (Python LLM Eval Framework)**.
  2. TypeScript provides a modern, type-safe frontend and sets up the test target for **Project 3 (TypeScript Playwright Automation)**.

---

## Log Entry 8: AI Provider & Model Choice
* **Date:** 2026-10-22
* **Status:** Finalized
* **Decision:** Selected **Google Gemini (`gemini-1.5-flash` / `gemini-2.0-flash`)** via Google AI Studio free tier.
* **Rationale:** Zero development cost, rapid response times, and native JSON schema output enforcement. Decoupled via service interface to allow switching providers via environment variables.

---

## Log Entry 9: Numerical Group Sizing & Error Feedback Mechanics
* **Date:** 2026-10-22
* **Status:** Finalized
* **Decision:**
  1. Replaced loose string group size with explicit integer `traveler_count: int` (1 to 6) to enable exact per-person arithmetic multipliers for admission tickets and meals.
  2. Adopted **Specific Error Feedback** for the self-correction loop (reporting exact math discrepancies and target categories) to maximize repair success on Retry 1.
