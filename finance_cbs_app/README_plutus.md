# Plutus — Personal Finance Management App

A full-stack personal finance platform built with modern web technologies. Plutus tracks expenses, income, investments, and net assets, and layers on AI-powered features (RAG-based categorization, on-demand Gemini summaries) and human-in-the-loop automation via Slack.

> Internal codename: `project-2b-cbs` (CBS = Cashbook System).

## 🚀 Tech Stack

### Frontend
- **Next.js 16** with Turbopack and TypeScript
- **React 19** with Context API for state management
- **Tailwind CSS 4** for responsive UI design
- **Firebase Authentication** with Google OAuth
- **Recharts** for data visualization
- **@dnd-kit** for drag-and-drop ordering (e.g. Quick Add patterns)
- **react-dropzone / papaparse** for CSV import flows

### Backend
- **FastAPI** (Python **3.12**) for REST API
- **slowapi** for rate limiting
- **Google BigQuery** as the primary data warehouse (DML-based writes)
- **Firestore** for vector embeddings, Quick Add patterns, Amazon order data, and Slack approval state
- **Vertex AI** text embeddings (`gemini-embedding-001`, MRL-truncated to 768 dims)
- **Gemini API** for category prediction and on-demand summaries
- **Slack SDK** for Slack Bot interactions and HITL approvals

### Infrastructure & Deployment
- **Google Cloud Run** (services + Jobs) for containerized deployment
- **Docker** multi-stage builds
- **Google Cloud Build** for CI/CD
- **Cloud Scheduler** for periodic batch jobs
- **Terraform** for Infrastructure as Code (Cloud Run, BigQuery, Firestore, IAM, Secret Manager, Scheduler)
- **Firebase** for authentication

## 🎯 Key Features

### 💰 Financial Management
- **Expense Tracking**: Daily expenses with categories, subcategories, tags, and notes
- **Income Recording**: Track income across multiple sources
- **Multi-Currency Support**: JPY/USD with FX rates
- **Negative Amounts**: Refunds and adjustments
- **Prorated Months**: Multi-month transactions auto-distributed across calendar months
- **Budget Watcher**: Monthly budget tracking with alerts

### 📈 Investment Portfolio
- **Stock Transaction Management**: Buy / sell / vest, with realized P&L tracking
- **Stock Sales Modal**: FIFO-based sale matching
- **Portfolio V3**: Real-time valuation with diversification, growth, and mini performance charts
- **Cash-Out Dashboard**: Visualizes cash freed up from sales
- **Portfolio Goals & Journal**: Decision journal and goal tracking for long-term discipline
- **Multi-Currency**: USD positions converted with current FX
- **Manual Stock Price**: Override prices for illiquid or untracked symbols
- **Listings**: Asset listings (real estate, etc.) with hardened validation

### 🏦 Asset Management
- **Current Assets**: Bank accounts, savings, crypto holdings
- **Service Provider Grouping**: Organize assets by financial institution
- **Bulk Entry V2**: Streamlined asset entry UI
- **Savings Tracker**: Dedicated savings goals view

### 🔐 Security & Authentication
- **Firebase Google OAuth** with email whitelist
- **End-to-End Token Verification** (frontend + Firebase Admin SDK on backend)
- **Authentication Middleware**: All `/api/v1/*` endpoints protected via `Depends(get_current_user_id)`, which verifies the Firebase token and injects the authenticated UID
- **Row-Level Security**: Application-layer `WHERE user_id = @user_id` enforcement on every BigQuery read/write of user-owned tables; server-derived UID (not client-supplied), so cross-user access cannot be attempted even with a valid token
- **Session Management**: 30-minute idle timeout with activity detection
- **Rate Limiting**: `slowapi` per-endpoint limits

### 📊 Data Visualization
- **Interactive Charts**: Expense trends, income breakdowns, portfolio performance
- **Dashboard V2**: Redesigned dashboard with on-demand AI summaries (Gemini)

### 🤖 RAG-Based Category Prediction
- **Hybrid Prediction**: Keyword matching + vector search + LLM generation
- **Vector Search**: Vertex AI `gemini-embedding-001` embeddings (3072 → 768 dim via MRL truncation) queried through BigQuery `VECTOR_SEARCH()` native function (migrated from in-process Firestore loop for cost + latency)
- **BigQuery Context**: Historical transactions inform LLM predictions
- **Gemini API**: Final category prediction with confidence scoring
- **Feedback Loop**: User corrections feed back into the embedding store
- **Cost-Efficient**: ~$0.17 to embed the full historical dataset

### 🧮 LLM Cost & Usage Tracking
- **LLM Gateway**: Single entry point (`backend/app/services/llm_gateway_service.py`) for every Gemini text-generation and Vertex AI Embedding call — `call_gemini()` and `call_embedding()`. The 5 producers (`sentinel`, `ai_summary`, `sbi_parser`, `category_prediction`, `vector_search`) all funnel through this gateway
- **BigQuery Invocation Log**: Each call records `invocation_id`, `user_id`, `feature`, `model`, `input_tokens`, `output_tokens`, `latency_ms`, `estimated_cost_usd`, `status`, `error_message`, and trace metadata to `llm_invocations` — partitioned by `DATE(created_at)`, clustered by `user_id, feature, model`, with `partition_expiration_days = 730` (2-year TTL)
- **DML Writes**: Logs persist via `INSERT INTO` parameterized SQL (not streaming insert) — keeps the streaming buffer out of the dashboard read path
- **Fault-Tolerant Logging**: Log-write failures are caught and printed so a BQ hiccup never blocks the LLM response; LLM exceptions themselves re-raise so per-caller fallbacks (e.g., `category_prediction`'s keyword-hint fallback) still run
- **Usage Dashboard**: New **Settings → Usage** tab visualizes monthly total cost, budget ceiling + linearly projected month-end landing, cost-by-feature breakdown, model donut, daily trend chart with Cost / Tokens / Calls toggles, and a recent invocations log with latency and status pills
- **Per-User Scoping**: All aggregation queries filter on `user_id` — dashboards are naturally per-user when multi-user support lands

### 🧪 AI Output Evaluation (Golden-set + Regression Gate)
- **Golden Set**: 28-row frozen test set at `backend/evals/golden/category_prediction_golden.jsonl`, built by `backend/scripts/build_golden_set.py` from real `prediction_corrections`. Single-label duplicates compressed to most-recent; multi-label `item_name` rows kept as context-dependent ambiguity; generic descriptions like `AMAZON CO JP` excluded (card-statement string carries no product info — un-solvable for the AI)
- **Eval Runner**: `backend/evals/run_category_eval.py` runs the live `predict_category()` pipeline against every golden row and reports accuracy, per-category precision / recall / F1, per-method success rate (`keyword_match` / `approved_rule` / `bigquery_similar` / `gemini`), and latency p50 / p95. Per-description caching + `asyncio.Semaphore(5)` keeps a 28-row run under ~1 minute
- **BigQuery Storage**: Two new tables — `eval_runs` (one row per run with `run_id`, `ran_at`, `git_sha`, `model_id`, `prompt_hash`, `accuracy`, latency percentiles, `feature`, `note`) partitioned by `DATE(ran_at)`, and `eval_metrics` (per-label P/R/F1 across `category` / `subcategory` / `method` dimensions) clustered by `run_id, dimension`. INSERT INTO DML writes (no streaming buffer)
- **PR Regression Gate**: `.github/workflows/eval-on-pr.yml` triggers only when prediction-related files change (`category_prediction_service.py`, `llm_gateway_service.py`, `vector_search_service.py`, `backend/evals/**`). Runs the eval, then `backend/evals/check_regression.py` compares the PR run against the latest `main` run; merge is blocked if `accuracy` drops by ≥ 2pt. `note` field distinguishes PR runs (`PR #<num>`) from main runs
- **Pending-Transaction Persistence Fix**: Previously `enrich_pending_transactions` computed predictions in-memory only — the AI's category never persisted to BigQuery `pending_transactions`, so `save_correction` logged `predicted_category = NULL` and `prediction_method` was inferred from confidence (broken: `approved_rule` confidence 1.0 was mis-labelled as `keyword_match`, `bigquery_similar` confidence 0.8 as `gemini`). Fixed by adding `_persist_enrichment_to_bq` (UPDATE pending_transactions with `item_name`, `item_category`, `item_subcategory`, `confidence`, `notes`, `prediction_method`) and a new `prediction_method STRING` column on `pending_transactions` so all four prediction paths are recorded accurately
- **Baseline**: First run (82 rows including AMAZON noise) scored 48.8% — dominated by un-solvable AMAZON CO JP rows. After excluding generic descriptions, the 28-row clean set scores 85.7% (24/28); `bigquery_similar` 79%, `approved_rule` 90%, `keyword_match` 100%. This baseline informs the regression gate

### 🔄 Self-Healing Pipeline (Agentic RAG)
- **Auto Rule Generator**: Detects frequent user-correction patterns and proposes new merchant categorization rules
- **Slack HITL**: Approval requests delivered as interactive Slack messages with Approve/Reject buttons
- **Incremental Embedding Updates**: Monthly batch job embeds only new transactions (cost-optimized)
- **Cloud Run Jobs + Cloud Scheduler**: Fully automated weekly/monthly execution provisioned via Terraform
- **Feedback-Driven Learning**: Closes the loop between user corrections and system improvements

### 📄 BigQuery → Google Sheets Sync (Cloud Run Job)
- **Daily Export**: BigQuery transaction records synced to Google Sheets via a dedicated Cloud Run Job
- **Multi-Tab Sync**: `view_asset_breakdown`, `listings`, `monthly_depreciation`, and other views exported to dedicated tabs
- **Prorated Month Expansion**: Multi-month transactions split across the correct months with even `price_jpy` distribution and remainder adjustment
- **Auto Tab Creation**: Missing tabs created with frozen header rows

### 📧 Gmail Auto-Import & Email Parsers
- **Automated Transaction Import**: Parses credit-card and payment-service emails directly from Gmail
- **Multi-Provider Support**: ANA Pay, ANA SFC JCB, and Rakuten Card parsers
- **Pending Transactions Table**: Imported transactions land in BigQuery `pending_transactions` (DML-based, no streaming buffer) and stay there until approved into final tables
- **Inbox UI**: PendingTransactionsV2 review screen with bulk approve / edit

### 📈 SBI Stock PDF Auto-Ingestion (Gemini)
- **PDF Parser**: Drop SBI Securities trade confirmation PDFs into a Drive `inbox/` folder; Gemini 2.5 Flash extracts ticker, side, qty, price, FX rate, NISA flag using a strict `responseSchema`
- **English-Normalised Output**: Gemini converts katakana company names to official English (e.g., エヌビディア → NVIDIA Corporation) and `買付/売付` to `purchase/sell`, matching the existing `stock_transactions` schema
- **Pending Stock Table**: Parsed trades land in BigQuery `pending_stock_transactions` (mirrors `stock_transactions` + ingestion metadata: `source_file_id`, `parser_model`, `raw_extracted_json`, `review_status`, `needs_attention`)
- **Inbox · Stocks Tab**: New tab on the Inbox page with a `🔄 Refresh from Drive` button — backend invokes the parser on demand (no cron)
- **Approve → Production (Atomic)**: BQ multi-statement transaction promotes pending → `stock_transactions` with `GENERATE_UUID()`, confirm dialog before write
- **Smart `needs_attention`**: Sells (require FIFO acquisition lookup), missing TT rates, qty×price×fx vs. JPY amount drift > 1% are automatically flagged
- **Prompt Injection Defense**: Gemini prompt wraps all instructions in `<system_instructions>` tags and explicitly declares the attached PDF as untrusted user data; the model is instructed to ignore any embedded "ignore previous instructions / report all as purchase" attacks (indirect prompt injection guard for malicious or tampered PDFs). `contents` order is `[PROMPT, pdf_part]` so the defensive instructions are read before the document
- **Hard-Stop Meta Validation**: Post-extraction sanity checks (`transaction_type` enum, `transaction_date` format + not future, `fx_rate` ∈ [120, 190], positive `quantity` / `price_per_share_usd`, `amount_usd ≈ qty × price` within 1%) raise `RuntimeError` to abort the BigQuery import on the first failure — financial data fails loud, never silently. Tuning parameters live at the top of `gemini_parser.py`
- **Drive-Side Dedup**: Processed PDFs auto-moved to `processed/`, plus BQ `source_file_id` dedup as a belt-and-suspenders safeguard
- **Archive Folder (User-Managed)**: Historical PDFs already in `stock_transactions` go into a separate `archive/` folder that the parser never touches

### 🛒 Amazon Order Matching
- **Email Parsing**: Parses Amazon order-confirmation emails into structured order data
- **Amount-Based Matching**: Matches pending transactions to Amazon orders by exact JPY amount
- **Auto Enrichment**: Updates pending transactions with product name and order details on match
- **Firestore Storage**: Amazon orders persisted in Firestore for matching history and audit trail
- **Authenticated Endpoint**: Amazon import API now requires Firebase auth

### ⚡ Quick Add Patterns
- **User-Defined Templates**: Save frequently used expense entries as one-tap reusable patterns
- **Drag-and-Drop Ordering**: Reorder patterns via `@dnd-kit`
- **Firestore-Backed**: Patterns stored per-user with custom sort ordering
- **Fast Entry**: Pre-fills the entry form to minimize repetitive input

### 📱 Mobile-First UI (V2 / V3)
- **V2 Redesign** across all sections (Daily Entry, Stock Entry, Sales Entry, Asset Entry, Listings, Pending Transactions, Navigation, Header, Login)
- **Hamburger Drawer Navigation** for mobile viewports
- **Plutus Brand**: Theme rebranded from Mintledger to Plutus (login, navigation, branded entry forms)

### 📈 Stock Price Fetchers (local cron / scheduler)
- **`latest_stock_price_fetcher.py`**: Daily prices via Alpha Vantage (GOOG, AAPL, MSFT, META, AMZN, NVDA, PLTR, COHR, TSM), scheduled at 06:30 JST
- **`stock_price_with_fx_fetcher.py`**: Historical prices with FX
- **`stock_price_latest_eom_fetcher.py`**: End-of-month price snapshots
- All write directly to BigQuery for portfolio valuation

### 💬 Slack Integration
- **Slack Notifications**: Rule proposals and alerts
- **Slack Bot**: Interactive approval flow for self-healing rule proposals

## 🏗️ Architecture

```
Frontend (Next.js 16)        Backend (FastAPI / Python 3.12)        Data Layer
┌─────────────────────┐      ┌──────────────────────────────┐      ┌────────────────────────┐
│ - React 19 + TS     │      │ - REST API (rate-limited)    │      │ BigQuery               │
│ - Firebase Auth     │ ───▶ │ - Firebase token middleware  │ ───▶ │  - records / stocks    │
│ - Tailwind CSS 4    │      │ - RAG Category Predictor     │      │  - pending_transactions│
│ - Recharts / dnd-kit│      │ - Gemini AI summaries        │      │  - listings / assets   │
└─────────────────────┘      │ - Gmail / Amazon parsers     │      │  - analytics views     │
                             │ - Slack Bot (HITL)           │      ├────────────────────────┤
                             └──────────────────────────────┘      │ Firestore              │
                                          │                        │  - vector embeddings   │
                                          │                        │  - quick add patterns  │
                                          ▼                        │  - amazon orders       │
                             ┌──────────────────────────────┐      └────────────────────────┘
                             │  Google Cloud Platform       │
                             │ - Cloud Run (api + frontend) │
                             │ - Cloud Run Jobs (sync, RAG) │
                             │ - Cloud Scheduler            │
                             │ - Cloud Build (CI/CD)        │
                             │ - Secret Manager             │
                             │ - Vertex AI / Gemini         │
                             │ - Terraform-managed IaC      │
                             └──────────────────────────────┘
```

## 💡 Architecture Decision: BigQuery + Firestore (no traditional RDB)

Plutus deliberately avoids running a 24/7 relational database. Instead it splits storage by access pattern:

- **BigQuery** (analytical): expenses, income, stock transactions, listings, current assets, pending transactions, all analytical views. Pay-per-query, columnar, serverless.
- **Firestore** (low-latency document): vector embeddings, Quick Add patterns, Amazon orders, Slack approval state. Single-digit-ms reads for UI hot paths.

For a personal-finance app with moderate volume, this combination is significantly cheaper than maintaining a persistent RDB instance while still serving sub-second UI interactions.

> Note: BigQuery writes were migrated from streaming insert to **DML** to eliminate the streaming buffer's "no UPDATE/DELETE for 90 minutes" constraint, which was blocking the pending-transactions approve flow.

> **Atomicity Note**: Multi-table writes (`daily_transaction_records` + `pending_transactions` UPDATE + `fixed_assets` / `travel_expenses` INSERT) are wrapped in **BigQuery multi-statement transactions** (`BEGIN TRANSACTION ... COMMIT TRANSACTION` with `EXCEPTION WHEN ERROR THEN ROLLBACK; RAISE;`). A `run_transactional_script()` helper in `backend/app/utils/bq_writer.py` guarantees all-or-nothing semantics — replacing the previous `asyncio.gather()` parallel-execution pattern that could leave partial state on failure.

> **Optimistic Locking Note**: Editable tables (`pending_stock_transactions`, `daily_transaction_records`, `listings`) carry a monotonic `version INT64` column. PATCH/PUT endpoints require the client to send `expected_version`; the SQL `WHERE version = @expected_version AND ...` gate fails atomically when a concurrent tab has already saved, and the API returns **HTTP 409 Conflict** instead of silently overwriting. Frontends (`PendingStocksInboxV2`, `ExpenseEditModal`, `ListingsManager`) surface the conflict as a non-modal "再読込" banner so the user can refetch and retry. INSERT-only tables (`pending_transactions`, `stock_transactions`, `current_assets`, `fixed_assets`) carry the same `version` column for forward compatibility, seeded to `1` on every insert.

> **Idempotency-Key Note**: To prevent duplicate INSERTs on flaky-network submits (user retapping a slow submit, fetch network-error retries), every authenticated POST/PUT/PATCH automatically carries an `Idempotency-Key: <uuid v4>` header. A pure-ASGI middleware (`backend/app/middleware/idempotency.py`) intercepts the header, atomically creates an `in_flight` document at `idempotency_keys/{user_id}_{key}` in Firestore via `doc_ref.create()`, runs the inner handler, then upserts the captured response (status + body) back into the document with a 24h `expires_at` TTL. Replays with the same key short-circuit straight from Firestore (no BigQuery write) and return the cached body plus an `X-Idempotency-Replay: true` header. Body-mismatch returns 422 (key reuse with different payload), in-flight collisions return 409. The frontend `apiCall()` wrapper (`frontend/src/lib/api.ts`) generates the UUID once per mutating call and reuses it across automatic retries (`MAX_RETRY_ATTEMPTS=3`, exponential backoff) so a single submit intent stays idempotent across network blips.

> **Row-Level Security Note**: Defense-in-depth on top of the Firebase email whitelist. Every editable BigQuery table (`daily_transaction_records`, `pending_transactions`, `fixed_assets`, `listings`, `sales`, `stock_transactions`, `pending_stock_transactions`, `current_assets`, `cooling_off_items`, `trim_items`, `portfolio_events_journal`, `portfolio_goals`, `manual_stock_prices`, `travel_expenses`, `savings_goals`, `budget_entries`) carries a `user_id STRING` column. A `get_current_user_id` FastAPI dependency extracts the verified Firebase UID from the bearer token and is injected into every endpoint; the service layer then enforces `WHERE user_id = @user_id` on every SELECT/UPDATE/DELETE and binds `user_id` on every INSERT (`MERGE` ON clauses include `user_id` too, so the same ticker symbol can belong to different users). The Slack webhook (which bypasses Firebase auth via signing verification) reads the data owner UID from `SLACK_OWNER_USER_ID` env var. This is application-layer enforcement (not BQ `ROW ACCESS POLICY`); single-user analytical views (`view_savings_summary`, `view_portfolio_holdings`, `monthly_expenses_prorated`, etc.) still aggregate without a `user_id` filter and will be reworked when multi-user support lands.

> **LLM Gateway Note**: All Gemini text-generation and Vertex AI Embedding traffic flows through a single `backend/app/services/llm_gateway_service.py` — `call_gemini()` for `generate_content`, `call_embedding()` for Vertex AI predict. Each call wraps the SDK invocation in `try/except/finally` to capture `usage_metadata` (input/output/cached token counts), elapsed `latency_ms`, and `status` (`success` / `error`), then writes a row to BigQuery `llm_invocations` via `INSERT INTO` parameterized DML — NOT `insert_rows_json` (the streaming buffer would otherwise gate the Usage dashboard read path). USD cost is computed from a per-model `PRICING` map (`input_per_1m_usd` / `output_per_1m_usd`) and stored in `estimated_cost_usd`. Log-write exceptions are caught and printed so a BigQuery hiccup never blocks the LLM response; LLM exceptions themselves re-raise so existing caller fallbacks (e.g., `category_prediction`'s keyword-hint fallback) still execute. The table is `PARTITION BY DATE(created_at) CLUSTER BY user_id, feature, model` with `partition_expiration_days = 730`. Batch jobs without an HTTP user context (SBI PDF parser, embedding backfill scripts) bind their `user_id` from env (`SBI_PARSER_USER_ID`, `SCRIPT_USER_ID`) so the same dashboard can attribute every USD spent.

## 🔧 Technical Highlights

### Frontend Engineering
- **Type Safety**: Full TypeScript with strict typing
- **State Management**: React Context with custom hooks
- **Component Architecture**: Reusable, modular components (V2/V3 family)
- **Mobile-First**: Responsive design with a hamburger drawer for small viewports
- **Performance**: Turbopack-powered dev and build pipelines
- **Resilient API Client**: `apiCall()` wrapper auto-mints `Idempotency-Key` UUIDs on every POST/PUT/PATCH and reuses the same key across 3-attempt network-error retries with exponential backoff — a single submit intent stays idempotent end-to-end

### Backend Engineering
- **RESTful API**: Versioned `/api/v1` endpoints with Pydantic validation
- **Authentication Middleware**: FastAPI dependency injection for Firebase token verification, with `get_current_user_id` injecting the verified UID into every protected handler
- **Row-Level Security**: Service layer enforces `WHERE user_id = @user_id` on every SELECT/UPDATE/DELETE and binds `user_id` on every INSERT for the 16 user-owned BigQuery tables; defense-in-depth on top of the email whitelist
- **LLM Gateway**: Every Gemini / Vertex AI call funnels through `llm_gateway_service.py` (`call_gemini` / `call_embedding`); the gateway logs token counts, latency, USD cost, and status to BigQuery `llm_invocations` via INSERT INTO DML, swallows log failures so they never block the LLM response, and re-raises LLM exceptions so per-caller fallbacks still run
- **Rate Limiting**: `slowapi` per-route limits
- **BigQuery DML**: Writes via `INSERT/UPDATE/DELETE` instead of streaming insert
- **Atomic Multi-Statement Transactions**: Cross-table writes wrapped in `BEGIN TRANSACTION ... COMMIT` with `EXCEPTION ... ROLLBACK; RAISE;` for all-or-nothing semantics (record approve, bulk insert, asset+pending writes)
- **Optimistic Locking**: `version INT64` column on every BigQuery table; PATCH/PUT endpoints gate on `expected_version` and return HTTP 409 on mismatch, eliminating lost-update races between concurrent tabs
- **Idempotency-Key Middleware**: Pure-ASGI middleware auto-protects every authenticated POST/PUT/PATCH against duplicate INSERTs on flaky-network submits. Firestore-backed key store (`idempotency_keys` collection, 24h TTL) caches the first response and replays it byte-for-byte on retry, never re-running the BigQuery write
- **Cost-Aware Design**: Serverless-first architecture
- **RAG Implementation**: Hybrid prediction (vector + BigQuery + LLM)
- **Self-Healing Pipeline**: Auto-rule generation with Slack HITL approval workflow
- **Email Parsers**: Pluggable parser architecture under `app/services/parsers/`

### DevOps & Infrastructure
- **Infrastructure as Code**: 100% Terraform (Cloud Run, BigQuery, Firestore, IAM, Secret Manager, Scheduler)
- **Containerization**: Docker (Python 3.12 slim base, non-root user)
- **CI/CD**: Cloud Build pipeline (build → push → deploy backend + sheets_sync Job)
- **Secret Management**: Google Secret Manager wired via `--update-secrets` in `cloudbuild.yaml`
- **Scheduled Jobs**: Cloud Run Jobs + Cloud Scheduler for sync, embedding refresh, rule generation
- **Monitoring**: Health check endpoint + structured logging

## 🔒 Firebase Authentication Implementation

### Architecture Overview
End-to-end Firebase authentication protects both frontend access and every backend API endpoint.

### Frontend Authentication (`frontend/`)
- `src/lib/firebase.ts` — Firebase SDK initialization
- `src/contexts/AuthContext.tsx` — Auth state + 30-min idle timeout
- `src/lib/api.ts` — `apiCall()` helper with automatic `Authorization: Bearer <id-token>` injection
- `src/components/Login.tsx` — Plutus-branded Google OAuth login screen

### Backend Authentication (`backend/`)
- `app/config/firebase_config.py` — Firebase Admin SDK initialization
- `app/dependencies/auth.py` — Token verification + email whitelist enforcement + `get_current_user_id` UID extractor
- All endpoint files — Protected with `Depends(get_current_user_id)`; the dependency runs `verify_firebase_token` as a sub-dependency and returns the verified UID for service-layer filtering

```python
# Authentication middleware (simplified)
async def verify_firebase_token(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> dict:
    decoded_token = get_firebase_auth().verify_id_token(credentials.credentials)
    if decoded_token.get('email') not in ALLOWED_EMAILS:
        raise HTTPException(status_code=403, detail="Access denied")
    return decoded_token

async def get_current_user_id(
    token_data: dict = Depends(verify_firebase_token),
) -> str:
    return token_data['uid']

@router.get("/api/v1/records")
async def get_records(user_id: str = Depends(get_current_user_id)):
    # service layer applies WHERE user_id = @user_id
    return await record_service.get_transaction_records(user_id=user_id)
```

### Security Features
1. **Email Whitelist**: Only the configured `ALLOWED_EMAIL` can access the system (coarse-grained authorization)
2. **Token Verification**: Every API request verified by Firebase Admin SDK
3. **Row-Level Security**: Application-layer `WHERE user_id = @user_id` on every BigQuery read/write of user-owned tables; the UID is server-derived from the verified token (clients cannot spoof it)
4. **Session Management**: 30-minute idle timeout on the frontend
5. **No Public Access**: All `/api/v1/*` endpoints require valid authentication
6. **Rate Limiting**: `slowapi` protects against abuse
7. **Secret Manager**: Credentials injected at deploy time, never committed

### Cloud Run Secrets (via `cloudbuild.yaml`)
- `FIREBASE_ADMIN_CREDENTIALS_JSON` — Firebase service account
- `GEMINI_API_KEY` — Gemini API key
- `DISCORD_WEBHOOK_URL` — Legacy notification channel
- `SLACK_BOT_TOKEN` / `SLACK_CHANNEL_ID` / `SLACK_SIGNING_SECRET` — Slack Bot integration

## 🎯 Skills Demonstrated
- **Full-Stack Development**: End-to-end product engineering
- **Modern Web**: React 19, Next.js 16, TypeScript, Tailwind CSS 4
- **Backend**: Python 3.12, FastAPI, BigQuery, Firestore, Google Cloud APIs
- **AI/ML Integration**: RAG, vector search, LLM orchestration, agentic workflows, self-healing pipelines
- **Cloud Engineering**: GCP, Docker, Cloud Run, Cloud Run Jobs, Cloud Build, serverless architecture
- **Infrastructure as Code**: Terraform across compute, data, IAM, and secrets
- **Authentication & Security**: Firebase, OAuth, rate limiting, session management
- **Data Engineering**: BigQuery DML, vector embeddings, cost-optimized analytics
- **API Design**: Versioned RESTful endpoints with Pydantic schemas
- **DevOps**: Containerization, CI/CD, scheduled jobs, observability

---

**Note**: Plutus is a personal finance management application built to demonstrate full-stack engineering, cloud-native architecture, AI integration, and end-to-end security with Firebase authentication.
