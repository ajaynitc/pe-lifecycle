# AI Assisted Performance Testing — PE LifeCycle

A browser-based chat UI for AI-driven JMeter performance testing, backed by an n8n multi-agent workflow.

## Live demo

Hosted on GitHub Pages: `https://<your-username>.github.io/pe-lifecycle/`

## Features

- 5 AI agents: Script Development, Test Designer, Executor, Report Generator, Analyzer
- Dual HAR recording flow for automatic JMeter correlation detection
- File upload: Postman collections, HAR files, JMX scripts
- Activity log with per-agent filtering and export
- Settings panel with live webhook connection test
- Hybrid storage: Postgres (structured) + Qdrant (vector/semantic)
- Dark mode support

## Setup

### 1. Host the UI (GitHub Pages)

Fork or clone this repo, then enable GitHub Pages in Settings → Pages → Source: `main` branch, `/ (root)`.

Your UI will be live at `https://<your-username>.github.io/<repo-name>/`.

### 2. Deploy the n8n workflow

Import `pe_lifecycle_n8n_workflow_v2.json` into your n8n instance and activate it.

Add credentials:
- **Anthropic API key** → 5 LLM nodes (claude-sonnet-4-5)
- **OpenAI API key** → 2 Embeddings nodes (text-embedding-3-small)
- **Qdrant URL + API key** → 2 Qdrant nodes

Create the Postgres table:
```sql
CREATE TABLE pe_test_runs (
  id SERIAL PRIMARY KEY,
  conversation_id TEXT,
  run_id TEXT,
  run_timestamp TIMESTAMPTZ DEFAULT NOW(),
  test_type TEXT,
  scenario_config JSONB,
  metrics_summary JSONB,
  artifacts_json JSONB,
  observations_text TEXT
);
```

Create the Qdrant collection (vector size 1536 for text-embedding-3-small):
```bash
curl -X PUT "https://your-qdrant-instance/collections/pe_test_runs" \
  -H "Content-Type: application/json" \
  -d '{"vectors": {"size": 1536, "distance": "Cosine"}}'
```

### 3. Connect the UI to n8n

Open the live GitHub Pages URL in your browser, click **Settings** (gear icon), paste your n8n webhook URL, and click **Save & test**.

Your webhook URL format: `https://your-n8n-instance.com/webhook/pe-lifecycle`

### 4. CORS

If you see CORS errors in the browser console, add the following to your n8n reverse proxy (nginx example):

```nginx
add_header Access-Control-Allow-Origin *;
add_header Access-Control-Allow-Methods "POST, OPTIONS";
add_header Access-Control-Allow-Headers "Content-Type";
```

Or handle OPTIONS preflight in n8n with a second webhook node on the same path.

## Files

| File | Description |
|---|---|
| `index.html` | Complete standalone UI — open directly in any browser |
| `pe_lifecycle_n8n_workflow_v2.json` | n8n workflow with hybrid Postgres + Qdrant storage |
| `README.md` | This file |

## Webhook payload format

**Chat message (JSON):**
```json
{
  "conversation_id": "conv_1234567890",
  "agent": "agent_1",
  "message": "Generate a JMeter script for my API",
  "history": [{"role": "user", "text": "..."}, {"role": "agent", "text": "..."}],
  "timestamp": "2026-04-29T10:00:00Z"
}
```

**File upload (multipart/form-data):**
- `payload` field: JSON string (same structure as above)
- `files` field(s): attached files (.har, .json, .jmx, etc.)

**Expected response:**
```json
{
  "response": "Agent reply text",
  "next_actions": ["Action 1", "Action 2", "Action 3"],
  "changes": ["Change log entry 1", "Change log entry 2"]
}
```
