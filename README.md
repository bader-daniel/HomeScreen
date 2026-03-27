# HomeScreen

HomeScreen is a small, configurable home dashboard that runs as a fullscreen web page and shows multiple **widgets** (electricity prices, savings graphs, Home Assistant metrics, etc.).

The goal is to be easy to extend and hack on over time, without locking into a huge framework.

---

## 1. High‑level architecture

HomeScreen is a simple web app with three layers:

1. **Frontend dashboard (browser)**
   - Single HTML page in kiosk/fullscreen on a tablet, mini‑PC, or wall‑mounted screen.
   - Uses CSS Grid/Flexbox for layout.
   - Uses JavaScript to load widget modules and refresh their data on a schedule.

2. **Backend (optional but recommended)**
   - Lightweight HTTP API (e.g. Flask/FastAPI, Express).
   - Handles:
     - Secrets (API keys, Home Assistant tokens, Avanza/Nordnet access).
     - Normalizing external data into simple JSON for widgets.
     - Caching results to avoid hammering external APIs.

3. **External services**
   - Electricity price API (Nordic market, per bidding area).
   - Home Assistant REST/WebSocket API for sensor data (soil moisture, etc.).
   - Savings data sources (Avanza, Nordnet, or exported files).

---

## 2. Widgets

HomeScreen is built around independent widgets. Each widget owns how it fetches data and how it renders itself into a tile on the dashboard.

### Electricity

Overview of electricity prices in your bidding area for **the whole day**, plus the current value.

- Shows:
  - Current spot price in your area (e.g. öre/kWh).
  - Bar or line chart of today’s hourly prices (and optionally the next 24 hours when available).
  - Simple highlight of “now” and upcoming cheap/expensive hours.
- Configuration:
  - Bidding area (e.g. `SE3`).
  - Currency / unit.
  - Refresh interval (e.g. every 5–10 minutes).

### Todos

A weekly reminder board for recurring “don’t forget” items.

- Shows:
  - A list of tasks for the current week (e.g. “take out recycling”, “watering day”, “garbage pickup”, “call parents”).
  - Optional per‑day grouping or icons for urgency.
- Data sources:
  - Simple local JSON/Markdown file.
  - Or an integration with a to‑do service (e.g. Todoist, Google Tasks) if desired later.
- Configuration:
  - Source (local file or API).
  - How many days ahead to show.

### Plants

Soil moisture and “time to water” indicators, powered by Home Assistant.

- Shows:
  - Current soil moisture for selected plants (percentage or sensor‑specific units).
  - Clear state per plant: “OK”, “getting dry soon”, “water now”.
  - Optional “last watered” timestamp if available from Home Assistant.
- Data source:
  - Home Assistant’s API (`/api/states/<entity_id>` or WebSocket streaming).
- Configuration:
  - List of Home Assistant entity IDs (e.g. `sensor.living_room_fern_moisture`).
  - Per‑plant thresholds for “dry” and “warning”.
  - Refresh interval (e.g. every 1–5 minutes).

### Saving

Current savings overview and trends, combining Avanza and Nordnet.

- Shows:
  - Current total savings (sum across configured accounts/portfolios).
  - Split by platform (Avanza vs Nordnet) and optionally by account type.
  - Graph of portfolio value over time with trend lines (e.g. last 3–12 months).
  - Simple stats such as:
    - Absolute and percentage change over selected periods.
    - Best/worst recent day or week.
- Data source:
  - Backend integration that:
    - Either talks directly to Avanza/Nordnet APIs, **or**
    - Reads exported CSV/JSON files that you update separately.
- Configuration:
  - Which accounts/portfolios to include.
  - Currency.
  - History window for graphs.
  - Refresh strategy (e.g. “update once per day” vs “on demand”).

---

## 3. Suggested tech stack

You can swap technologies, but the design assumes:

- **Frontend**
  - HTML + CSS (Grid/Flexbox for layout).
  - Vanilla JavaScript with ES modules.
  - Optional small chart library (e.g. Chart.js) for graphs.

- **Backend**
  - Your choice (Python/Flask/FastAPI or Node/Express).
  - Responsibilities:
    - Talk to external APIs (power prices, Avanza/Nordnet, Home Assistant).
    - Store and protect secrets (API keys, tokens).
    - Expose simple, widget‑friendly JSON endpoints.

If you want to start simpler, you can initially skip the backend and only build widgets that can safely call public APIs from the browser (e.g. electricity prices), then introduce a backend when you add sensitive data.

---

## 4. Project structure

Suggested layout:

```text
HomeScreen/
  README.md
  config/
    homescreen.config.json      # dashboard + widget configuration
  backend/
    app.<py|js>                 # small API server
    services/                   # integrations (power prices, HA, savings)
  frontend/
    index.html                  # main dashboard page
    css/
      homescreen.css            # layout + theme
    js/
      main.js                   # bootstraps dashboard, loads widgets
      widgets/
        electricity.js
        todos.js
        plants.js
        saving.js
  docs/
    api-notes.md                # notes on external APIs and auth

