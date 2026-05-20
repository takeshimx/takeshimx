# Hi there, I'm Takeshi 👋

### Ex-Google PgM / AI & Data Platform Operations / Full-Stack AI Engineer

**Bridging the gap between Business and Engineering with Deep Technical Depth.**

Fusing 11 years of Large-Scale Data Operations leadership at Google with "Generative AI × Serverless" implementation skills honed during a 2025 technical deep dive.
I lead the end-to-end lifecycle—from "Translating Business Requirements" to "Scalable AI & Data Platform Design"—to drive DX and practical AI integration.

---

**ビジネスとエンジニアリングの「ギャップ」を埋め、AI/データ基盤の深い実装力に基づき実行をリードする TPgM。**

Googleでの11年間にわたる大規模データオペレーション統括（Data Ops）の経験と、2025年に、技術を深掘りするための期間（Technical Deep Dive）として集中的に磨き上げた「Generative AI × Serverless」の実装スキルを融合。
「ビジネス要件の翻訳」から「スケーラブルなプラットフォーム設計」までを一気通貫でリードし、実用的なAI導入を推進します。

---

## 🚀 Featured Projects: Technical Deep Dive

A collection of serverless, AI-native applications developed during a 2025 technical deep dive, centering on **"User Sovereignty (Data Ownership)"** and **"Serverless FinOps"** architectures.

### 1. ⚾ [Diamond Lens] - MLB Analytics AI Platform

**LLM Harness, Multi-Agent Orchestration, dbt Semantic Layer & MCP Server** | Loom Demo: TBA | [GitHub Repository](https://github.com/takeshimx/diamond-lens)

- **Insight:** A production AI platform for MLB analytics — engineered around a single LLM Gateway, judge-based evaluation, and a dbt Semantic Layer that replaces fragile text-to-SQL with governed text-to-metric.
- **English Summary:** A natural-language MLB analytics platform built with a **harness-first AI design**. The chat path was collapsed from a `Supervisor + 4 LangGraph sub-agents` topology into a single `ChatOrchestrator` (raw `google-genai` SDK + `tool_use` loop), reducing **4 LLM calls → 1** per query; cross-domain strategy reports retain a 5-node `StrategyAgent` (LangGraph). Replaces fragile text-to-SQL with **dbt Semantic Layer (MetricFlow)** running as an internal Cloud Run service via **OIDC service-to-service auth** — every metric defined once in YAML, with **canary rollout via `USE_SEMANTIC_LAYER` flag**. All Gemini calls flow through an **LLM Gateway** with a versioned **Prompt Registry**, a **CI/CD Evaluation Gate** against a golden dataset (blocks deploy on accuracy < 80%), **5 LLM-as-a-Judge services** (parse / synthesizer / reflection / routing / drift-alert), and a **HITL feedback loop** that funnels 👎 ratings back into the golden set. Also exposes an **MCP Server** for Claude Desktop, and an **ML Model Registry + CI/CD Drift Gate** (KS / PSI + cosine-distance semantic drift via BigQuery ML embeddings).
- **日本語ダイジェスト:** MLB分析を題材に、**「LLM harness 層」を最初から設計した本番AIプラットフォーム**。チャット経路は旧 `Supervisor + 4 LangGraph sub-agent` 構成を畳み、`ChatOrchestrator`（素の `google-genai` SDK + `tool_use` ループ）に統合 — **LLM呼び出し回数を 4回 → 1回** に削減。横断的な戦略レポートには 5ノードの `StrategyAgent`（LangGraph）を残存。脆い text-to-SQL は **dbt Semantic Layer (MetricFlow)** に置き換え、Cloud Run 内部サービスとして **OIDC サービス間認証** 経由でアクセス — メトリクス定義は YAML に一元化し、**`USE_SEMANTIC_LAYER` フラグでカナリアロールアウト**。全Gemini呼び出しは **LLM Gateway** を経由し、バージョン管理付き **Prompt Registry**、ゴールデンデータセットに対する **CI/CD評価ゲート（精度80%未満でデプロイ阻止）**、**5種のLLM-as-a-Judge**（parse / synthesizer / reflection / routing / drift-alert）、**HITLフィードバックループ**（👎評価 → ゴールデンセット拡張）を備える。さらに **MCPサーバー** 経由で Claude Desktop から直接アクセス可能、**Model Registry + CI/CD Drift Gate**（KS / PSI + BigQuery ML によるコサイン距離 semantic drift 検知）も搭載。

### 2. 💰 [Sovereign Finance] - Personal Finance AI Platform

**LLM Harness, Hybrid RAG, Eval Regression Gate & Production Reliability** | Loom Demo: TBA | [GitHub Repository](https://github.com/takeshimx/personal-finance-architecture)

- **Insight:** A cost-optimized FinOps platform built with the same harness-first AI discipline as Diamond Lens — golden-set eval gate, prompt-injection-hardened PDF ingestion, and production reliability primitives (idempotency, atomic transactions, optimistic locking) on a BigQuery + Firestore serverless stack.
- **English Summary:** A unified personal-finance platform sharing **the same harness-first AI discipline as Diamond Lens**. Category prediction is a **4-stage Hybrid RAG cascade** (approved rules → keyword → BigQuery `VECTOR_SEARCH` over `gemini-embedding-001` embeddings → Gemini), gated in CI by a **golden-set PR regression check** that blocks merge on ≥ 2pt accuracy drop. Every Gemini and Vertex AI embedding call routes through a **single LLM Gateway** (`call_gemini` / `call_embedding`) logging token counts, USD cost, latency, and feature tag to BigQuery `llm_invocations` — powering an in-app **Settings → Usage** dashboard, with prompts pinned via a **versioned Prompt Registry** (`ACTIVE_VERSIONS`). A **Self-Healing Pipeline (Agentic RAG)** auto-generates merchant rules from user corrections and routes them through a **Slack HITL** approval flow; Brokerage trade-confirmation PDF ingestion runs Gemini 2.5 Flash with **explicit prompt-injection defense** (`<system_instructions>` tags + untrusted-data declaration) and **hard-stop meta validation** (FX range, qty×price drift) that fails loud before any BQ write. Production reliability is layered on: **BigQuery multi-statement transactions** for all-or-nothing cross-table writes, **optimistic locking** (`version INT64` + HTTP 409), and a pure-ASGI **Idempotency-Key middleware** (Firestore-backed, 24h TTL, byte-for-byte replay) auto-protecting every authenticated mutating endpoint against duplicate INSERTs on flaky-network retries. Runs at **< $1/mo** on a BigQuery + Firestore + Cloud Run serverless stack.
- **日本語ダイジェスト:** Diamond Lens と同じ **「harness 層を最初に作る」思想** で設計した低コスト FinOps プラットフォーム。カテゴリ予測は **4段カスケードの Hybrid RAG**（承認済みルール → キーワード → BigQuery `VECTOR_SEARCH`（`gemini-embedding-001` 埋め込み） → Gemini）で、**ゴールデンセット + PR リグレッションゲート** をCIに組み込み、精度が ≥ 2pt 低下したらマージをブロック。全Gemini呼び出し・Vertex AI 埋め込み呼び出しは **単一の LLM Gateway**（`call_gemini` / `call_embedding`）を経由し、トークン数・USDコスト・レイテンシ・feature タグを BigQuery `llm_invocations` に DML 書き込み — アプリ内の **Settings → Usage** ダッシュボードで可視化、プロンプトは **バージョン管理付きレジストリ**（`ACTIVE_VERSIONS` 明示固定）で運用。**Self-Healing Pipeline（Agentic RAG）** によりユーザー修正からカテゴリルールを自動生成し **Slack HITL** 承認フローへ。証券会社の取引報告書 PDF 取り込みは Gemini 2.5 Flash + **明示的なプロンプトインジェクション防御**（`<system_instructions>` タグ + 添付 PDF を untrusted data として宣言）と **ハードストップなメタ検証**（FX 範囲、qty×price 誤差）で BQ 書き込み前に fail-loud で停止。本番信頼性のため、**BigQuery マルチステートメントトランザクション** によるオールオアナッシング書き込み、**楽観ロック**（`version INT64` + HTTP 409）、**Idempotency-Key ミドルウェア**（pure-ASGI / Firestore-backed / 24h TTL / バイト単位リプレイ）で、ネットワーク不安定時の認証付き POST/PUT/PATCH を重複 INSERT から保護。BigQuery + Firestore + Cloud Run のサーバレス構成で **月額 < $1** で運用。

### 3. 🌴 [Travel Agent] - Autonomous Unstructured Data Workflow

**Automated Booking & Multi-Source Trust Scoring** | Loom Demo: TBA | [GitHub Repository](https://github.com/takeshimx/travel-planner-docs)

- **Insight:** Automates journey planning by extracting bookings from emails and evaluating hotel quality using AI trust scoring.
- **English Summary:** An autonomous travel agent specializing in unstructured data processing. It extracts booking details via the **Gmail API** and analyzes hotel quality using a custom **Multi-Source Trust Scoring** system powered by **Firecrawl** and **Gemini 2.5 Flash**. Features a **Cross-Project BigQuery Integration** to synchronize expense data across disparate financial datasets.
- **日本語ダイジェスト:** 旅程管理、メールからの予約自動抽出、AIによるホテルの信頼性評価を一気通貫で自動化する自律型トラベルコンパニオン。**Gmail API** による予約情報の自動パースに加え、**Firecrawl** と **Gemini 2.5 Flash** を活用した **Multi-Source Trust Scoring** を実装し、サクラ投稿等のノイズを排除。さらに、Finance App 側の **BigQuery データをプロジェクト横断で取得**し、予算と実績を統合管理します。

---

## 🛠 Technology Stack

**Cloud & Infrastructure:**
![Google Cloud](https://img.shields.io/badge/GoogleCloud-%234285F4.svg?style=for-the-badge&logo=google-cloud&logoColor=white) ![Cloud Run](https://img.shields.io/badge/Cloud%20Run-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white) ![Firestore](https://img.shields.io/badge/Firestore-FFCA28?style=for-the-badge&logo=firebase&logoColor=black) ![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white) ![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white) ![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white) ![Firebase](https://img.shields.io/badge/firebase-%23039BE5.svg?style=for-the-badge&logo=firebase)

**AI Agent & Backend:**
![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54) ![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi) ![LangChain](https://img.shields.io/badge/LangChain-1C3C3C?style=for-the-badge&logo=langchain&logoColor=white) ![LangGraph](https://img.shields.io/badge/LangGraph-1C3C3C?style=for-the-badge&logo=langchain&logoColor=white) ![Gemini](https://img.shields.io/badge/Google%20Gemini-8E75B2?style=for-the-badge&logo=google%20gemini&logoColor=white) ![BigQuery](https://img.shields.io/badge/BigQuery-669DF6?style=for-the-badge&logo=googlebigquery&logoColor=white) ![Vertex AI](https://img.shields.io/badge/Vertex%20AI-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white) ![dbt](https://img.shields.io/badge/dbt-FF694B?style=for-the-badge&logo=dbt&logoColor=white)

**Frontend & UX:**
![Next JS](https://img.shields.io/badge/Next-black?style=for-the-badge&logo=next.js&logoColor=white) ![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB) ![TypeScript](https://img.shields.io/badge/typescript-%23007ACC.svg?style=for-the-badge&logo=typescript&logoColor=white) ![TailwindCSS](https://img.shields.io/badge/tailwindcss-%2338B2AC.svg?style=for-the-badge&logo=tailwind-css&logoColor=white) ![Vite](https://img.shields.io/badge/vite-%23646CFF.svg?style=for-the-badge&logo=vite&logoColor=white)

**Integrations:**
![Slack](https://img.shields.io/badge/Slack-4A154B?style=for-the-badge&logo=slack&logoColor=white) ![MCP](https://img.shields.io/badge/MCP-000000?style=for-the-badge&logoColor=white) ![Pydantic](https://img.shields.io/badge/Pydantic-E92063?style=for-the-badge&logo=pydantic&logoColor=white)

---

## 💼 Professional Background

- **Google (2013 - 2025):** Program Manager (Data Ops & Search Quality)
  - **Large-Scale Deployment:** Led the successful launch of Indoor Maps for **1,200+ major venues** (airports, transit hubs) across JAPAC, managing end-to-end data collection logistics and ensuring strict quality standards.
  - **Strategic Partnership Ops:** Directed Partnership Operations for Google Maps projects, managing **40+ external partners** across Japan and Australia to drive large-scale data acquisition deliverables.
  - **Data Operations Leadership:** Led high-volume data collection teams of **200+ operators**, ensuring strict quality standards for Search and Geo products across the JAPAC region.
  - **AI/ML Model Improvement:** Managed manual moderation operations processing **2M+ user-generated content items monthly**, directly contributing to ground truth data management and model accuracy.
- **Focus:** Bridging the gap between Business Requirements and Technical Implementation (Data Ops to AI Solutions).

[LinkedIn](https://www.linkedin.com/in/takeshi-matsubayashi-007b0698/)
