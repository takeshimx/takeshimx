# Hi there, I'm Takeshi 👋
### Ex-Google PgM (12y) turned Full-Stack AI Engineer

ビジネス課題を「スケーラブルなアーキテクチャ」と「実装」で解決する Forward Deployed Engineer / Solutions Architect。
Google Japanでの12年間のプログラムマネジメント経験と、Gemini/BigQueryを駆使したフルスタック開発スキルを武器に、企業のDXとAI実装を推進します。

---

## 🚀 Featured Projects (Sabbatical 2025)
2025年5月~12月(900時間+)で開発した、Serverless & AIネイティブなアプリケーション群です。
**"User Sovereignty (データ主権)"** と **"Serverless FinOps"** を設計思想の核としています。

### 1. ⚾ [Diamond Lens] - MLB Analytics AI Agent
**Natural Language Interface (NLI) to SQL Engine for Business Intelligence**
[Loom Demo Video Link Here] | [Repository Link]

* **Problem:** 専門知識なしではSQLを書けず、膨大なライブデータからの即時インサイト抽出が困難。
* **Solution:** 自然言語による意図解釈から「動的SQL」を生成し、BigQueryから直接集計結果を抽出するNLI-to-SQLエンジン。
* **Technical Highlights:**
    * **Gemini 2.5 Flash** による複雑なクエリ意図の正規化（選手名の名寄せ、統計メトリクスの抽出）。
    * **Parameterized Queries** による動的SQL生成と、インジェクション耐性を確保した堅牢なデータパイプライン。
    * **Statcast連携:** Exit Velocityなどの高度な統計予測モデル（Linear Regression）を統合。
* **Tech:** Gemini, BigQuery, FastAPI, React, Cloud Run, Terraform

### 2. 💰 [Sovereign Finance] - Personal AI CFO
**Serverless FinOps Architecture & Hybrid RAG Categorization**
[Loom Demo Video Link Here] | [Repository Link]

* **Problem:** 既存アプリの柔軟性不足と、データプライバシー（User Sovereignty）の欠如。
* **Solution:** BigQueryを中心とした「究極的にランニングコスト（月額数円）を抑えた」データ主権重視のAI CFO。
* **Technical Highlights:**
    * **Hybrid RAG:** Vertex AI Embeddings + Firestore Vector Search によるセマンティック検索と、BigQueryの履歴データを組み合わせた高精度な自動収支分類。
    * **Feedback Loop:** ユーザーによる修正結果を履歴にフィードバックし、Embedding精度の継続的改善を実現。
    * **FinOps:** コンピューティングコストをゼロに抑えるサーバーレス設計。
* **Tech:** Next.js, Vertex AI (Vector Search), Firestore, BigQuery, Gmail API

### 3. 🌴 [Travel Agent] - Autonomous Unstructured Data Workflow
**Automated Booking Management & Multi-Source Trust Scoring**
[Loom Demo Video Link Here] | [Repository Link]

* **Problem:** 予約メールやWebレビューなどの「非構造化データ」の処理コストと、情報の不確実性。
* **Solution:** Gmailからの予約情報自動抽出と、情報の信憑性を検証する「フェイクレビュー検知」ロジックを備えたAIトラベルエージェント。
* **Technical Highlights:**
    * **Autonomous Agent:** LangChain等の重いフレームワークに依存せず、Firecrawl + Gemini を直接統合した軽量かつ高性能な自律型エージェント。
    * **Multi-Source Logic:** Google MapsとBooking.com等の相反する評価データを統計的に分析し、独自の信頼スコア（Trust Score）を算出。
* **Tech:** Agentic AI Workflow, Firecrawl, Gemini, FastAPI, BigQuery

---

## 🛠 Technology Stack
**Cloud & Infrastructure:**
![Google Cloud](https://img.shields.io/badge/GoogleCloud-%234285F4.svg?style=for-the-badge&logo=google-cloud&logoColor=white) ![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white) ![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)

**Backend & AI:**
![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54) ![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi) ![Gemini](https://img.shields.io/badge/Google%20Gemini-8E75B2?style=for-the-badge&logo=google%20gemini&logoColor=white) ![BigQuery](https://img.shields.io/badge/BigQuery-669DF6?style=for-the-badge&logo=googlebigquery&logoColor=white)

**Frontend:**
![Next JS](https://img.shields.io/badge/Next-black?style=for-the-badge&logo=next.js&logoColor=white) ![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB) ![TailwindCSS](https://img.shields.io/badge/tailwindcss-%2338B2AC.svg?style=for-the-badge&logo=tailwind-css&logoColor=white)

---

## 💼 Professional Background
* **Google Japan (201x - 2025):** Program Manager (12 years)
    * Led cross-functional projects... (ここに簡単な実績を英語で1-2行)
* **Focus:** Bridging the gap between Business Requirements and Technical Implementation.

[Contact Me / LinkedIn Link]