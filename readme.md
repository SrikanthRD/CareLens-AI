# Secure EHR Insight – Clinical Validator

A privacy-first pipeline for answering clinical questions over Electronic Health Records (EHRs). Patient identifiers are masked before anything is stored or sent to a model, answers are grounded in retrieved records, and every response is validated against safety rules before it reaches the user.

> **Status:** demo / educational project. Use only synthetic or properly de-identified data. This is not a medical device and must not be used for real clinical decisions.

## How it works

![Alt text](image.png)

**Offline: ingestion and de-identification**

1. Load EHR records from `data/` (notes and tables).
2. Prepare the data with the helper scripts in `scripts/`.
3. Detect and mask personal data (names, IDs, dates, contact details) with Presidio and spaCy.
4. Convert the cleaned text to vectors with sentence-transformers.
5. Store the vectors in PostgreSQL with pgvector. No raw identifiers are stored.

**Online: question answering**

1. A clinician asks a question in the Streamlit UI.
2. FastAPI receives the request.
3. An input guardrail (NeMo Guardrails, plus Presidio on the query) screens the question.
4. The retriever embeds the question and finds the closest records in pgvector.
5. The LLM drafts an answer using only the retrieved records.
6. An output validator checks safety and PII. A failed check blocks or rewrites the answer.

## Design principles

- **Privacy first:** raw identifiers never reach the vector store or the model.
- **Grounded answers:** responses must rest on retrieved records.
- **Fail safe:** a failed validation check blocks or rewrites the answer.

## Repository structure

| Path | Purpose |
|---|---|
| `data/` | Sample or synthetic clinical records |
| `scripts/` | Helper scripts (data loading, embedding, demos) |
| `src/` | Application code: API, UI, privacy, retrieval and validation logic |
| `instructor_notes/` | Teaching notes for workshops and demos |
| `requirements.txt` | Pinned Python dependencies |
| `uv_instructions.txt` | Notes for creating the environment with `uv` |

## Tech stack

| Layer | Packages |
|---|---|
| API | `fastapi`, `uvicorn` |
| UI | `streamlit` |
| Database | PostgreSQL, `pgvector`, `psycopg`, `sqlalchemy` |
| AI / ML | `sentence-transformers`, `transformers`, `torch`, `huggingface-hub` |
| Privacy and safety | `presidio-analyzer`, `presidio-anonymizer`, `nemoguardrails` |
| NLP | `spacy`, `en-core-web-lg` |
| Utilities | `pandas`, `numpy`, `python-dotenv`, `requests` |

## Getting started

### Prerequisites

- Python 3.12
- [`uv`](https://docs.astral.sh/uv/)
- PostgreSQL with the `pgvector` extension
- Several GB of disk space (`torch`, `transformers` and the spaCy large model are big)

### Install

```bash
# create and activate the environment
uv venv --python 3.12 .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# install dependencies
uv pip install -r requirements.txt
```

### Configure

Create a `.env` file in the project root:

```bash
# TODO: replace with the variables this project actually reads
DATABASE_URL=postgresql+psycopg://user:password@localhost:5432/ehr
```

### Run

```bash
# TODO: replace with the real entry points in src/
uvicorn src.main:app --reload     # API
streamlit run src/app.py          # UI
```

## Security and privacy notes

- Never commit real patient data or secrets. Keep credentials in `.env` and make sure it is in `.gitignore`.
- Automated PII detection is not perfect. Review its output before trusting it with real data.
- Guardrails reduce risk but do not remove it. Keep a human in the loop for anything clinical.

## License

WIP
