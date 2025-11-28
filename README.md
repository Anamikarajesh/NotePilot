# Learning AI Agent

An interactive FastAPI application that wraps a LangGraph-powered LangChain agent so you can read from and write to local text files via natural-language prompts. The included single-page UI lets you send prompts to the agent and see responses without leaving the browser.

## Features
- **Note-focused agent** – Uses `ChatOpenAI` with a ReAct workflow to interpret prompts and decide when to read or write files.
- **Built-in tools** – Includes `read_note` and `write_note` utilities for working with any UTF-8 text file path the process can access.
- **FastAPI backend** – Serves both the API (`/agent`) and a lightweight HTML interface at `/`.
- **Environment-driven config** – Loads credentials and settings from a `.env` file via `python-dotenv`.

## Project structure
```
├── agent.py          # Agent + tool definitions
├── main.py           # FastAPI app exposing the web UI and /agent endpoint
├── templates/
│   └── index.html    # Minimal front-end for sending prompts
├── public/           # (Optional) place static assets here if needed
├── requirements.txt  # Python dependencies
└── note.txt          # Example note file the agent can edit
```

## Prerequisites
- Python 3.10 or newer
- An OpenAI API key with access to `gpt-4` (or whichever model you configure)

## Setup
1. **Clone the repo & enter it**
   ```powershell
   git clone https://github.com/techwithtim/BuildAndDeployAIAgent.git
   cd BuildAndDeployAIAgent
   ```
2. **Create a virtual environment (recommended)**
   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```
3. **Install dependencies**
   ```powershell
   pip install -r requirements.txt
   ```
4. **Configure environment variables** – create a `.env` file in the project root and add at least:
   ```ini
   OPENAI_API_KEY=sk-your-key
   # Optional: override model/temperature if desired
   OPENAI_MODEL=gpt-4
   OPENAI_TEMPERATURE=0
   ```
   `agent.py` currently hardcodes `gpt-4` and `temperature=0`. Update the file or extend the `.env` loading logic if you want to make those settings dynamic.

## Running locally
```powershell
uvicorn main:app --reload --port 8000
```
Then open `http://127.0.0.1:8000/` to reach the UI. The form posts prompts to `/agent` and streams the agent response into the page.

## API reference
| Method | Path    | Description                                   |
|--------|---------|-----------------------------------------------|
| GET    | `/`     | Serves `templates/index.html` UI.             |
| POST   | `/agent`| Invokes the agent with JSON `{"prompt": "..."}` |

Example request:
```bash
curl -X POST http://127.0.0.1:8000/agent \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Summarize note.txt"}'
```
Successful responses return:
```json
{
  "response": "Contents of 'note.txt': ..."
}
```
Errors use FastAPI's standard format, e.g. HTTP 400 for empty prompts or HTTP 500 if the agent raises.

## Notes & limitations
- `write_note` overwrites the entire file. Add your own append/merge strategy if needed.
- The process can only access files the OS user has permission to read/write.
- Long-running or recursive tool use is capped with `recursion_limit=50` in `agent.py`.
- For production use, add authentication, rate limiting, logging, and persistence beyond plain text files.

## Next steps
- Parameterize the OpenAI model and temperature via env vars.
- Add additional LangChain tools (e.g., directory listing, note history).
- Wrap the FastAPI app with a frontend build system or component framework for richer UX.
