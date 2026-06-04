# DATABASE.md

# XAUUSD SMC Fibonacci AI Signal System

Version: 2.0

Database: Firebase Firestore

Status: Production Database Specification

---

# 1. Purpose

This document defines the complete database architecture for the XAUUSD SMC Fibonacci AI Signal System.

The database is responsible for:

* Signal storage
* Signal history
* AI validation history
* System monitoring
* Connection monitoring
* Notification history
* Audit logging
* Analytics aggregation

The database must not store:

* Dummy data
* Mock data
* Temporary test data in production

---

# 2. Database Technology

Provider:

```text
Firebase Firestore
```

Mode:

```text
Native Mode
```

Region:

```text
asia-southeast1
```

Consistency:

```text
Strong Consistency
```

---

# 3. Collection Overview

```text
signals

system

connections

notifications

ai_history

scan_history

logs

analytics
```

---

# 4. Collection: signals

Purpose:

Store every generated signal.

Document ID:

```text
auto-generated uuid
```

Schema:

```json
{
  "id": "",
  "timestamp": "",
  "mode": "SCALPING",
  "type": "BUY",

  "symbol": "XAUUSD",

  "entry": 0,

  "sl": 0,

  "tp1": 0,

  "tp2": 0,

  "rr_tp1": 0,

  "rr_tp2": 0,

  "confidence": 0,

  "bias": "",

  "trend": "",

  "atr": 0,

  "fvg_high": 0,

  "fvg_low": 0,

  "fvg_midpoint": 0,

  "fib_zone_low": 0,

  "fib_zone_high": 0,

  "bos_detected": false,

  "choch_detected": false,

  "killzone_active": false,

  "news_blocked": false,

  "ai_verdict": "",

  "ai_reason": "",

  "status": "ACTIVE",

  "result": "PENDING",

  "created_at": "",

  "updated_at": ""
}
```

---

# 5. Signal Status

Allowed values:

```text
ACTIVE

EXPIRED

CANCELLED

COMPLETED
```

---

# 6. Signal Result

Allowed values:

```text
PENDING

TP1_HIT

TP2_HIT

STOP_LOSS

BREAKEVEN
```

---

# 7. Collection: system

Purpose:

Store global application state.

Document:

```text
connection_test
```

Schema:

```json
{
  "status": "ONLINE",

  "last_check": "",

  "version": "",

  "environment": "PRODUCTION",

  "scheduler_status": "",

  "market_feed_status": "",

  "updated_at": ""
}
```

---

# 8. Collection: connections

Purpose:

Monitor external service health.

Document Structure:

```json
{
  "service": "Telegram",

  "status": "ONLINE",

  "latency_ms": 0,

  "last_check": "",

  "error_message": "",

  "updated_at": ""
}
```

Services:

```text
Backend

Firestore

Telegram

Gemini

NewsAPI

GNews

Yahoo

TwelveData

Scheduler
```

---

# 9. Collection: notifications

Purpose:

Store Telegram delivery history.

Schema:

```json
{
  "signal_id": "",

  "telegram_message_id": "",

  "delivery_status": "",

  "delivery_time_ms": 0,

  "error": "",

  "created_at": ""
}
```

---

# 10. Collection: ai_history

Purpose:

Store all Gemini validation results.

Schema:

```json
{
  "signal_id": "",

  "model": "gemini-1.5-flash",

  "input_payload": {},

  "verdict": "",

  "reason": "",

  "response_time_ms": 0,

  "created_at": ""
}
```

---

# 11. Collection: scan_history

Purpose:

Store every scan execution.

Schema:

```json
{
  "scan_id": "",

  "started_at": "",

  "completed_at": "",

  "execution_time_ms": 0,

  "signal_found": false,

  "signal_id": "",

  "status": "",

  "error": ""
}
```

---

# 12. Collection: logs

Purpose:

Centralized audit logging.

Schema:

```json
{
  "category": "",

  "level": "",

  "message": "",

  "metadata": {},

  "created_at": ""
}
```

Categories:

```text
SYSTEM

SIGNAL

DATABASE

AI

TELEGRAM

NEWS

MARKET

SCHEDULER

SECURITY
```

Levels:

```text
INFO

WARNING

ERROR

CRITICAL
```

---

# 13. Collection: analytics

Purpose:

Store aggregated metrics.

Document:

```text
daily_summary
```

Schema:

```json
{
  "date": "",

  "total_signals": 0,

  "buy_signals": 0,

  "sell_signals": 0,

  "tp1_hits": 0,

  "tp2_hits": 0,

  "stop_loss_hits": 0,

  "win_rate": 0,

  "loss_rate": 0,

  "average_rr": 0,

  "ai_acceptance_rate": 0,

  "telegram_success_rate": 0
}
```

---

# 14. Firestore Index Strategy

Required Indexes:

### Signals

```text
timestamp DESC

created_at DESC

confidence DESC

type ASC

mode ASC

status ASC

result ASC
```

Composite:

```text
type + timestamp

mode + timestamp

confidence + timestamp

status + timestamp

result + timestamp
```

---

# 15. Query Optimization

Most frequent queries:

```text
Latest Signal

Signal History

High Confidence Signals

Today's Signals

Signal Statistics
```

Limit all dashboard queries:

```text
Maximum 100 records
```

Use pagination.

Never load entire collections.

---

# 16. Timestamp Standard

All timestamps:

```text
UTC Storage
```

Display:

```text
Asia/Makassar
```

Format:

```text
DD/MM/YYYY HH:mm:ss WITA
```

---

# 17. Data Validation Rules

Before save:

Required:

```text
symbol

entry

sl

tp1

tp2

confidence

timestamp
```

Validation:

```text
confidence >= 0

confidence <= 100

entry > 0

sl > 0

tp1 > 0

tp2 > 0
```

---

# 18. Retention Policy

signals

```text
Permanent
```

analytics

```text
Permanent
```

logs

```text
90 Days
```

notifications

```text
90 Days
```

scan_history

```text
90 Days
```

ai_history

```text
180 Days
```

---

# 19. Backup Strategy

Daily Backup

```text
01:00 WITA
```

Retention:

```text
30 Daily

12 Monthly
```

Backup Contents:

```text
signals

analytics

system

connections

notifications

ai_history
```

---

# 20. Recovery Strategy

Recovery Objectives

RPO

```text
24 Hours
```

RTO

```text
1 Hour
```

Recovery Flow:

```text
Restore Firestore Export

↓

Verify Collections

↓

Verify Indexes

↓

Verify Connectivity

↓

Resume Scheduler
```

---

# 21. Security Rules

Only backend service account may write.

Frontend:

```text
Read Only
```

Blocked:

```text
Direct Client Writes

Direct Collection Deletes

Anonymous Access
```

---

# 22. Firestore Security Rules

```javascript
rules_version = '2';

service cloud.firestore {

  match /databases/{database}/documents {

    match /{document=**} {

      allow read: if request.auth != null;

      allow write: if false;
    }
  }
}
```

Backend uses Firebase Admin SDK.

---

# 23. Repository Layer Design

Required repositories:

```text
SignalRepository

AnalyticsRepository

ConnectionRepository

NotificationRepository

AIHistoryRepository

LogRepository

SystemRepository
```

Repository Pattern is mandatory.

Business logic must never access Firestore directly.

---

# 24. Database Health Monitoring

Monitor:

```text
Read Latency

Write Latency

Connection State

Index Errors

Permission Errors
```

States:

```text
ONLINE

OFFLINE

DEGRADED

RECONNECTING
```

---

# 25. Definition of Database Completion

Database implementation is complete only if:

* Firestore connected
* Collections created
* Validation rules implemented
* Repositories implemented
* Indexes created
* Backup process configured
* Recovery procedure documented
* Security rules applied
* Health monitoring operational
* Analytics aggregation operational
* No dummy data
* No mock data
* Production deployment verified

