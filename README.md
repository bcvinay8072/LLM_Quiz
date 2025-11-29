# LLM Quiz Solver Agent — Rewritten

This repository contains an autonomous agent that programmatically solves multi-step, data-focused quiz tasks by combining a language model with small, focused tools for scraping, file handling, code execution, and result submission.

Badges
- License: MIT
- Requires: Python 3.12+
- Framework: FastAPI

Table of contents
- Quick summary
- Design & components
- What it does
- Repo layout
- Setup (local / Docker)
- Configuration
- Running it
- API reference
- Tools overview
- Deployment notes
- License & credits

Quick summary
The agent accepts a quiz URL through an HTTP endpoint, inspects and navigates quiz pages, uses a language model to plan actions, runs targeted tools (scrapers, downloaders, small Python snippets), and can post answers to evaluation endpoints. It is meant as a research/demo project for automated data tasks and is configured to be run locally or inside a container.

Core ideas (short)
- Orchestrator: a state-machine driven agent controls which tools run next.
- Tools: single-purpose helpers (render web pages, fetch files, run Python, submit answers).
- LLM: used to interpret instructions, plan the toolchain, and produce code when appropriate.
- Background execution: long-running jobs are started asynchronously to avoid HTTP timeouts.
- Extensible: add or replace tools and swap the LLM backend.

Design & components
- Server: FastAPI app that exposes a /solve endpoint and a health check.
- Agent: a LangGraph-based state machine implementing the decision logic and tool orchestration.
- Tools: modular Python modules under tools/ (web renderer, downloader, code runner, poster, dependency manager).
- Storage: downloaded assets are placed under LLMFiles/ during runtime.
- LLM client: configured for Google Gemini or any compatible provider (API key via env).

What the system can do
- Render JavaScript-heavy pages (Playwright).
- Download files (PDF/CSV/images).
- Generate and execute Python code in a sandboxed subprocess to transform and analyze data.
- Submit structured answers to remote endpoints with handling for retries and failure modes.
- Install Python packages on demand when a task requires extra libraries.

Repository layout
LLM_Quiz/
- main.py — HTTP server with endpoints and background task management
- agent.py — agent orchestration logic and state machine
- pyproject.toml — project dependencies and packaging
- Dockerfile — image that bundles Playwright + runtime
- tools/
  - web_scraper.py
  - download_file.py
  - code_generate_and_run.py
  - send_request.py
  - add_dependencies.py
- LLMFiles/ — runtime-only directory for downloaded assets (not checked in)
- README.md — this file

Getting started (minimal)
1. Clone:
   git clone https://github.com/bcvinay8072/LLM_Quiz.git
   cd LLM_Quiz

2. Create and activate a virtual environment:
   python -m venv venv
   # Windows
   .\venv\Scripts\activate
   # macOS / Linux
   source venv/bin/activate

3. Install:
   pip install -e .
   playwright install chromium

Running locally
- Start the server:
  python main.py
- The default host/port is 0.0.0.0:7860 (can be changed in code/config).

Quick test (submit a job)
curl -X POST http://localhost:7860/solve \
  -H "Content-Type: application/json" \
  -d '{
    "email": "you@example.com",
    "secret": "your_secret_value",
    "url": "https://tds-llm-analysis.s-anand.net/demo"
  }'

The endpoint returns immediately with a small acknowledgement (e.g., {"status":"ok"}) while the agent processes the task in the background.

Environment variables
Create a .env file in the project root (or export the vars in your environment):

EMAIL=your.email@example.com
SECRET=your_secret_string
GOOGLE_API_KEY=your_gemini_api_key_here

Notes on the LLM key
- For Gemini: obtain a key from Google AI Studio and set GOOGLE_API_KEY.
- The code is written so that another provider can be wired in with minimal changes.

Docker
- Build:
  docker build -t llm-quiz-agent .
- Run:
  docker run -p 7860:7860 \
    -e EMAIL="you@example.com" \
    -e SECRET="your_secret" \
    -e GOOGLE_API_KEY="your_api_key" \
    llm-quiz-agent

Deploying to Hugging Face Spaces (Docker)
1. Create a new Space configured for Docker.
2. Push the repository to the Space.
3. Set the EMAIL, SECRET, and GOOGLE_API_KEY secrets in the Space settings.
4. The Space will build and run the container.

API reference
POST /solve
- Body:
  {
    "email": "you@example.com",
    "secret": "your_secret",
    "url": "https://example.com/quiz"
  }
- Responses:
  - 200 OK — accepted and agent started
  - 400 — malformed request
  - 403 — invalid secret

GET /healthz
- Small health payload, e.g. {"status":"ok","uptime_seconds":123}

Toolset (summary)
- get_rendered_html — Playwright-based page rendering
- download_file — fetches and saves external files
- run_code — executes Python code in a subprocess and returns outputs
- post_request — sends results to a submission endpoint with retries
- add_dependencies — installs packages at runtime when needed

Operational considerations
- Rate limits: the LLM client should enforce conservative throttling (example: 9 req/min as a suggested limit).
- Safety: code executed by the agent runs in a subprocess; avoid running untrusted system commands and sandbox appropriately.
- Monitoring: check logs for long-running tasks and failures; consider adding alerting around the background worker.

Auth & secrets
- The server validates a simple shared SECRET before starting a job. For production use, replace this with proper authentication and secret management.

## License
This project is licensed under the [MIT License](./LICENSE).

Author
Chenchu Vinay Boga — Tools in Data Science (TDS), IIT Madras
