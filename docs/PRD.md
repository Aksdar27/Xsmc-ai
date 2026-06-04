# PRD â€” XAUUSD SMC Fibonacci AI Signal System
Version: 2.1  
Status: Production-Ready Specification  
Owner: Personal Use  
Last Updated: 3 June 2026  
Timezone: Asia/Makassar (WITA)

## 0. Purpose of this document

This document is the single source of truth for the project.

It defines the product scope, business rules, signal logic, system behavior, quality standards, and delivery constraints for the XAUUSD SMC Fibonacci AI Signal System.

### Hard rules

- All market data must come from real APIs.
- The system is not a broker.
- The system is not copy trading.
- The system is not SaaS.
- The system does not auto-trade.
- The system only:
  - analyzes market data
  - validates setups
  - generates signals
  - stores history
  - sends notifications
- No dummy data.
- No mock responses.
- No hardcoded signal logic.
- No random signal generation.
- No placeholder analysis.

### Reading rule

Implement the system in sequence. Do not skip audit, architecture validation, testing, verification, or final review.

---

## 1. Project overview

### Objective

Build an automated XAUUSD signal analysis system based on Smart Money Concept (SMC), Fibonacci logic, ATR-based risk control, news filtering, and AI validation.

The system must identify high-quality trading setups using real market data and publish signals through a controlled, auditable workflow.

### Product outcome

The system should provide:

- market analysis
- setup validation
- signal generation
- signal history storage
- notification delivery
- operational monitoring
- manual scan capability
- connection health visibility
- mobile-friendly access

### Target user

Primary use is personal trading operations.

The system is intended for a single operator who needs reliable, production-grade signal generation with strong validation and clear observability.

---

## 2. Scope

### In scope

- XAUUSD market monitoring
- real-time and historical candle ingestion
- trend and market structure analysis
- BOS detection
- CHOCH detection
- FVG detection
- Fibonacci discount zone calculation
- ATR calculation
- killzone session filtering
- news event filtering
- AI-based setup validation
- confidence scoring
- signal persistence
- Telegram notification delivery
- dashboard monitoring
- manual scan execution
- system health monitoring
- connection monitoring
- frontend dashboard
- Android packaging through Capacitor

### Out of scope

- live order execution
- auto-trading
- broker integration for execution
- copy trading
- portfolio management
- multi-tenant SaaS features
- public user onboarding flow
- marketing pages
- chart-heavy analytics screens
- speculative or simulated signals

---

## 3. Success criteria

The product is successful only if it can:

1. fetch market data from real services
2. detect market structure reliably
3. detect BOS and CHOCH correctly
4. detect FVG conditions correctly
5. compute Fibonacci zones correctly
6. compute ATR correctly
7. block trades during high-impact news windows
8. validate setups using AI
9. generate signals only when all rules pass
10. store every signal in Firestore
11. notify Telegram reliably
12. expose a working dashboard
13. run on a predictable schedule
14. survive source failure using fallback logic
15. deploy successfully to the selected platforms
16. build into an Android APK
17. pass functional and integration tests
18. operate end-to-end without critical errors

---

## 4. Functional requirements

### 4.1 Market data engine

The system must retrieve real XAUUSD market data.

#### Primary source

- TwelveData

#### Primary symbol

- `XAU/USD`

#### Required data

- current price
- OHLC candle data
- historical candles

#### Fallback source

- Yahoo Finance

#### Fallback symbol

- `GC=F`

#### Fallback triggers

Use fallback when the primary source returns:

- timeout
- HTTP error
- rate limit
- empty response
- malformed response

#### Data quality rules

- Reject incomplete candle sets.
- Reject malformed numeric fields.
- Reject stale or duplicate responses where freshness cannot be confirmed.
- Normalize all timestamps to `Asia/Makassar`.

---

### 4.2 Market structure engine

The system must infer trend and structure from price action.

#### Structure requirements

- detect swing highs and swing lows
- detect market bias
- detect trend direction
- confirm structure shift events

#### Swing detection rule

Use a 5-candle fractal model.

A bullish swing high exists when the middle candle high is higher than the highs of the two candles before and the two candles after it.

A bearish swing low exists when the middle candle low is lower than the lows of the two candles before and the two candles after it.

#### BOS rule

- Bullish BOS: close above the last swing high
- Bearish BOS: close below the last swing low

#### Trend logic

- Bullish BOS implies bullish trend
- Bearish BOS implies bearish trend

---

### 4.3 CHOCH engine

The system must detect Change of Character.

#### Bullish CHOCH

Conditions:

- previous trend was bearish
- price breaks the last lower high
- confirmation must use candle close
- wick-only break is ignored

#### Bearish CHOCH

Conditions:

- previous trend was bullish
- price breaks the last higher low
- confirmation must use candle close
- wick-only break is ignored

---

### 4.4 FVG engine

The system must detect fair value gaps.

#### Bullish FVG

A bullish FVG exists when the first candle high is below the third candle low.

#### Bearish FVG

A bearish FVG exists when the first candle low is above the third candle high.

#### Gap rule

- Minimum gap size: `0.30 ATR`

#### Entry zone

- Use the 50% midpoint of the FVG as the preferred entry area.

#### Midpoint formula

`(FVG High + FVG Low) / 2`

---

### 4.5 Fibonacci engine

The system must calculate Fibonacci retracement zones from the swing preceding BOS.

#### Bullish Fibonacci

- 0% = swing low
- 100% = swing high

#### Bearish Fibonacci

- 0% = swing high
- 100% = swing low

#### Discount zone

A signal is only valid when price is inside the defined discount zone.

#### Discount zone boundaries

- 50.0%
- 61.8%

The system must treat the discount zone as a rule-based filter, not as an optional indicator.

---

### 4.6 ATR engine

The system must calculate Average True Range.

#### ATR settings

- Period: 14

#### Timeframe usage

- Scalping: ATR on M5
- Intraday: ATR on M15

#### Risk rule

- Stop loss = ATR Ã— 1.0

#### Requirements

- No randomization
- No synthetic volatility adjustments
- Use real candle data only

---

### 4.7 Trading modes

The system must support at least two modes.

#### Scalping mode

- Bias timeframe: H1
- Execution timeframe: M5
- Confirmation timeframe: M1
- Signal limit: 1 signal per 15 minutes

#### Intraday mode

- Bias timeframe: H4
- Execution timeframe: M15
- Confirmation timeframe: M15
- Signal limit: 1 signal per 60 minutes

#### Mode behavior

- The mode must affect timeframe selection.
- The mode must affect signal frequency.
- The mode must affect TP logic.
- The mode must affect ATR usage.

---

### 4.8 Entry engine

The system must only generate entries when all required conditions are true.

#### BUY setup

All conditions must be true:

1. bullish bias
2. bullish FVG
3. price at FVG midpoint
4. price inside discount zone
5. bullish CHOCH

#### SELL setup

All conditions must be true:

1. bearish bias
2. bearish FVG
3. price at FVG midpoint
4. price inside discount zone
5. bearish CHOCH

#### Entry rule

A setup that fails even one required condition must be rejected.

---

### 4.9 Take profit engine

The system must compute TP targets using a fixed risk-reward structure.

#### Scalping

- TP1 = RR 2.0
- TP2 = RR 3.5

#### Intraday

- TP1 = RR 2.5
- TP2 = RR 4.0

#### Requirements

- TP values must be calculated from the actual entry and stop loss.
- No hardcoded fake TP values.
- Maintain consistency with the selected mode.

---

### 4.10 Killzone engine

The system must block or enable signal generation based on session windows.

#### Active sessions

- 15:00 - 18:00 WITA
- 21:30 - 00:00 WITA

#### Outside killzone

- status: `WAITING_KILLZONE`

#### Behavior

- Outside the active sessions, the scanner must remain in waiting state.
- Inside the active sessions, the scanner may evaluate setups normally.

---

### 4.11 News filter engine

The system must detect and block signal generation during high-impact news windows.

#### Sources

- NewsAPI
- GNews

#### Keyword list

- USD
- Federal Reserve
- Interest Rate
- FOMC
- CPI
- PPI
- NFP
- Gold

#### Block window

- 15 minutes before the event
- 15 minutes after the event

#### Status

- `NEWS_BLOCK`

#### Behavior

- If a relevant high-impact event is active or imminent, the system must block signals.
- News evaluation must be logged and visible in the dashboard.
- News blocking must be reversible when the window expires.

---

### 4.12 AI validation engine

The system must use a language model to validate setup quality.

#### Provider

- Gemini 1.5 Flash

#### Model

- `gemini-1.5-flash`

#### Timeout

- 20 seconds

#### Retry policy

- 3 attempts

#### Input

- structured setup JSON

#### Output

- `HIGH_QUALITY`
- `LOW_QUALITY`
- reason

#### Decision rule

- `HIGH_QUALITY` â†’ signal accepted
- `LOW_QUALITY` â†’ signal rejected

#### Behavior

The AI must validate the setup, not replace the deterministic rules.

The AI layer is a quality filter, not the sole signal generator.

---

### 4.13 Confidence engine

The system must compute a confidence score for each candidate signal.

#### Score allocation

- Bias: 20
- FVG: 20
- Fibonacci: 20
- CHOCH: 20
- AI: 20

#### Range

- Maximum: 100
- Minimum signal threshold: 70

#### Rule

A signal below threshold must not be treated as a valid publishable signal.

---

### 4.14 Signal generation

The system must generate a structured signal only when all gating conditions pass.

#### Required signal fields

- id
- timestamp
- mode
- type
- symbol
- entry
- sl
- tp1
- tp2
- confidence
- bias
- trend
- atr
- fvg_high
- fvg_low
- ai_verdict
- ai_reason
- status
- result
- created_at
- updated_at

#### Signal lifecycle

A signal should move through clear states such as:

- pending
- valid
- invalid
- sent
- failed
- cancelled

#### Signal rejection reasons

The system must store rejection reasons when a setup is invalid, blocked by news, blocked by killzone, rejected by AI, or fails a data-quality check.

---

### 4.15 Notification engine

The system must deliver signals to Telegram.

#### Required environment variables

- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_CHAT_ID`

#### Message content

- XAUUSD SIGNAL
- Mode
- Type
- Entry
- SL
- TP1
- TP2
- Confidence
- AI Verdict
- Timestamp

#### Behavior

- Successful send must be logged.
- Failed send must be logged.
- Retry strategy must be defined in backend implementation.
- Telegram failure must not corrupt stored signal data.

---

### 4.16 Scheduler

The system must run in scheduled intervals.

#### Framework

- APScheduler
- BackgroundScheduler

#### Interval

- Every 60 seconds

#### Execution flow

1. Fetch market data
2. Check killzone
3. Check news
4. Calculate BOS
5. Calculate CHOCH
6. Calculate FVG
7. Calculate Fibonacci
8. Calculate ATR
9. Build setup
10. Compute confidence score
11. Run Gemini validation
12. Save to Firestore
13. Send to Telegram

#### Rules

- Do not execute steps out of sequence.
- Do not send a signal before validation is complete.
- Do not persist invalid data as a valid signal.

---

## 5. Frontend requirements

### 5.1 Frontend stack

- React 19
- TailwindCSS
- Shadcn UI
- Capacitor Android

### 5.2 Frontend goals

The frontend must feel like:

- an institutional trading terminal
- an enterprise monitoring dashboard
- a professional fintech application
- a production-grade mobile trading system

The frontend must not feel like:

- a marketing site
- a landing page
- a generic admin template
- a portfolio website

### 5.3 Visual principles

- high information density
- low visual noise
- high readability
- maximum space efficiency
- strong performance
- institutional-grade appearance

### 5.4 Typography

- Primary font: Inter
- Page title: 22â€“24px, weight 700
- Section title: 14â€“15px, weight 600
- Card label: 10â€“11px, uppercase, letter spacing 0.06emâ€“0.08em
- Main metric: 18â€“22px, weight 700, maximum 24px
- Secondary text: 11â€“13px
- Status text: 11â€“12px

### 5.5 Color system

- Primary background: `#070B14`
- Secondary background: `#0F172A`
- Card background: `#111827`
- Border: `#1E293B`
- Primary accent: `#D4AF37`
- Gold gradient: `#D4AF37 -> #F7D774`
- Success: `#00C853`
- Warning: `#FFB300`
- Danger: `#FF3D57`
- Info: `#29B6F6`
- Text primary: `#FFFFFF`
- Text secondary: `#94A3B8`

### 5.6 Spacing system

- Card padding: 10â€“12px
- Card gap: 10â€“12px
- Section gap: 14â€“16px
- Border radius: 12px
- Touch area minimum: 48px

### 5.7 Layout structure

#### Fixed header

Always visible and showing:

- logo
- application name
- current time
- current date
- notification badge
- system health indicator
- connection status indicator
- user profile menu

#### Fixed bottom navigation

Always visible and showing:

- Dashboard
- Scanner
- Signals
- Analytics
- Settings

#### Navigation rules

- active highlight
- gold accent indicator
- smooth transition
- haptic feedback
- one-hand friendly layout

### 5.8 Dashboard sections

The dashboard is the main operational page.

#### Section 1: system alerts

Show only when there is a problem.

Examples:

- backend offline
- Telegram failed
- Gemini failed
- Firestore error
- market feed error
- news block active

Hide the section when the system is normal.

#### Section 2: system overview

- grid: 2 columns
- card height: 110â€“120px

Cards:

- System Health
- Market Session
- Killzone Status
- Trading Mode

#### Section 3: connection status

- compact list
- maximum height: 220px

Show:

- Market Feed
- Firestore
- Telegram
- Gemini
- NewsAPI
- GNews
- Scheduler
- Yahoo Fallback
- TwelveData

Statuses:

- ONLINE
- OFFLINE
- DEGRADED
- RECONNECTING

#### Section 4: latest signal

Show:

- type
- entry
- SL
- TP1
- TP2
- RR
- confidence
- timestamp
- AI verdict
- signal status

Actions:

- Copy Signal
- Share Signal
- View Detail

#### Section 5: performance summary

- grid: 2 columns
- card height: 100â€“110px

Show:

- total signals
- today's signals
- win rate
- loss rate
- average RR
- pending signals

#### Section 6: AI validation status

Show:

- current model
- validation status
- acceptance rate
- average validation time
- last validation

#### Section 7: news monitoring

Show:

- news filter status
- current block
- blocked keywords
- next high impact news
- impact level

#### Section 8: recent activity timeline

Show:

- signal created
- signal sent
- AI approved
- AI rejected
- news block triggered
- system restart
- connection recovered

#### Section 9: quick actions

Grid: 2 columns

Buttons:

- Manual Scan
- Refresh Data
- Health Check
- Test Telegram
- Test Gemini
- Reconnect Services

### 5.9 Scanner screen

Show the latest scan result.

Display:

- market bias
- BOS
- CHOCH
- FVG
- Fibonacci
- ATR
- killzone
- news filter
- confidence score
- AI validation
- final decision

Statuses:

- VALID
- INVALID
- WAITING
- BLOCKED

### 5.10 Signals screen

Show all signals.

Features:

- search
- sort
- filter
- pagination
- date range

Filters:

- BUY
- SELL
- SCALPING
- INTRADAY
- HIGH CONFIDENCE
- LOW CONFIDENCE
- SUCCESS
- FAILED
- PENDING
- CANCELLED

The signal card must stay compact so the full summary can be seen without opening details.

Display:

- type
- entry
- SL
- TP1
- TP2
- RR
- confidence
- timestamp
- status
- AI verdict

### 5.11 Analytics screen

The analytics screen must not use charts.

Use premium statistics cards instead.

Grid: 2 columns  
Card height: 100px

Show:

- total signals
- win rate
- loss rate
- average RR
- highest RR
- lowest RR
- AI acceptance rate
- AI rejection rate
- signal frequency
- Telegram success rate
- system uptime
- database availability

### 5.12 History screen

Show system history categories:

- Signal History
- AI History
- Telegram History
- News History
- System Logs
- Connection Logs

Features:

- search
- sort
- date filter
- export CSV

### 5.13 Settings screen

Use accordion layout.

Default state: collapsed

Categories:

- General
- Trading
- Notifications
- Connections
- Security
- Application
- Session Management
- Device Management

#### Session management

Show:

- current device
- current session
- login time
- last activity
- IP information

Actions:

- logout current session
- logout other devices
- logout all devices
- emergency logout

#### Device management

Show:

- device name
- platform
- login time
- last activity
- status

Actions:

- remove device
- force logout device

#### Connection monitor

Use a compact table.

Columns:

- service
- status
- latency
- last check

Services:

- Backend
- Firestore
- Telegram
- Gemini
- NewsAPI
- GNews
- Yahoo
- TwelveData
- Scheduler

#### Notification center

Categories:

- Signals
- System
- AI
- Database
- Telegram
- Market
- Security

Actions:

- Mark Read
- Mark All Read
- Delete
- Search
- Filter

### 5.14 Loading and empty states

Every process must include:

- skeleton loader
- progress indicator
- loading message

Do not use blank screens.

Empty states must be compact and action-oriented.

### 5.15 Micro-interactions

Every action must have:

- press state
- loading state
- success state
- error state
- transition animation
- haptic feedback

Animations must be minimal and fast.

### 5.16 Mobile optimization

Target device:

- Android

Target width:

- 360px to 480px

Rules:

- responsive
- fast rendering
- low memory usage
- one-hand navigation
- safe-area compatible
- high information density
- minimal scrolling

#### Performance targets

- Initial load: under 3 seconds
- Dashboard load: under 1 second
- Navigation: under 200 ms
- API display: under 500 ms
- memory efficient
- battery efficient

### 5.17 Accessibility

- large text support
- high contrast mode
- screen reader support
- color blind indicators
- keyboard navigation support

### 5.18 Frontend acceptance rule

The frontend must look like a real professional trading application used for real-time monitoring and signal validation.

It must not look like a generic admin template.

It must not contain charts.

It must not contain empty areas without function.

Every component must have a clear operational purpose.

---

## 6. Database requirements

### 6.1 Database platform

- Firebase Firestore

### 6.2 Core collections

#### `signals`

Document fields:

- id
- timestamp
- mode
- type
- symbol
- entry
- sl
- tp1
- tp2
- confidence
- bias
- trend
- atr
- fvg_high
- fvg_low
- ai_verdict
- ai_reason
- status
- result
- created_at
- updated_at

#### `system`

Document fields:

- `connection_test`
- status
- last_check

### 6.3 Data requirements

The database must support:

- signal history
- AI decision history
- Telegram delivery history
- system health state
- connection status
- scan results
- audit-ready logs

### 6.4 Consistency requirements

- Keep timestamps normalized to WITA.
- Track created and updated time for auditable records.
- Avoid silent overwrites without versioning or traceability.

---

## 7. API requirements

### 7.1 Standard response format

All endpoints must use a standard envelope.

```json
{
  "success": true,
  "message": "",
  "data": {}
}
```

### 7.2 Required endpoints

- `POST /api/save_config`
- `GET /api/test_connection`
- `GET /api/system/status`
- `POST /api/scan`
- `GET /api/signals`
- `GET /api/latest-signal`
- `POST /api/send_signal`
- `POST /api/switch_mode`

### 7.3 Additional operational endpoints

The implementation should also support endpoints for:

- health checks
- connection status
- manual testing of Telegram
- manual testing of Gemini
- reconnect services
- analytics
- logs

### 7.4 API quality rules

- validate request payloads
- return clear errors
- never crash on malformed input
- log every failure path
- protect sensitive credentials
- support frontend refresh operations

---

## 8. Architecture requirements

### 8.1 Backend stack

- Python 3.12
- Flask
- Gunicorn
- APScheduler

### 8.2 External services

- TwelveData
- Yahoo Finance
- NewsAPI
- GNews
- Gemini 1.5 Flash
- Telegram Bot API

### 8.3 Deployment targets

- Frontend â†’ Vercel
- Backend â†’ Railway
- Database â†’ Firebase
- Android â†’ Capacitor APK

### 8.4 Project structure

Recommended repository structure:

```text
frontend/
backend/
docs/
credentials/
scripts/
mobile/
tests/
```

### 8.5 Environment variables

Required variables:

- `TWELVEDATA_API_KEY`
- `NEWSAPI_KEY`
- `GNEWS_API_KEY`
- `GEMINI_API_KEY`
- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_CHAT_ID`
- `FIREBASE_PROJECT_ID`
- `GOOGLE_APPLICATION_CREDENTIALS`
- `APP_TIMEZONE=Asia/Makassar`

### 8.6 Secret handling

- API keys must live in environment variables.
- Do not hardcode secrets.
- Firebase service account JSON must not be committed.
- `.gitignore` must exclude `.env`, `credentials/`, and `*.json`.

---

## 9. Security requirements

- use environment variables for secrets
- avoid committing credential files
- protect API endpoints with validation
- log security-relevant events
- support emergency logout
- support device/session revocation
- keep operational data auditable

---

## 10. Logging and observability

### 10.1 Logging

The system must log:

- market fetch outcomes
- fallback activation
- killzone decisions
- news block decisions
- BOS / CHOCH / FVG / Fibonacci / ATR outputs
- AI validation results
- confidence score calculation
- Firestore writes
- Telegram sends
- API errors
- scheduler runs
- connection failures
- recovery events

### 10.2 Observability

The system must expose:

- system health
- connection health
- last successful scan
- last successful signal
- current block status
- backend uptime
- database availability
- Telegram delivery status
- AI validation status

---

## 11. Testing requirements

The system must pass:

- Market Connection Test
- Firestore Test
- Telegram Test
- Gemini Test
- News Test
- Signal Engine Test
- Deployment Test
- Android Build Test

### Test policy

- Every new feature must include tests.
- Every module must have unit tests where practical.
- Integration tests are required for external services and critical workflows.

---

## 12. Definition of done

The project is complete only when all of the following are true:

1. All APIs work.
2. Firestore is connected.
3. Gemini is connected.
4. Telegram is connected.
5. TwelveData works.
6. Yahoo fallback works.
7. News filter works.
8. Scheduler works.
9. Signal is stored.
10. Signal is sent.
11. Frontend works.
12. Railway deployment succeeds.
13. Vercel deployment succeeds.
14. APK can be built.
15. No dummy data is used.
16. No mock responses are used.
17. All features use real data.
18. The system runs end-to-end without critical errors.

---

## 13. Execution instructions for AI implementation

- Read this PRD as the single source of truth.
- Target production-ready completion.
- Do not use dummy data.
- Do not use mock responses.
- Do not use hardcoded signals.
- Do not use random signals.
- All data must come from real APIs.
- Do not make assumptions when the specification is missing.
- Document ambiguities first.
- Verify each implementation before moving to the next phase.
- Do not delete files without a clear reason.
- Every major change must include a report.

### Execution phases

#### Phase 1 â€” Audit

Audit the entire repository:

- project structure
- dependencies
- environment variables
- Firebase
- backend
- frontend
- mobile
- deployment

Create:

- `docs/AUDIT_REPORT.md`
- `docs/GAP_ANALYSIS.md`

Continue only after the audit is complete.

#### Phase 2 â€” Architecture validation

Compare the repository against this PRD.

Verify:

- architecture
- database
- API
- scheduler
- Firebase
- Telegram
- Gemini
- market data

Create:

- `docs/ARCHITECTURE_REVIEW.md`

#### Phase 3 â€” Backend implementation

Implement the full backend according to this PRD.

Required modules:

- Flask
- APScheduler
- Firestore
- Gemini validation
- TwelveData integration
- Yahoo Finance fallback
- NewsAPI integration
- GNews integration
- Telegram integration
- Signal engine
- BOS engine
- CHOCH engine
- FVG engine
- Fibonacci engine
- ATR engine
- Confidence engine

Create unit tests for every module.

#### Phase 4 â€” Firestore

Verify:

- connection
- collection design
- document schema

Implement the full schema according to this PRD.

Create a database verification script.

#### Phase 5 â€” API

Implement every endpoint listed in this PRD.

Add:

- validation
- error handling
- logging

Create API documentation.

#### Phase 6 â€” Frontend

Implement the dashboard according to this PRD.

The frontend must include:

- system status
- market status
- killzone status
- news status
- latest signal
- signal history
- manual scan
- connection status

The frontend must connect to the real backend.

#### Phase 7 â€” Mobile

Implement Capacitor.

Ensure the app can be built into an APK.

#### Phase 8 â€” Security

Verify:

- environment variables
- secret management
- Firebase credentials
- API keys

Ensure no credentials are stored in the repository.

Update `.gitignore` if needed.

#### Phase 9 â€” Testing

Run:

- backend tests
- frontend tests
- Firestore tests
- Telegram tests
- Gemini tests
- scheduler tests
- deployment tests

Create:

- `docs/TEST_REPORT.md`

#### Phase 10 â€” Deployment

Prepare:

- Railway deployment
- Vercel deployment

Create:

- `docs/DEPLOYMENT_GUIDE.md`

Ensure the production build succeeds.

#### Phase 11 â€” Final verification

Verify the entire Definition of Done.

Create:

- `docs/FINAL_VERIFICATION.md`

### Execution rules

- Work step by step.
- Do not jump to deployment before testing is complete.
- Do not mark a task as done without verification.
- After each phase, write a report.
- If a blocker is found, document it and continue with unblocked work.
- Prioritize stability, accuracy, and PRD compliance.
- All implementations must use real data and real services as specified in this PRD.
- Always include comprehensive unit tests for every new feature.

---

## 14. Change policy

Any future change must preserve these rules:

- real data only
- no synthetic signals
- no mock services in production
- no chart requirement in frontend analytics
- no silent behavior changes
- no bypass of validation gates

---

## 15. Notes

This PRD is intentionally strict.

The system is expected to behave like a production financial operations tool, not a demo.

Any implementation detail not defined here must be documented before coding begins.