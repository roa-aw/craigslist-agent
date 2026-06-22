# Craigslist Vehicle Sales Agent

An autonomous AI sales agent that answers natural-language questions about a real used-car inventory. Built with OpenAI function calling + a SQLite inventory layer from the Craigslist Cars & Trucks dataset.

## Project structure

```
craigslist-agent/
├── data/               # Place vehicles.csv here (from Kaggle)
├── db/
│   ├── __init__.py     # DB connection helper
│   └── setup.py        # CSV → SQLite pipeline
├── tools/
│   ├── __init__.py     # Tool definitions (OpenAI schemas + dispatch map)
│   └── search.py       # search_inventory(), get_similar(), get_makes(), get_price_range()
├── agent/
│   ├── __init__.py
│   └── agent.py        # SalesAgent class – LLM + tool loop
├── cli.py              # Multi-turn CLI entry point
├── requirements.txt
└── .env.example
```

## Setup

### 1. Clone and install dependencies

```bash
git clone <your-repo-url>
cd craigslist-agent
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

### 2. Get the dataset

Download `vehicles.csv` from [Kaggle](https://www.kaggle.com/datasets/austinreese/craigslist-carstrucks-data) and place it at:

```
data/vehicles.csv
```

### 3. Build the inventory database

```bash
python -m db.setup
```

This cleans ~400k rows and writes `db/inventory.db` (takes ~10 seconds).

### 4. Add your OpenAI API key

```bash
cp .env.example .env
# edit .env and paste your key
```

### 5. Run the agent

```bash
python cli.py
# or use a smarter model:
python cli.py --model gpt-4o
```

## Example conversations

```
You: Do you have any Honda Civics under $15,000?
You: Show me trucks from 2015 or newer in good condition
You: What's the average price of a used Toyota Tacoma?
You: Find me a blue Ford F-150 in California
You: Nothing under $20k? What about similar options?
```

## Tools available to the agent

| Tool | Purpose |
|---|---|
| `search_inventory()` | Main search — filters by make, model, price, year, condition, color, region, etc. |
| `get_similar()` | Fuzzy fallback when exact search returns 0 results |
| `get_makes()` | List all available manufacturers |
| `get_price_range()` | Min/max/avg price for a make or model |

## How it works

1. User types a natural-language query
2. OpenAI GPT decides which tool to call and extracts parameters
3. The tool queries SQLite and returns structured JSON
4. GPT converts the JSON into a friendly, readable response
5. Conversation history is maintained for multi-turn context

## Suggested Git commit sequence

```
init: project scaffold and requirements
data: CSV cleaning and SQLite setup pipeline
tools: implement search_inventory with multi-filter support
tools: add get_similar, get_makes, get_price_range
agent: OpenAI function-calling loop with history
cli: multi-turn rich CLI with spinner and markdown rendering
polish: no-results fallback, clarification handling, README
```
