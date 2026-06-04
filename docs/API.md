# API.md

# XAUUSD SMC Fibonacci AI Signal System

Version: 2.0

API Specification

Status: Production Ready

---

# 1. API Overview

This document defines all API contracts used by:

* Frontend Dashboard
* Mobile Application
* Scheduler
* Monitoring Services
* Telegram Integration

All APIs must use:

```text
HTTPS
JSON
UTF-8
REST Architecture
```

---

# 2. Base URL

Development

```text
http://localhost:5000/api
```

Production

```text
https://api.your-domain.com/api
```

---

# 3. Standard Response Format

Success

```json
{
  "success": true,
  "message": "Operation completed",
  "data": {}
}
```

Error

```json
{
  "success": false,
  "message": "Error message",
  "error_code": "ERROR_CODE",
  "data": null
}
```

---

# 4. HTTP Status Codes

```text
200 OK

201 Created

400 Bad Request

401 Unauthorized

403 Forbidden

404 Not Found

409 Conflict

422 Validation Error

429 Rate Limited

500 Internal Server Error

503 Service Unavailable
```

---

# 5. GET /api/health

Purpose

System health check.

Response

```json
{
  "success": true,
  "data": {
    "status": "ONLINE",
    "version": "2.0",
    "timestamp": ""
  }
}
```

---

# 6. GET /api/system/status

Purpose

Return overall system status.

Response

```json
{
  "success": true,
  "data": {
    "system_status": "ONLINE",
    "scheduler": "ONLINE",
    "market_feed": "ONLINE",
    "database": "ONLINE",
    "telegram": "ONLINE",
    "gemini": "ONLINE"
  }
}
```

---

# 7. GET /api/connections

Purpose

Return all service connection statuses.

Response

```json
{
  "success": true,
  "data": [
    {
      "service": "Telegram",
      "status": "ONLINE",
      "latency_ms": 120
    }
  ]
}
```

---

# 8. GET /api/latest-signal

Purpose

Return latest generated signal.

Response

```json
{
  "success": true,
  "data": {
    "id": "",
    "symbol": "XAUUSD",
    "type": "BUY",
    "entry": 0,
    "sl": 0,
    "tp1": 0,
    "tp2": 0,
    "confidence": 0,
    "status": "",
    "timestamp": ""
  }
}
```

---

# 9. GET /api/signals

Purpose

Return signal history.

Query Parameters

```text
page

limit

type

mode

status

result

date_from

date_to
```

Example

```http
GET /api/signals?page=1&limit=20
```

Response

```json
{
  "success": true,
  "data": {
    "items": [],
    "total": 0,
    "page": 1,
    "limit": 20
  }
}
```

---

# 10. GET /api/signals/{id}

Purpose

Get signal detail.

Response

```json
{
  "success": true,
  "data": {
    "id": "",
    "symbol": "",
    "entry": 0,
    "sl": 0,
    "tp1": 0,
    "tp2": 0,
    "confidence": 0,
    "ai_verdict": "",
    "ai_reason": ""
  }
}
```

---

# 11. POST /api/scan

Purpose

Execute manual market scan.

Request

```json
{
  "mode": "SCALPING"
}
```

Response

```json
{
  "success": true,
  "message": "Scan completed",
  "data": {
    "signal_found": true,
    "signal_id": ""
  }
}
```

---

# 12. POST /api/send_signal

Purpose

Send signal to Telegram manually.

Request

```json
{
  "signal_id": ""
}
```

Response

```json
{
  "success": true,
  "message": "Signal delivered"
}
```

---

# 13. POST /api/switch_mode

Purpose

Switch trading mode.

Request

```json
{
  "mode": "SCALPING"
}
```

Allowed Values

```text
SCALPING

INTRADAY
```

Response

```json
{
  "success": true,
  "message": "Mode updated"
}
```

---

# 14. POST /api/save_config

Purpose

Store application configuration.

Request

```json
{
  "telegram_chat_id": "",
  "timezone": "Asia/Makassar"
}
```

Response

```json
{
  "success": true
}
```

---

# 15. GET /api/test_connection

Purpose

Test all external services.

Response

```json
{
  "success": true,
  "data": {
    "firestore": true,
    "telegram": true,
    "gemini": true,
    "newsapi": true,
    "gnews": true,
    "twelvedata": true
  }
}
```

---

# 16. POST /api/test-telegram

Purpose

Verify Telegram integration.

Response

```json
{
  "success": true,
  "message": "Telegram test successful"
}
```

---

# 17. POST /api/test-gemini

Purpose

Verify Gemini integration.

Response

```json
{
  "success": true,
  "message": "Gemini test successful"
}
```

---

# 18. POST /api/reconnect

Purpose

Reconnect failed services.

Request

```json
{
  "service": "Telegram"
}
```

Response

```json
{
  "success": true,
  "message": "Reconnect initiated"
}
```

---

# 19. GET /api/analytics

Purpose

Return dashboard analytics.

Response

```json
{
  "success": true,
  "data": {
    "total_signals": 0,
    "win_rate": 0,
    "loss_rate": 0,
    "average_rr": 0,
    "ai_acceptance_rate": 0,
    "telegram_success_rate": 0
  }
}
```

---

# 20. GET /api/history

Purpose

Return activity history.

Query Parameters

```text
page

limit

category

date_from

date_to
```

Categories

```text
SIGNAL

AI

SYSTEM

DATABASE

TELEGRAM

NEWS
```

---

# 21. GET /api/logs

Purpose

Return system logs.

Response

```json
{
  "success": true,
  "data": {
    "items": []
  }
}
```

---

# 22. GET /api/news/status

Purpose

Return news filter status.

Response

```json
{
  "success": true,
  "data": {
    "news_block": false,
    "next_high_impact_news": "",
    "impact_level": ""
  }
}
```

---

# 23. GET /api/scanner/latest

Purpose

Return latest scan result.

Response

```json
{
  "success": true,
  "data": {
    "bias": "",
    "bos": true,
    "choch": true,
    "fvg": true,
    "fib": true,
    "atr": 0,
    "confidence": 0,
    "decision": "VALID"
  }
}
```

---

# 24. GET /api/dashboard

Purpose

Single endpoint optimized for dashboard loading.

Response

```json
{
  "success": true,
  "data": {
    "system_health": {},
    "connections": [],
    "latest_signal": {},
    "analytics": {},
    "news_status": {},
    "mode": "SCALPING"
  }
}
```

---

# 25. Validation Rules

Mode

```text
SCALPING

INTRADAY
```

Signal Type

```text
BUY

SELL
```

Confidence

```text
0 - 100
```

Entry

```text
Must be > 0
```

---

# 26. Error Codes

```text
INVALID_REQUEST

INVALID_MODE

INVALID_SIGNAL

SIGNAL_NOT_FOUND

DATABASE_ERROR

MARKET_DATA_ERROR

NEWS_API_ERROR

TELEGRAM_ERROR

GEMINI_ERROR

FIRESTORE_ERROR

RATE_LIMIT_EXCEEDED

SERVICE_UNAVAILABLE
```

---

# 27. Rate Limiting

Public Endpoints

```text
60 requests/minute
```

Scan Endpoint

```text
10 requests/minute
```

Reconnect Endpoint

```text
5 requests/minute
```

---

# 28. Authentication

Current Version

```text
Single User System
```

Future Ready

```text
JWT Authentication
Role Based Access Control
```

---

# 29. Logging Requirements

Every request must log:

```text
Request ID

Method

Endpoint

Execution Time

Response Code

IP Address

Timestamp
```

---

# 30. OpenAPI Compatibility

Backend must generate:

```text
swagger.json
```

Documentation Endpoint

```text
GET /api/docs
```

Framework Recommendation

```text
Flask-RESTX
```

or

```text
Flasgger
```

---

# 31. Definition of API Completion

API implementation is complete only if:

* All endpoints implemented
* Validation implemented
* Error handling implemented
* Logging implemented
* Swagger documentation available
* Frontend integration verified
* Mobile integration verified
* Firestore integration verified
* Gemini integration verified
* Telegram integration verified
* No mock responses
* No dummy data
* Production deployment tested

