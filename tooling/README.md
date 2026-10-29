# Tooling

Generate synthetic PHI notes, run deidentification, evaluate results, and visualize in a dashboard.

## Prerequisites

- Python 3.12+
- [uv](https://docs.astral.sh/uv/) package manager
- Node.js and npm (for dashboard frontend)
- AWS credentials configured (`aws configure`)
- Amazon Bedrock access to Claude Sonnet 4.5. For a new AWS account, complete
  [Anthropic's one-time use-case form](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html)
  before running the generator or deidentification.

## Environment Variables

Create `tooling/.env` with these non-secret settings (the file is ignored by Git):

```dotenv
AWS_REGION=us-east-1
BEDROCK_MODEL_ID=us.anthropic.claude-sonnet-4-5-20250929-v1:0
LOGFIRE_SEND_TO_LOGFIRE=false
```

AWS credentials are read from the terminal's standard AWS credential chain;
do not copy access keys into this file. To use a named profile, set
`export AWS_PROFILE=your-profile` in the terminal before running commands.
`AWS_REGION` must be set explicitly for the agent. Logfire export is disabled
for this local setup.

## Setup

```bash
cd tooling
uv sync --all-packages --locked
```

## Workflow

Run all commands from the `tooling/` directory.

### 1. Generate Synthetic Notes

```bash
uv run --env-file .env --package cli generate-notes --type emergency_dept --count 10
```

Creates notes in `data/synthetic_data/notes/` and manifests in `data/synthetic_data/manifests/`.

### 2. Run Deidentification

```bash
uv run --env-file .env --package cli deidentification
```

Reads from `data/synthetic_data/notes/`, writes:
- `data/output/` — raw responses
- `data/output-json/` — position data
- `data/output-text/` — redacted text

To process an existing folder of `.txt` notes:

```bash
uv run --env-file .env --package cli deidentification \
  --dataset /absolute/path/to/notes --output-dir data/output --concurrency 1
```

Check the completion log for the number of successful documents. The current
CLI can exit with status 0 even when AWS rejects every document; verify that
JSON and redacted text files were created before using the results.

### 3. Evaluate Results

```bash
uv run --env-file .env --package cli evaluate-deidentification
```

Compares predictions against ground truth manifests. Writes results to `data/eval_results/`.
Evaluation and the dashboard's annotation comparison require matching ground
truth manifests. The dashboard reads notes and results from the default paths
listed above.

### 4. Start Dashboard

View precision/recall metrics, browse false positives/negatives, and compare redactions against ground truth.

**API server:**
```bash
uv run --package api uvicorn --app-dir dashboard/api/src api.main:app \
  --host 127.0.0.1 --port 8000
```

**Frontend:**
```bash
cd dashboard/frontend
npm ci
npm run dev -- --host 127.0.0.1 --port 5173 --strictPort
```

Open http://127.0.0.1:5173. Run the API and frontend in separate terminals;
stop either server with Ctrl+C. Check the API at http://127.0.0.1:8000/api/health.
