# GameSense

GameSense is a multi-sport intelligence platform that provides explainable, data-driven insights into team and player performance. The project focuses on soccer and the NFL, delivering structured analysis that helps fans understand *why* teams and players perform the way they do, while also supporting informed decision-making in fantasy sports and betting-adjacent contexts.

Rather than producing opaque predictions, GameSense emphasizes transparency and context. Insights are derived from historical performance data, matchup characteristics, and trend analysis, and are presented in a clear, human-readable format.

## Supported Sports

- Soccer
- NFL

The system is designed to be sport-agnostic, allowing additional leagues and sports to be added with minimal architectural changes.

## Core Features

- Team performance analysis with form and trend tracking
- Player-level insights based on role, usage, and consistency
- Matchup previews highlighting stylistic and contextual factors
- Rolling metrics and historical splits (home/away, opponent strength, etc.)
- Explainable insights designed for fans, analysts, and fantasy/betting users
- Conversational copilot interface for natural-language queries (start/sit, trade evaluation, waiver recommendations)
- Precomputed analytics for fast, scalable API responses

## Architecture Overview

GameSense follows a separation-of-concerns architecture optimized for analytics-heavy workloads:

- **Analytics & ingestion workers** perform data collection, feature engineering, and metric computation offline.
- **REST API layer** serves precomputed insights efficiently and handles conversational queries via Claude tool use.
- **Web frontend** renders structured insight pages, visualizations, and a copilot chat interface.

This design avoids expensive real-time computation and enables scalable insight delivery.

### Copilot Integration

The conversational layer uses Claude with tool use to interpret natural-language questions and route them to the appropriate analytics endpoints:

```
┌─────────────────────┐     ┌──────────────────────┐     ┌─────────────────┐
│   Next.js Frontend  │────▶│   FastAPI Backend     │────▶│   Claude API    │
│   Chat + Dashboards │◀────│   /api/chat + /api/*  │◀────│   (tool use)    │
└─────────────────────┘     └──────┬───────────────┘     └─────────────────┘
                                   │
                            ┌──────▼───────────────┐
                            │   Data Layer          │
                            │  - PostgreSQL         │
                            │  - Redis cache        │
                            │  - Background workers │
                            └───────────────────────┘
```

1. User sends a query ("Should I start Salah or Saka this week?")
2. Backend forwards the message to Claude with analytics tools defined
3. Claude decides which tools to call (e.g., `compare_players`, `get_matchup_info`)
4. Backend executes the tool against precomputed data
5. Claude interprets the results and responds conversationally

## Tech Stack

### Frontend

- Next.js
- TypeScript
- Tailwind CSS
- Recharts / ECharts for data visualization

### Backend

- Python
- FastAPI
- Pydantic
- Anthropic SDK (Claude tool use for copilot)
- Background workers (RQ or Celery)

### Data & Infrastructure

- PostgreSQL (primary database)
- Redis (caching and job queues)
- S3-compatible object storage for datasets and artifacts
- Docker for containerization
- GitHub Actions for CI

## Copilot Tools

The conversational copilot exposes the following tools to Claude:

| Tool | Description |
|------|-------------|
| `get_player_stats` | Fetch a player's stats for a given season and week range |
| `compare_players` | Compare 2+ players for start/sit decisions with matchup context |
| `get_injury_report` | Get current injury statuses for a team or player |
| `evaluate_trade` | Analyze a proposed fantasy trade for rest-of-season value |
| `waiver_recommendations` | Surface top waiver wire pickups by position |
| `get_matchup_info` | Get team matchup details and defensive/opponent rankings |

Tools are defined as JSON schemas in `backend/tools.py` and dispatched through handlers in `backend/fantasy_data.py`. Adding a new tool requires defining its schema, implementing its handler, and registering it in the dispatch map.

## Data & Analytics Approach

- Batch ingestion of historical and contextual sports data
- Feature engineering for rolling averages, matchup indicators, and performance splits
- Precomputation of insight tables for teams, players, and matchups
- Aggressive caching of read-heavy endpoints
- Emphasis on explainability over black-box prediction

## Project Structure

```
gamesense/
├── backend/
│   ├── main.py              # FastAPI app, Claude integration, chat endpoint
│   ├── tools.py             # Tool definitions for Claude
│   ├── fantasy_data.py      # Data fetching, analysis, and tool handlers
│   ├── models.py            # Pydantic models
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── App.jsx          # Chat + dashboard interface
│   │   └── main.jsx         # Entry point
│   ├── index.html
│   ├── package.json
│   └── vite.config.js
└── README.md
```

## Getting Started

### Prerequisites

- Python 3.11+
- Node.js 18+
- An Anthropic API key ([console.anthropic.com](https://console.anthropic.com/))

### Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # Add your ANTHROPIC_API_KEY
uvicorn main:app --reload --port 8000
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Open [http://localhost:5173](http://localhost:5173).

## Data Sources

The starter includes mock data for immediate development. Production integrations:

- **Sleeper API** (free, no key) — ADP, player data, trending players
- **ESPN Fantasy API** (unofficial) — league-specific data
- **SportRadar** (paid) — comprehensive stats across sports
- **Football-Data.org** — soccer fixtures, standings, and results

See `backend/fantasy_data.py` for integration points.

## Extending the Platform

### Adding a new tool

1. Define the tool schema in `backend/tools.py`
2. Implement the handler in `backend/fantasy_data.py`
3. Register it in the tool dispatch map in `backend/main.py`

### Adding a new sport

The architecture is sport-agnostic. Add sport-specific data fetchers and tools following the same pattern. The copilot will automatically route queries to the appropriate tools.

## License

MIT
