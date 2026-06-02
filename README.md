# 🚌 AI-Driven Public Transport Operations Decision Agent

> **Version 1.5 | April 2026**  
> A production-oriented multi-agent AI system for real-time urban transit optimization.

[![Platform](https://img.shields.io/badge/Platform-n8n-blue)](https://n8n.io)
[![LLM](https://img.shields.io/badge/LLM-Groq%20Llama--3.3--70b-orange)](https://groq.com)
[![Language](https://img.shields.io/badge/Language-JavaScript-yellow)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![Agents](https://img.shields.io/badge/Sub--Agents-4-green)](#architecture)
[![Functions](https://img.shields.io/badge/Functions-12-purple)](#system-functions-fn1--fn12)
[![Data Source](https://img.shields.io/badge/Data-MBTA%20v3%20API-red)](https://api-v3.mbta.com)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [System Functions (Fn1–Fn12)](#system-functions-fn1--fn12)
- [Sub-Agents](#sub-agents)
- [Confidence-Based Escalation](#confidence-based-escalation)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Deliverables](#deliverables)
- [Testing](#testing)
- [Security](#security)
- [Ethical Framework](#ethical-framework)
- [Cost Estimation (COCOMO)](#cost-estimation-cocomo)
- [Future Roadmap](#future-roadmap)
- [Project Info](#project-info)

---

## Overview

The **AI-Driven PT Operations Decision Agent** is a fully autonomous, multi-agent system built to optimize real-time transit operations across urban bus networks. It monitors **228 vehicles across 85 routes** in the MBTA Boston network, executing a complete decision cycle every **2 minutes**.

Built on the **n8n workflow automation platform** and powered by **Groq Llama-3.3-70b**, the system ingests live data, detects congestion, classifies operational risk, and executes rerouting and frequency decisions — all without human intervention under normal conditions.

```
Schedule Trigger (every 2 min)
        │
        ▼
   MBTA v3 API ──► Data Ingestion (Fn1–Fn2)
        │
        ▼
  Main Coordinator Brain (Llama-3.3-70b)
   ┌────┴─────────────────────────┐
   ▼         ▼         ▼         ▼
 SA1       SA2       SA3       SA4
 Risk    Anomaly  Decisions   Audit
   └────┬─────────────────────────┘
        │
        ▼
  Confidence Gate (≥ 0.6?)
   ├── YES → Autonomous Action Layer → HTML Dashboard
   └── NO  → Human Escalation Email
```

---

## Key Features

- **Real-time monitoring** — 2-minute polling cycle against the live MBTA v3 API
- **Multi-agent architecture** — 4 specialized sub-agents coordinated by a central Brain
- **12 discrete functions** — from raw API ingestion to structured output delivery
- **Confidence-based escalation** — autonomous action only when confidence ≥ 0.6
- **Ethical load balancing** — route-agnostic thresholds prevent bias toward high-density corridors
- **Prompt injection hardening** — whitelist-based data normalization and system prompt guardrails
- **Full audit trail** — every cycle logged by SA4 with timestamp and AI reasoning path
- **HTML dashboard output** — fleet statistics, active alerts, route performance, traffic index
- **Mobile risk alert app** — MIT App Inventor dispatcher interface
- **Intelligence analytics dashboard** — Firebase-hosted with Radar charts, Doughnut charts, and KPI cards

---

## Architecture

The system follows a layered hub-and-spoke architecture:

| Layer | Components | Role |
|---|---|---|
| **Input** | MBTA v3 API, Schedule Trigger | Live data ingestion every 2 minutes |
| **Reasoning** | Main Coordinator (Llama-3.3-70b) | Task delegation, context management |
| **Sub-Agent** | SA1, SA2, SA3, SA4 | Specialized cognitive tasks |
| **Decision** | Confidence Gate, Merge Node | Output aggregation and routing |
| **Output** | HTML Dashboard, Email Alerts | Operational delivery |

### Traffic Index Formula

```
Index = (avgDelay × 0.5) + (avgLoad × 5)

  < 4.0  →  🟢 Green  (Normal Operations)
  4–7    →  🟡 Yellow (Moderate Concern)
  > 7.0  →  🔴 Red    (Critical Congestion)
```

---

## System Functions (Fn1–Fn12)

| ID | Function | Agent | Description |
|---|---|---|---|
| Fn1 | Ingest Transport Data | SA1 | Pull live vehicle data from MBTA v3 API (position, occupancy, status, route) |
| Fn2 | Clean & Normalise | SA1 | Clamp load_factor to [0,1], delay_minutes ≥ 0, discard records > 10 min old |
| Fn3 | Detect Congestion | SA1 | Flag routes where avgDelay > 5 min or avgLoad > 0.85 |
| Fn4 | Assess Delays | SA1 | Compute avgDelay and avgLoad per route; produce delay_assessment object |
| Fn5 | Monitor Utilisation | SA2 | Flag overloaded (>0.9), underutilized (<0.3), and sudden load spikes (>0.2 in 5 min) |
| Fn6 | Classify Risk | SA1 | Low / Medium / High risk based on congestion thresholds |
| Fn7 | Rerouting Decisions | SA3 | Generate rerouting_decisions array for all High-risk routes |
| Fn8 | Frequency Adjustment | SA3 | Increase to 5-min intervals (load >0.85), decrease to 20-min (load <0.3) |
| Fn9 | Maintenance Alerts | SA4 | Flag vehicles with repeated delays across consecutive cycles |
| Fn10 | Escalation & Handoff | SA4 | Route to Action Layer (confidence ≥ 0.6) or human email (confidence < 0.6) |
| Fn11 | Audit Log | SA4 | Structured log entry per cycle: timestamp, risk level, actions, notes |
| Fn12 | Effectiveness Score | SA4 | Score 0–1 per cycle (High=0.2, Medium=0.5, Low=0.8) |

---

## Sub-Agents

### SA1 — Risk Analysis Agent
Handles data ingestion (Fn1), normalization (Fn2), congestion detection (Fn3–Fn4), and risk classification (Fn6). Uses standardized mathematical thresholds to ensure route-agnostic fairness.

### SA2 — Anomaly Detection Agent
Runs independently in parallel. Monitors vehicle utilization, sudden load spikes, and persistent underservice patterns (Fn5).

### SA3 — Optimization Agent
Generates rerouting decisions (Fn7) and frequency adjustment recommendations (Fn8). Outputs a confidence score used by the downstream escalation gate.

### SA4 — Execution & QA Agent
Manages maintenance alerts (Fn9), the escalation/handoff circuit (Fn10), audit logging (Fn11), and effectiveness scoring (Fn12). Acts as the system's black-box recorder.

---

## Confidence-Based Escalation

| Confidence Score | Risk Level | Action |
|---|---|---|
| ≥ 0.80 | Low | Autonomous Dashboard Update |
| 0.60 – 0.79 | Medium | Operational Alert Generated |
| < 0.60 | **High** | **Execution blocked — Human escalation email sent** |

The escalation path is a **critical exception circuit**, not a routine operating mode. The system currently escalates approximately **15% of cases**.

---

## Getting Started

### Prerequisites

- [n8n](https://n8n.io) (self-hosted or cloud)
- Groq API key — [console.groq.com](https://console.groq.com)
- MBTA v3 API key — [api-v3.mbta.com](https://api-v3.mbta.com)
- Node.js ≥ 18

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/<your-username>/pt-operations-agent.git
   cd pt-operations-agent
   ```

2. **Import the n8n workflow**
   - Open your n8n instance
   - Go to **Workflows → Import from File**
   - Select `workflow/PT_MultiAgent_Main.json`

3. **Configure credentials** (see [Configuration](#configuration))

4. **Activate the workflow**
   - Toggle the workflow to **Active**
   - The Schedule Trigger will begin polling every 2 minutes

---

## Configuration

All secrets are managed via n8n environment variables — never hardcoded.

| Variable | Description |
|---|---|
| `GROQ_API_KEY` | Groq LLM API key |
| `MBTA_API_KEY` | MBTA v3 API key |
| `ESCALATION_EMAIL` | Dispatcher email address for low-confidence alerts |
| `FIREBASE_CONFIG` | Firebase hosting config (for dashboard deployment) |

Set these in **n8n → Settings → Environment Variables** or in your `.env` file for self-hosted deployments.

---

## Deliverables

This project includes five parallel deliverables beyond the core n8n workflow:

| Deliverable | Stack | Description |
|---|---|---|
| **Core AI Workflow** | n8n + Groq + JavaScript | 12-function multi-agent pipeline |
| **Mobile Risk Alert App** | MIT App Inventor | Single-screen dispatcher command center |
| **Intelligence Dashboard** | HTML5 + Tailwind CSS + Chart.js + Firebase | Radar, Doughnut charts, KPI cards |
| **Physical Robot Design** | ESP32 + Arduino Mega + sensors | Hardware mapping of all 12 agent functions |
| **NexusTransport AI Startup** | SaaS concept | Per-route subscription model for transit authorities |

---

## Testing

```
Test Coverage:
  ✅ Functional testing   — Fn1 through Fn12
  ✅ Unit testing         — P-1 Test Plan (4 core test cases)
  ✅ Subsystem testing    — Stubs for congestion and maintenance subsystems
  ✅ Integration testing  — P-2 Integration Plan (voice input → rerouting)
  ✅ Escalation testing   — Forced low-confidence payload (confidence=0.4)
  ✅ Non-functional       — 120s cycle window, 99.9% uptime, encryption validation
```

To run the escalation test, inject this payload manually into the Confidence Switch node:

```json
{
  "delay": 45,
  "load": 0.95,
  "confidence": 0.4
}
```

Expected result: email branch triggered, dispatcher receives HTML report with **"Human Intervention Required"** banner.

---

## Security

Security is implemented in three layers:

**Layer 1 — Ingress & API Security**
- API keys stored via n8n credential encryption (never in plaintext)
- MBTA v3 API rate limit compliance to prevent IP bans

**Layer 2 — Prompt Injection Defense**
- Fn2 whitelist filter: only transport-specific attributes pass to agents
- Negative constraint guardrails in the Main Coordinator system prompt

**Layer 3 — SA4 Compliance & Audit**
- Every AI-generated action logged with timestamp and reasoning path
- Non-repudiation enforced; out-of-scope actions flagged in dashboard

---

## Ethical Framework

| Principle | Implementation |
|---|---|
| **Fairness** | Route-agnostic Traffic Index thresholds — no demographic or geographic bias |
| **Transparency** | Plain-language `reasoning` field included in all outputs and escalation emails |
| **Privacy** | Minimum viable data policy — Vehicle IDs and Trip IDs only; no passenger PII |
| **Safety** | Confidence gate halts execution and contacts a human when certainty collapses |
| **Accountability** | 100% cycle audit logging; SA4 flags any out-of-scope AI actions |

---

## Cost Estimation (COCOMO)

| Metric | Value |
|---|---|
| KDSI | 2.5 (2,500 lines of JavaScript) |
| Project Mode | Semi-Detached |
| EAF | 1.1441 |
| CMM Level | 2 (Repeatable) |
| Effort | **8.62 person-months** |
| Duration | **5.31 months** |
| Team Size | 3 developers |
| **Recommended Cost** | **$103,440 USD** (at $75/hr) |
| Cost Range | $68,960 – $172,400 USD |

---

## Future Roadmap

| Priority | Feature | Description |
|---|---|---|
| 1 | **Geospatial Visualization** | Live map-based operator view replacing HTML text dashboard |
| 2 | **Reinforcement Learning** | Replace static confidence threshold with a feedback-learning model |
| 3 | **Voice Integration** | Hands-free operator monitoring via voice output mode |
| 4 | **Vector Database** | Pinecone/Weaviate for weekly traffic pattern historical recall |
| 5 | **LangGraph Upgrade** | ReAct-loop agents with stateful self-correction cycles (Version 2.0) |

---

## Project Info

| Attribute | Detail |
|---|---|
| **Project Name** | AI-Driven PT Operations Decision Agent |
| **Version** | 1.5 |
| **Author** | Muhammad Mobin |
| **Organization** | NexusTransport AI |
| **Report Date** | April 24, 2026 |
| **Data Source** | MBTA v3 API (Boston bus network) |
| **Vehicles Monitored** | 228 vehicles across 85 routes |
| **Decision Cycle** | Every 2 minutes |
| **Avg Cycle Time** | ~45 seconds |
| **Escalation Rate** | ~15% of cycles |

---

## References

- MBTA v3 API — [api-v3.mbta.com](https://api-v3.mbta.com)
- Groq Cloud — [console.groq.com](https://console.groq.com)
- n8n Documentation — [docs.n8n.io](https://docs.n8n.io)
- LangChain Agents — [python.langchain.com](https://python.langchain.com)
- OWASP LLM Top 10 — [owasp.org](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- IEEE AIS Ethical Guidelines — [standards.ieee.org](https://standards.ieee.org)

---

*NexusTransport AI — Autonomous Transit Intelligence*
