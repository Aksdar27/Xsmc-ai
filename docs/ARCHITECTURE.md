# ARCHITECTURE.md

# XAUUSD SMC Fibonacci AI Signal System

Version: 2.0

Status: Production Architecture Specification

---

# 1. Purpose

This document defines the complete system architecture for the XAUUSD Smart Money Concept (SMC) Fibonacci AI Signal System.

The architecture must support:

* Real-time market analysis
* Signal generation
* AI validation
* Telegram notification delivery
* Firestore persistence
* Mobile application support
* Production deployment

The architecture must not use:

* Dummy data
* Mock responses
* Hardcoded signals
* Simulated analysis

All data must come from real services.

---

# 2. High-Level Architecture

```text
                 USER

                   │

                   ▼

          React Dashboard
          Capacitor Mobile

                   │

                   ▼

              Flask API

                   │

 ┌─────────────────┼─────────────────┐
 │                 │                 │
 ▼                 ▼                 ▼

Signal Engine   Scheduler      Health Monitor

 │                 │                 │
 │                 │                 │
 ▼                 ▼                 ▼

BOS Engine    APScheduler    Connection Checker
CHOCH Engine
FVG Engine
Fib Engine
ATR Engine
Confidence Engine

                   │

                   ▼

             AI Validator

                   │

                   ▼

             Gemini API

                   │

                   ▼

              Firestore

                   │

                   ▼

            Telegram Bot
```

---

# 3. Technology Stack

## Frontend

```text
React 19
TypeScript
TailwindCSS
Shadcn UI
React Query
Axios
React Router
```

## Mobile

```text
Capacitor
Android APK
```

## Backend

```text
Python 3.12
Flask
Gunicorn
APScheduler
Pydantic
```

## Database

```text
Firebase Firestore
```

## External Services

```text
TwelveData
Yahoo Finance
NewsAPI
GNews
Gemini 1.5 Flash
Telegram Bot API
```

---

# 4. Repository Structure

```text
root/

├── backend/
│
├── frontend/
│
├── mobile/
│
├── docs/
│
├── tests/
│
├── scripts/
│
├── credentials/
│
├── .env.example
│
├── README.md
│
└── docker-compose.yml
```

---

# 5. Backend Architecture

```text
backend/

├── app.py

├── api/
│   ├── scan.py
│   ├── signals.py
│   ├── status.py
│   └── settings.py
│
├── engines/
│   ├── bos_engine.py
│   ├── choch_engine.py
│   ├── fvg_engine.py
│   ├── fibonacci_engine.py
│   ├── atr_engine.py
│   ├── confidence_engine.py
│   └── signal_engine.py
│
├── services/
│   ├── twelvedata_service.py
│   ├── yahoo_service.py
│   ├── gemini_service.py
│   ├── news_service.py
│   ├── telegram_service.py
│   └── firestore_service.py
│
├── scheduler/
│   └── scanner_scheduler.py
│
├── validators/
│
├── models/
│
├── repositories/
│
└── utils/
```

---

# 6. Frontend Architecture

## Screens

```text
Dashboard
Scanner
Signals
Analytics
History
Settings
```

## Shared Components

```text
Status Card
Signal Card
Connection Card
Metric Card
Timeline Item
Notification Item
```

## State Management

```text
React Query

Used for:

- API cache
- polling
- background refresh
- synchronization
```

---

# 7. Mobile Architecture

Capacitor wraps the React application.

```text
React Frontend

      ↓

Capacitor

      ↓

Android APK
```

Single codebase.

No separate mobile backend.

---

# 8. Market Data Architecture

## Primary Provider

```text
TwelveData
```

Symbol:

```text
XAU/USD
```

## Fallback Provider

```text
Yahoo Finance
```

Symbol:

```text
GC=F
```

---

# 9. Market Data Failover

Failure conditions:

```text
HTTP Error
Timeout
429 Rate Limit
Malformed Response
Empty Response
```

Flow:

```text
Try TwelveData

Success
    ↓
 Continue

Failed
    ↓

Switch to Yahoo

Success
    ↓
 Continue

Failed
    ↓

Market Feed Error
```

---

# 10. Signal Engine Architecture

```text
Market Data

      ↓

BOS Engine

      ↓

CHOCH Engine

      ↓

FVG Engine

      ↓

Fibonacci Engine

      ↓

ATR Engine

      ↓

Confidence Engine

      ↓

AI Validation

      ↓

Signal Decision
```

---

# 11. Scheduler Architecture

Framework:

```text
APScheduler
BackgroundScheduler
```

Interval:

```text
60 seconds
```

Execution Flow:

```text
Fetch Market Data

↓

Check Killzone

↓

Check News Filter

↓

Calculate BOS

↓

Calculate CHOCH

↓

Calculate FVG

↓

Calculate Fibonacci

↓

Calculate ATR

↓

Calculate Confidence

↓

Run Gemini Validation

↓

Save Signal

↓

Send Telegram

↓

Update Dashboard
```

---

# 12. AI Validation Architecture

Provider:

```text
Gemini 1.5 Flash
```

Input:

```json
{
  "trend": "",
  "bos": "",
  "choch": "",
  "fvg": "",
  "fib": "",
  "atr": "",
  "confidence": ""
}
```

Output:

```json
{
  "verdict": "HIGH_QUALITY",
  "reason": ""
}
```

Retry:

```text
3 Attempts
```

Timeout:

```text
20 Seconds
```

---

# 13. Notification Architecture

```text
Signal Generated

      ↓

Build Message

      ↓

Telegram API

      ↓

Delivery Status

      ↓

Firestore Log
```

---

# 14. Firestore Architecture

Collections:

```text
signals
system
connections
logs
notifications
ai_history
```

All write operations must be atomic.

---

# 15. Health Monitoring

Health Monitor checks:

```text
Backend
Firestore
Gemini
Telegram
NewsAPI
GNews
Yahoo
TwelveData
Scheduler
```

Possible states:

```text
ONLINE
OFFLINE
DEGRADED
RECONNECTING
```

---

# 16. Logging Architecture

Levels:

```text
INFO
WARNING
ERROR
CRITICAL
```

Log Categories:

```text
System
Signals
AI
Telegram
Database
Scheduler
Connections
```

---

# 17. Security Architecture

Rules:

```text
No secrets in repository
No Firebase credentials committed
No hardcoded API keys
No exposed tokens
```

Store secrets only in:

```text
Environment Variables
```

---

# 18. Deployment Architecture

Frontend

```text
Vercel
```

Backend

```text
Railway
```

Database

```text
Firebase Firestore
```

Mobile

```text
Capacitor Android Build
```

---

# 19. Observability

Required metrics:

```text
API Latency
Signal Generation Time
Gemini Response Time
Telegram Delivery Time
Database Write Time
Scheduler Runtime
```

Retention:

```text
30 Days
```

---

# 20. Scalability Requirements

System must support:

```text
Continuous 24/7 Operation
Automatic Recovery
API Failure Handling
Provider Failover
Connection Re-establishment
```

---

# 21. Definition of Architecture Completion

Architecture is complete only if:

* Backend modules implemented
* Firestore connected
* Gemini connected
* Telegram connected
* TwelveData connected
* Yahoo fallback works
* Scheduler works
* Dashboard connected
* Mobile build succeeds
* Health monitoring operational
* Logging operational
* No dummy data
* No mock responses
* All integrations use real services

