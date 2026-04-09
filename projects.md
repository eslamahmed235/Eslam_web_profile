# BOOM AI

**Subtitle:** Intelligent Campaign Proposal & Contract Automation Platform

**Date:** Mar 2026

**Client:** BOOM — Influencer Marketing Agency

**Problem:** BOOM's proposal and contract workflow required 12+ manual steps: interpreting Fireflies.ai meeting recordings, searching influencer databases, drafting proposals, managing revision cycles, and manually preparing contracts — a process heavily dependent on human interpretation across scattered data sources, with no version control or audit trail.

**Solution:** Designed and delivered a production-grade multi-agent AI platform with two specialized agents orchestrated via LangGraph. The Proposal Generation Agent ingests Fireflies.ai meeting summaries, extracts campaign objectives and constraints, performs hybrid influencer retrieval (semantic search + metadata filtering + ranking), and generates structured proposals from templates. The Contract Generation Agent transforms approved proposals into legally structured contracts with clause assembly, then synchronizes with PandaDoc for e-signature workflows. A dedicated Workflow Orchestration Layer manages state transitions, version tracking, and mandatory human validation checkpoints across both phases.

**Tech:** LangGraph, LangChain, OpenAI GPT-4, Airtable, Fireflies.ai API, PandaDoc API, Pinecone, FastAPI, Docker, Python

**Features:**
- Two-phase agentic workflow: Proposal Generation → Contract Generation with mandatory human validation gates
- Hybrid influencer retrieval engine combining semantic search, metadata filtering, and weighted ranking algorithms
- Fireflies.ai meeting summary ingestion with automatic campaign requirement extraction and NLP understanding
- Template-based document generation with AI-assisted iterative revision and full version control tracking
- PandaDoc integration for automated contract rendering, client notification, and electronic signature workflows
- Workflow orchestration layer with state management, audit logging, and approval enforcement

**Results:**
- 70% — Faster proposal turnaround
- 5x — Client throughput increase
- 6 steps — Down from 12+ manual steps

---

# ONDA

**Subtitle:** Multi-Agent AI Therapist Chatbot with Culturally-Sensitive Personalization

**Date:** Jan 2026

**Client:** Mental Health Startup

**Problem:** Delivering safe, empathetic AI-driven mental health conversations requires real-time emotion detection, multi-turn therapeutic context across sessions, strict dual-gate safety protocols with clinical-grade crisis handling, cultural and spiritual sensitivity (especially for Arabic-speaking users), and deep personalization — far beyond the capabilities of a standard chatbot or single-model system. Existing mental health tools lack long-term memory, culturally-aware tone adaptation, and structured therapeutic progression tracking.

**Solution:** Architected a production-grade multi-agent LLM system orchestrated via LangGraph with five specialized agents coordinated by a Supervisor controller. The CDM (Core Dialogue Model) Agent generates empathetic, culturally-sensitive therapeutic responses personalized via a Case & Style Report (CSR) built from user profiles. A Summary Agent extracts structured insights from conversations with 70% similarity deduplication. A Profile Agent maintains a 6-layer user profile (preferences, therapy progress, skills, sensitivities, conversation memory, long-term accumulation) with intelligent merging. A Dual-Gate Safety system combines an OpenAI Moderation API pre-check with a Guard Agent for crisis detection and clinical escalation. Memory architecture uses PostgreSQL for reliable long-term storage and Redis for sub-second hot-path reads, with a two-phase execution model: Critical Path (<2s user response via CDM → Guard) and Background Path (<800ms profile enrichment via Summary → Profile agents running in parallel).

**Tech:** LangGraph, LangChain, OpenAI GPT-4, PostgreSQL, Redis, FastAPI, Python, Pydantic, WebSocket, Docker, Nginx, GitHub Actions, AWS EC2, Sentry

**Features:**
- 5-agent orchestration via Supervisor controller: CDM Agent, Summary Agent, Profile Agent, Guard Agent, and dynamic routing with time-budgeted critical (<2s) and background (<800ms) execution paths
- Case & Style Report (CSR) personalization engine adapting tone (calm, warm, encouraging, reflective, structured, cheerful), dialect (Egyptian Arabic), pacing, content style, and spiritual reference preferences per user
- Dual-Gate Safety: Gate 1 (OpenAI Moderation API pre-check for self-harm, violence, harassment) → Gate 2 (Guard Agent with crisis detection, clinical fallback responses, and escalation to emergency resources)
- 6-layer persistent user profile: preferences, therapy profile (stages: rapport → explore → reflect → plan → wrap), learned skills, sensitivities/triggers, session memory, and long-term conversation accumulation — with 70% similarity deduplication on merge
- Dual-layer memory architecture: PostgreSQL for reliable long-term storage (profiles, summaries, safety flags) + Redis cache for hot-path reads (recent context, profile summary, therapeutic stage) with automatic fallback
- Structured therapeutic progression tracking across sessions with theme extraction, goal monitoring, strength identification, trigger awareness, and stage-based conversation steering

**Results:**
- <2s — Critical path response
- 5 agents — Orchestrated per turn
- Dual-gate — Clinical-grade safety

**Production & Deployment:**
Deployed as a containerized microservice architecture on AWS EC2 using Docker Compose — FastAPI application server, PostgreSQL database, and Redis cache each running as isolated containers behind an Nginx reverse proxy with SSL termination. CI/CD pipeline via GitHub Actions automates testing, image builds, and zero-downtime deployments on every push to main. WebSocket connections are load-balanced for concurrent real-time chat sessions. Sentry integrated for error tracking and performance monitoring across all agent execution paths. Health check endpoints expose per-agent latency, Redis hit rates, and database connection pool status. Secrets managed via AWS Secrets Manager; all patient data encrypted at rest (AES-256) and in transit (TLS 1.3). The platform has been running in production with stable uptime, handling concurrent therapeutic sessions with consistent sub-2-second response times.

**Business Value:**
- Enables scalable, personalized mental health support for Arabic-speaking populations— a massively underserved market with 400M+ speakers
- Reduces therapist workload by handling initial intake, ongoing check-ins, and structured therapeutic progression autonomously
- Cultural and spiritual sensitivity (dialect, religious references, tone) drives higher user trust, engagement, and retention vs. generic English-only solutions
- Long-term memory and profile continuity across sessions creates compounding therapeutic value — each conversation makes the next one more effective
- Dual-gate safety architecture meets clinical-grade requirements for crisis handling, enabling partnerships with healthcare institutions and regulatory compliance
- Two-phase execution model (immediate response + background enrichment) maintains sub-2-second UX while continuously improving personalization quality

---

# AI Law Advisor

**Subtitle:** Saudi Labor Law AI Contract Review, Compliance System & Legal Chatbot

**Date:** Sep 2025

**Client:** Legal Tech / HR Compliance (Saudi Arabia)

**Problem:** Saudi labor law compliance requires clause-by-clause contract analysis against 240+ legal articles spanning employment contracts, wages, termination, leave, probation, and workplace safety — a process that takes legal professionals 4–6 hours per contract and is error-prone when done manually. HR teams at SMEs lack specialized legal knowledge for accurate compliance assessment. Additionally, employees and employers need 24/7 access to authoritative legal guidance in Arabic with source-backed answers, not generic AI responses that hallucinate legal references.

**Solution:** Engineered a comprehensive two-system AI legal platform: (1) A Contract Review Engine implementing four experimental approaches benchmarked head-to-head across 18 model configurations — fine-tuned GPT-4.1/4.1-mini trained on Saudi labor law corpora, RAG-enhanced models (GPT-3.5-turbo through GPT-5-mini/nano and o4-mini) with FAISS-indexed law articles and metadata-rich retrieval, baseline GPT controls, and Hybrid RAG+Fine-tuned models combining domain specialization with explicit legal grounding. Built a full Arabic NLP pipeline with clause chunking, diacritics normalization, metadata-rich context injection (article numbers + chapter names), and structured JSON-validated outputs. Evaluated rigorously across 6 metrics using the RAGAS framework with a standardized evaluator (gpt-4o-mini, temperature=0) ensuring unbiased cross-model comparison. (2) A Legal Advisor Chatbot with dual-mode interaction (casual chat vs. legal consultation), context-aware multi-turn conversations, and 3-level reference-backed answers (direct answer → source summary with article/chapter → full law text) — all grounded exclusively in Saudi Labor Law with zero hallucination design.

**Tech:** OpenAI GPT-4.1, OpenAI GPT-5, FAISS, RAGAS, LangChain, LangSmith, FastAPI, React, Python, Pydantic, Arabic NLP, Fine-tuning API, Docker, Nginx, GitHub Actions

**Features:**
- Four-family experimental architecture: Fine-tuned GPT only vs. RAG-enhanced GPT vs. Baseline GPT vs. Hybrid (RAG + Fine-tuned) — benchmarked across 18 model configurations including GPT-3.5-turbo, GPT-4.1, GPT-4o, GPT-5-mini, GPT-5-nano, and o4-mini
- Custom fine-tuning pipeline: domain-specific training data generation from Saudi labor law corpora, fine-tuned GPT-4.1 and GPT-4.1-mini models learning legal decision boundaries from curated compliant/violation samples
- FAISS-indexed RAG with metadata-rich retrieval: article numbers, chapter names, and law text injected into prompts via format_with_metadata() for grounded legal citations — eliminating hallucinated article references
- 6-metric RAGAS evaluation suite with standardized evaluator (gpt-4o-mini, temperature=0): Exact Match, BLEU Score, Semantic Similarity, Factual Correctness, Answer Accuracy, and Response Relevancy — ensuring unbiased cross-model comparison
- Arabic NLP pipeline: clause-level contract chunking, diacritics stripping and normalization, legal entity extraction, metadata injection, and structured JSON-validated compliance outputs (صحيحة/مخالفة + article citation)
- Dual-mode Legal Chatbot: context-aware multi-turn conversations with conversation memory, 3-level reference-backed answers (direct answer → article/chapter source summary → full law text), and 240+ indexed Saudi labor law articles covering contracts, wages, termination, leave, probation, and workplace safety

**Results:**
- 18 models — Benchmarked head-to-head
- 6 metrics — RAGAS evaluation suite
- 240+ — Law articles indexed

**Production & Deployment:**
Shipped as a Docker Compose stack with three services: FastAPI backend exposing RESTful contract review and chatbot endpoints, a FAISS vector store service with pre-built index volumes for instant cold starts, and an Nginx gateway handling SSL, rate limiting, and request routing. React frontend communicates via API with streaming responses for the chatbot. CI/CD pipeline via GitHub Actions runs the full RAGAS evaluation suite on every model update — blocking deployment if any metric regresses below baseline thresholds. LangSmith integrated in production for per-request trace logging, retrieval quality monitoring, and latency alerting. The FAISS index and fine-tuned model weights are versioned as artifacts with rollback support. Structured logging ships to a centralized stack for audit compliance. The system has been running in production serving HR teams and legal departments with zero-hallucination contract analysis and 24/7 legal consultation.

**Business Value:**
- Reduces contract review time from 4–6 hours to minutes — enabling legal teams to process 10x more contracts with consistent quality
- Four-family experimental approach (Fine-tuned vs. RAG vs. Baseline vs. Hybrid across 18 configurations) delivers evidence-based model selection, not guesswork — clients get the optimal accuracy/cost tradeoff for their specific use case
- Zero-hallucination legal chatbot with source-backed answers builds trust with HR teams and legal departments who cannot afford fabricated article references in compliance contexts
- Arabic-native NLP pipeline with diacritics normalization and clause chunking solves a critical gap — most legal AI tools fail on Arabic text, leaving the Saudi market underserved
- 24/7 legal advisor accessibility democratizes labor law knowledge for SMEs that cannot afford dedicated legal counsel, reducing compliance violations and associated penalties
- Standardized RAGAS evaluation with 6 metrics provides auditable model performance evidence — essential for regulated industries where AI decision transparency is mandatory

---

# SQL Query & Viz

**Subtitle:** LLM-Powered Natural Language to SQL Analytics Platform

**Date:** Jul 2025

**Client:** Telecom Data Analytics

**Problem:** Telecom companies accumulate massive datasets spanning subscriber usage, billing cycles, payment histories, and device networks. Extracting meaningful insights from this data traditionally requires significant SQL expertise — limiting direct access to actionable information for business users and decision-makers who need real-time analytics without technical bottlenecks.

**Solution:** Built an end-to-end LLM-powered analytics assistant using LangGraph that transforms natural language business questions into accurate, schema-aware SQL queries executed against Azure SQL databases. The pipeline features a multi-stage graph architecture: an NL-to-SQL generation node using GPT-4 with full schema injection and prompt engineering, a database execution layer returning structured DataFrames, an intelligent chart generation node producing up to three distinct Chart.js configurations rendered via QuickChart API, and an insight generation node delivering business-style natural language summaries contextualizing the results.

**Tech:** LangGraph, LangChain, OpenAI GPT-4, Azure SQL, QuickChart API, Streamlit, Pandas, Python

**Features:**
- Natural language to SQL translation with full database schema awareness and query validation
- Multi-stage LangGraph pipeline: SQL generation → execution → chart rendering → insight generation
- Automatic generation of up to three distinct Chart.js visualizations per query with type diversity
- Business-style natural language insight summaries contextualizing query results for decision-makers
- Document upload and dynamic database update capabilities for evolving telecom datasets
- Streamlit-based interactive UI with real-time query processing and result visualization

**Results:**
- 3 charts — Auto-generated per query
- Instant — NL to SQL to insights
- Zero SQL — Required from users

---

# NextGen RAG Chatbot

**Subtitle:** Enterprise-Grade Conversational AI with Advanced RAG Architecture

**Date:** Jun 2025

**Client:** Enterprise Client

**Problem:** Traditional chatbot systems struggle with maintaining conversation context across sessions, retrieving accurate information from large document corpora, and delivering production-grade reliability with proper observability and error handling. Enterprises need a next-generation conversational AI that combines persistent memory, advanced retrieval-augmented generation, and robust deployment architecture.

**Solution:** Engineered a production-grade RAG chatbot using LangGraph with a sophisticated multi-node orchestration architecture. The system features a persistence and memory layer for multi-session conversation continuity, a retrieval-augmented generation flow with semantic search over vectorized document stores, an optimized prompting strategy for context-aware response generation, and async execution models for high-throughput inference. Integrated comprehensive observability, security compliance layers, and error handling with graceful fallbacks.

**Tech:** LangGraph, LangChain, FAISS, OpenAI GPT-4, FastAPI, Redis, Docker, Python

**Features:**
- LangGraph-based multi-node orchestration with state management and conditional routing
- Persistent memory system for multi-session conversation continuity and context retention
- Advanced RAG flow with semantic retrieval, context assembly, and relevance scoring
- Optimized prompting strategy with dynamic context window management and response quality control
- Full observability stack with monitoring, logging, and performance tracking dashboards
- Async execution model for high-throughput, low-latency inference at production scale

**Results:**
- Production — Enterprise-grade deployment
- Multi-session — Persistent memory
- Async — High-throughput inference

---

# WzGATE

**Subtitle:** Intelligent Real Estate Conversational AI with RAG

**Date:** Mar 2025

**Client:** Real Estate Company (Egypt)

**Problem:** Real estate agents were overwhelmed by repetitive property queries across WhatsApp and web channels, with no way to intelligently distinguish between property-specific questions (requiring database lookup) and general market or neighborhood questions (requiring web knowledge). Response times averaged 4+ hours, causing lost leads.

**Solution:** Architected a dual-mode conversational AI system using LangGraph with a classifier LLM that dynamically routes queries between two specialized RAG subgraphs: a Property RAG for database-backed listing queries with FAISS semantic search over structured property data, and a General Knowledge RAG using Tavily real-time web search for market trends, neighborhood info, and regulatory questions. Deployed via Twilio WhatsApp API and Streamlit web interface for omni-channel delivery.

**Tech:** LangGraph, LangChain, FAISS, Tavily API, Twilio WhatsApp, Streamlit, OpenAI GPT-4, SQLAlchemy, Python

**Features:**
- Dual-mode RAG architecture with intelligent classifier LLM for dynamic query routing between subgraphs
- Property-specific RAG: FAISS semantic search over structured listing data with metadata filtering (price, area, type)
- General knowledge RAG: Tavily real-time web search for market trends, neighborhood information, and regulations
- Omni-channel deployment: WhatsApp via Twilio API + Streamlit web interface with unified conversation state
- Contextual follow-up handling with multi-turn conversation management and session persistence
- Automated lead qualification and intent extraction for CRM integration

**Results:**
- Dual-mode — Intelligent query routing
- Omni-channel — WhatsApp + Web
- <30s — Avg. response time

---

# Mandatory List Checker

**Subtitle:** AI-Powered Bilingual Product Matching & Compliance Verification

**Date:** Mar 2025

**Client:** Government / Regulatory Compliance

**Problem:** Organizations must verify whether product names — often messy, bilingual (Arabic/English), or inconsistently formatted — appear on official mandatory lists and determine Local Content Certificate and baseline requirements. Manual Excel lookups are slow, error-prone, and don't scale when processing large product catalogs, leading to compliance gaps and certification delays.

**Solution:** Built an interactive AI-driven compliance verification tool with a three-tier matching pipeline: exact dictionary lookup, lexical similarity scoring via RapidFuzz, and semantic vector search using FAISS with OpenAI embeddings. The system handles Arabic/English normalization, automatic language detection, and QA-threshold-based flagging for manual review. Integrated an LLM-powered conversational agent with function-calling tools enabling users to query, explain, filter, toggle flags, and correct matching results through natural dialogue — all persisted in Streamlit session state with exportable results.

**Tech:** OpenAI GPT-4, FAISS, RapidFuzz, Streamlit, Pandas, NumPy, Python

**Features:**
- Three-tier matching pipeline: exact lookup → lexical (RapidFuzz) → semantic (FAISS + embeddings) with best-score selection
- Bilingual Arabic/English text normalization: diacritics stripping, punctuation cleaning, case normalization
- LLM-powered chat agent with function-calling tools for querying, explaining, filtering, and editing match results
- QA-threshold flagging system for automatic identification of matches requiring manual verification
- CSV/XLSX upload support with one-click demo data for rapid testing and onboarding
- Exportable results with match method, confidence scores, baseline status, and certification requirements

**Results:**
- 3-tier — Matching pipeline
- Bilingual — Arabic + English
- High precision — Minimal false matches

---

# KFSC Edu Assist

**Subtitle:** Arabic Academic RAG Assistant with Custom NLP

**Date:** Feb 2025

**Client:** KFSC Academic Institution (Saudi Arabia)

**Problem:** Students and faculty needed instant, accurate answers from Arabic academic content spanning multiple courses and departments. Existing search tools failed catastrophically on Arabic text due to dialect variations, diacritics inconsistency, morphological complexity, and lack of semantic understanding for academic terminology.

**Solution:** Developed a production RAG system with a custom Arabic NLP normalization pipeline handling diacritics stripping, morphological analysis, stopword removal, and dialect normalization. Implemented semantic chunking optimized for academic document structure (lectures, textbooks, exam materials). Built a course-aware retrieval system with metadata-driven filtering and dynamic context assembly. Rigorously evaluated using RAGAS, AL-BLEU, and ROUGE metrics with real-time LangSmith monitoring.

**Tech:** LangGraph, LangChain, FAISS, OpenAI GPT-4, RAGAS, LangSmith, Streamlit, CAMeL Tools, Python

**Features:**
- Custom Arabic NLP pipeline: diacritics normalization, morphological analysis, stopword removal, dialect handling
- Semantic chunking engine optimized for academic document structures (lectures, textbooks, exams, syllabi)
- Course-aware RAG retrieval with metadata-driven filtering by department, course code, and content type
- Rigorous evaluation suite: RAGAS + AL-BLEU + ROUGE with per-query performance breakdowns
- Real-time LangSmith observability: latency monitoring, retrieval quality tracking, and chain debugging
- Dynamic context window management with relevance scoring and redundancy elimination

**Results:**
- >95% — Context recall
- >86% — Faithfulness score
- Multi-course — Academic coverage

---

# Hajj-GPT

**Subtitle:** First-of-Kind Islamic Reference RAG Chatbot

**Date:** Jan 2025

**Client:** Islamic Guidance Platform

**Problem:** Over 2 million Hajj pilgrims annually need authentic, real-time religious guidance on rituals, prayers, and regulations. Existing resources are static books and websites with no interactivity, personalization, or multilingual support — leaving non-Arabic speakers particularly underserved during one of the most important spiritual journeys of their lives.

**Solution:** Pioneered the first Islamic reference RAG chatbot powered by retrieval-augmented generation over authenticated religious texts (Quran, Hadith, and scholarly Fiqh references). Built a retrieval pipeline with FAISS-indexed religious corpus, source verification layer ensuring only authenticated texts are cited, and dynamic content personalization based on pilgrim progress and ritual stage. Integrated multilingual FAQ generation supporting Arabic, English, and Urdu.

**Tech:** LangChain, OpenAI GPT-4, FAISS, Streamlit, Python, Sentence Transformers, MongoDB

**Features:**
- First-of-kind Arabic-native Islamic RAG chatbot with authenticated religious text retrieval and citation
- Source verification layer ensuring only scholarly-authenticated Quran, Hadith, and Fiqh references are cited
- Ritual tracking engine: personalized guidance adapting to the pilgrim's current stage (Ihram, Tawaf, Sa'i, etc.)
- Multilingual support with dynamic FAQ generation in Arabic, English, and Urdu
- Context-aware conversational flow handling follow-up questions with theological accuracy
- Offline-capable reference cache for areas with limited connectivity during Hajj

**Results:**
- First-of-kind — Islamic AI reference
- 3 languages — Arabic, English, Urdu
- 2M+ — Potential pilgrim users

---

# Arabic Dialect Text-to-Image

**Subtitle:** Pioneering Arabic Dialect Image Generation Research

**Date:** Oct–Dec 2024

**Client:** Academic Research / NLP Innovation

**Problem:** State-of-the-art text-to-image generation systems (Stable Diffusion, DALL-E, Midjourney) are exclusively trained on English text, leaving Arabic — a language spoken by 400M+ people across diverse dialects — completely unsupported. No prior research existed on generating images from Arabic dialect prompts, creating a significant gap in multilingual AI capabilities.

**Solution:** Pioneered one of the first systematic studies on Arabic dialect text-to-image generation, designing and evaluating multiple transformation pipelines: dialect-to-MSA (Modern Standard Arabic) and dialect-to-English translation approaches across 4 Arabic dialects (Egyptian, Gulf, Levantine, Maghreb). Benchmarked 5+ translation models and 3 diffusion architectures (Stable Diffusion 1.5, SD-XL, SD 2.1). Generated and rigorously evaluated 36K+ images using FID, Inception Score, and CLIP Score metrics.

**Tech:** Stable Diffusion XL, Gemini Pro, CLIP, PyTorch, HuggingFace Diffusers, MarianMT, Python, CUDA, Weights & Biases

**Features:**
- Comprehensive multi-dialect pipeline: Egyptian, Gulf, Levantine, and Maghreb Arabic dialect support
- Dual transformation approach: dialect→MSA and dialect→English with comparative quality analysis
- 5+ translation model benchmark: Gemini, MarianMT, Helsinki-NLP, and custom fine-tuned models
- 3 diffusion architecture evaluation: Stable Diffusion 1.5, SD-XL, and SD 2.1 with quality comparison
- 36K+ image generation and evaluation using FID, Inception Score, and CLIP Score metrics
- Reproducible research pipeline with Weights & Biases experiment tracking and artifact versioning

**Results:**
- 36K+ — Images generated & evaluated
- 4 dialects — Arabic dialect coverage
- First study — Arabic dialect image gen

---

# Cairo 3A AI Poultry

**Subtitle:** Predictive Analytics & Conversational AI Platform for Poultry Farm Intelligence

**Date:** Jul 2024

**Client:** Cairo 3A — via Datalentech

**Problem:** Large-scale poultry operations managing multiple farms and houses generate massive daily telemetry — bird weights, mortality counts, feed consumption, medication logs, and environmental readings — but lack both predictive tools and accessible interfaces to act on that data. Farm managers rely on intuition and manual dashboard navigation, leading to reactive decision-making: feed waste goes undetected until cycle end, mortality spikes are caught too late, selling-period timing is guesswork, and non-technical staff are locked out of critical analytics. No existing system combines farm operations data with weather intelligence, forward-looking ML predictions, and a natural-language interface that lets any user query the platform conversationally.

**Solution:** Designed and delivered a full-stack predictive analytics platform with a Gemini-powered conversational AI layer. The prediction engine forecasts four critical poultry metrics — feed consumption, bird weight, mortality rate, and culls — by ingesting live farm data from Oracle DB, enriching with weather APIs, and training sequence-based ML models on 80+ engineered features across seven categories (rolling statistics, lag features, weather correlations, statistical moments, trigonometric time encodings, and farm metadata). The platform delivers four interactive modules: General Charts with ideal-zone analysis, Scenario Analysis ranked by Broiler Index, Model Performance dashboards (MAPE, MSE, MAE, R²), and RBAC User Management. On top of this, a Gemini-based chatbot service provides natural-language access to every platform capability — using intent classification to dynamically route queries to the correct API endpoint, parameter extraction with slot-filling for missing values (org codes, house IDs, dates, cycle numbers), multi-turn conversation context tracking for follow-up questions, and a modular plug-in handler architecture that makes adding new endpoints trivial. The chatbot formats responses as plain-language summaries, tables, or chart-ready JSON, with a full audit log of every query for traceability and future model retraining.

**Tech:** Python, Gemini API, scikit-learn, XGBoost, Pandas, NumPy, Oracle DB, FastAPI, React, Redis, Docker, Nginx, GitHub Actions, Weather API, JWT

**Features:**
- Four-metric forecasting engine: predicts feed consumption, bird weight, mortality rate, and culls per house per cycle — using sequence-based ML models trained on historical cycle data enriched with weather correlations across 80+ engineered features (rolling stats, lag features, weather correlations, statistical moments, trigonometric encodings, farm metadata)
- Gemini-powered conversational AI chatbot: intent classification dynamically routes natural-language queries to the correct API endpoint (predictions, scenarios, cycles, weather, training, user management, logs) — non-technical farm managers get business intelligence by simply asking questions instead of navigating dashboards
- Smart parameter extraction with slot-filling: the chatbot extracts org codes, house IDs, dates, cycle numbers, and metric types from user messages — and auto-asks targeted follow-ups when parameters are missing ('Which farm?' / 'Which cycle?') before executing the query
- Multi-turn conversation context manager: tracks session history across N turns so users can ask follow-ups naturally ('Show last cycle for farm 3… Now compare it to farm 5') — context-aware slot-filling carries forward previously mentioned entities
- Scenario Analysis simulator: generates what-if selling-period timelines by varying start/end ages and sales events — ranks scenarios by Broiler Index (composite of weight, mortality, feed efficiency) with interactive bubble charts and per-scenario feature importance analysis (SHAP-style contribution charts)
- Model Performance observatory: tracks MAPE, MSE, MAE, and R² per metric per house — with error distribution scatter plots (overestimation vs. underestimation), error trend timelines, metric comparison tabs, and per-house predicted-vs-actual comparison tables
- Modular plug-in endpoint architecture: each API domain (poultry cycles, predictions, training, weather, user management, model registry, harvest planning, logs) is a self-contained handler module with its own parameter validation, request construction, response formatting, and error management — adding a new endpoint means writing one handler class and registering it
- Multi-source data pipeline: Oracle DB ingestion for daily farm telemetry + weather API enrichment (temperature, humidity, wind, precipitation, pressure, cloud cover, soil temperature, daylight duration) + weekly aggregation — with chatbot audit logging storing every query/response pair with timestamp, user, intent, confidence score, and endpoint for analytics and Gemini retraining

**Results:**
- 80+ — Engineered features
- 4 metrics — Forecasted per cycle
- <5% — Avg prediction error
- Conversational AI — Natural-language access

**Production & Deployment:** Deployed as a containerized stack via Docker Compose: FastAPI backend serving prediction, scenario, and chatbot endpoints, React frontend with embedded conversational interface, Oracle DB connection pool for live farm data ingestion, Redis for conversation session state and context caching, and Nginx reverse proxy with SSL termination. The chatbot service runs as a dedicated FastAPI microservice with JWT authentication, rate limiting, and role-based endpoint access — ensuring farm managers only query data within their authorized scope. CI/CD via GitHub Actions runs model validation tests and chatbot intent regression tests on every push, blocking deployment on MAPE regression or intent classification accuracy drops. Feature engineering pipeline runs on scheduled cron, pulling daily telemetry and weather data, computing rolling/lag/correlation features, and retraining models per farm. Chatbot audit logs feed into an analytics pipeline for monitoring query patterns, intent confidence distributions, and identifying gaps for Gemini prompt refinement. Model artifacts and chatbot prompt versions are both tracked with rollback support.

**Business Value:**
- Democratizes farm intelligence for non-technical users — farm managers, supervisors, and field staff get predictions, scenarios, and analytics by chatting in natural language instead of navigating complex dashboards or waiting for analysts
- Conversational AI with multi-turn context eliminates the learning curve entirely — new users are productive from day one, asking follow-up questions naturally without understanding the underlying data model or API structure
- Scenario Analysis with Broiler Index ranking lets managers optimize selling-period timing for maximum output — simulating dozens of start/end date combinations in seconds via chat or dashboard
- 80+ engineered features including weather correlations surface hidden drivers of mortality and growth — with chatbot explanations that translate complex model outputs into actionable plain-language insights
- Plug-in handler architecture future-proofs the platform — new data sources, endpoints, or analytics capabilities can be added without touching core chatbot logic, reducing expansion cost by orders of magnitude
- Full audit trail of every chatbot interaction enables compliance reporting, usage analytics, and continuous Gemini prompt refinement — the system gets smarter with every query

---

# Mealit

**Subtitle:** AI-Powered Meal Planning & Kitchen Assistant with Recommendation Engine

**Date:** Jan 2023

**Client:** Suez Canal University — Graduation Project

**Problem:** The rise of busy lifestyles and fast-food consumption has led to a decline in home-cooked meals, contributing to health issues and significant household food waste. Users lack convenient tools to plan meals based on available ingredients, discover recipes matching dietary preferences, and manage grocery shopping efficiently — resulting in poor nutrition, wasted food, and unnecessary spending.

**Solution:** Developed a full-stack AI-powered meal planning application featuring an intelligent recommendation system that suggests recipes based on available household ingredients using collaborative and content-based filtering algorithms. The system includes image recognition for ingredient identification, a role-based chatbot for personalized culinary guidance, a shopping cart with price comparison across grocery stores, and low-ingredient alerts. Built with React frontend, Node.js backend, Flutter mobile app, and Python ML services with Redux state management.

**Tech:** Python, React, Node.js, Flutter, Redux, MongoDB, TensorFlow, scikit-learn

**Features:**
- AI recommendation engine using collaborative and content-based filtering for personalized recipe suggestions
- Image recognition system for automatic ingredient identification from photos
- Role-based chatbot providing personalized cooking guidance, dietary advice, and meal planning assistance
- Shopping cart with cross-store price comparison and automatic low-ingredient alerts
- Cross-platform delivery: React web application + Flutter mobile app with unified backend
- Dietary restriction filtering, favorite recipe management, and step-by-step cooking instructions with timers

**Results:**
- Full-stack — Web + Mobile app
- AI-powered — Recipe recommendations
- Role-based — Personalized chatbot

---

# IoT Healthcare System

**Subtitle:** Real-Time Biomedical Monitoring & Emergency Alert Platform

**Date:** Dec 2022

**Client:** Benha University — Master's Graduation Project

**Problem:** Emergency healthcare situations require continuous, real-time monitoring of critical patient vital signs with immediate data transmission to medical professionals. Traditional monitoring systems are wired, immobile, and lack remote accessibility — preventing timely intervention for patients in emergency scenarios, rural areas, or home care settings where rapid response is critical.

**Solution:** Developed an innovative IoT healthcare monitoring system featuring a custom biomedical instrument equipped with eight sensors capable of monitoring ten different health parameters simultaneously. The system transmits captured health data via NodeMCU Wi-Fi connectivity to a microcontroller for processing, then delivers real-time readings to healthcare professionals through both an Android mobile application and a Thinger.io web dashboard. Designed for emergency scenarios with wireless data transfer, remote monitoring capabilities, and instant alert mechanisms.

**Tech:** Arduino, NodeMCU, Android SDK, Thinger.io, Wi-Fi, IoT Sensors, Java, C++

**Features:**
- Custom biomedical instrument with 8 sensors monitoring 10 health parameters simultaneously
- Real-time wireless data transmission via NodeMCU Wi-Fi to cloud-based processing
- Android mobile application for healthcare professionals to monitor patient vitals remotely
- Thinger.io web dashboard for real-time visualization and remote patient condition monitoring
- Emergency alert system with configurable thresholds for critical vital sign changes
- Microcontroller-based data aggregation and processing for efficient sensor data management

**Results:**
- 10 params — Health parameters monitored
- Real-time — Wireless data transmission
- Dual platform — Android + Web dashboard

---

# AI Recruitment System

**Subtitle:** Intelligent Candidate Recommendation & Chatbot Platform for Recruitment

**Date:** Mar 2023

**Client:** Curve AI — Talents Arena

**Problem:** Traditional recruitment processes rely on manual resume screening and keyword-based matching, resulting in high candidate-job misalignment, slow hiring cycles, and poor candidate experience. Recruiters spend excessive time sourcing and evaluating candidates without intelligent tools to surface the best matches, while candidates lack visibility into relevant opportunities matching their actual skills and experience.

**Solution:** Built an end-to-end AI-powered recruitment recommendation system leveraging NLP and neural networks for intelligent candidate-job matching. The platform features dual recommendation engines: one for surfacing job openings tailored to company requirements, and another for recommending candidates based on qualifications, skills, and experience. Integrated LLaMA 7B for candidate summarization and LLaMA 13B for advanced context-aware conversational flows via a chatbot that enables recruiters to discuss and evaluate candidates. Engineered data processing pipelines with Pandas and JSON schemas for dynamic handling of resumes and job postings, with A/B testing and ranking evaluation achieving 0.81 Precision@5.

**Tech:** LLaMA 7B/13B, LangChain, Pandas, Docker, Semantic Ranking, A/B Testing, Python, NLP

**Features:**
- Dual recommendation engine: job recommendations for recruiters and candidate recommendations for specific openings
- LLaMA 7B-powered candidate summarization highlighting key qualifications and attributes for quick evaluation
- LLaMA 13B chatbot integration enabling recruiters to discuss, compare, and evaluate candidates conversationally
- Candidate dashboard displaying relevant job openings and career opportunity exploration
- Data processing pipelines with JSON schema processing and feature extraction for dynamic resume/job handling
- A/B testing and offline benchmarking with ranking evaluation metrics for continuous model improvement

**Results:**
- 0.81 — Precision@5 accuracy
- 35% — Less hiring misalignment
- 60% — Faster hiring cycle

---

# Social Media Analysis

**Subtitle:** Large-Scale Tweet Analytics with Predictive Modeling

**Date:** Sep 2021

**Client:** University of Ottawa — Microsoft-Sponsored Capstone

**Problem:** Understanding public sentiment and demographic patterns across social media at scale requires processing millions of unstructured text data points with high accuracy. Traditional analytics tools cannot handle the volume, linguistic diversity, and real-time demands of large-scale social media analysis needed for actionable business and research insights.

**Solution:** Processed 10M+ tweets for large-scale social media analysis, building predictive models for user demographics, 5-level sentiment classification, and multi-label emotion detection. Developed comprehensive NLP pipelines including text preprocessing, feature engineering, and ensemble classification models. Deployed interactive dashboards via Power BI and Plotly for real-time actionable analytics, enabling stakeholders to explore trends, sentiment shifts, and demographic insights dynamically.

**Tech:** Python, scikit-learn, Power BI, Plotly, NLP, Pandas, NumPy

**Features:**
- Large-scale processing pipeline handling 10M+ tweets with efficient text preprocessing and feature extraction
- 5-level sentiment classification with ensemble ML models for nuanced opinion mining
- Multi-label emotion detection identifying complex emotional patterns across tweet content
- User demographic prediction models for audience segmentation and targeting insights
- Interactive Power BI and Plotly dashboards for real-time trend visualization and drill-down analytics
- Microsoft-sponsored research with production-grade analytics pipeline deployment

**Results:**
- 10M+ — Tweets processed
- 5-level — Sentiment classification
- Real-time — Interactive dashboards

---

# IoT Botnet Detection

**Subtitle:** Adaptive ML-Based Network Intrusion Detection System

**Date:** Apr 2021

**Client:** Academic Research — Network Security

**Problem:** The rapid proliferation of IoT devices has created an expanding attack surface for botnet-based network intrusions. Traditional rule-based intrusion detection systems cannot adapt to evolving attack patterns, suffer from high false positive rates, and lack the ability to process network traffic streams in real-time — leaving IoT networks vulnerable to DDoS, data exfiltration, and device hijacking attacks.

**Solution:** Developed a machine learning system for detecting network intrusions and IoT botnet attacks using Decision Tree, Random Forest, and SVM algorithms trained on structured network traffic datasets. Implemented a real-time detection pipeline using Apache Kafka for stream processing, enabling continuous monitoring of network traffic patterns. Designed the system to adapt over time with iterative evaluation using accuracy, F1-score, and detailed classification reports, enhancing detection performance as attack patterns evolve.

**Tech:** scikit-learn, Apache Kafka, Random Forest, SVM, Decision Tree, Pandas, Python

**Features:**
- Multi-algorithm ensemble: Decision Tree, Random Forest, and SVM for robust intrusion classification
- Real-time stream processing pipeline using Apache Kafka for continuous network traffic monitoring
- Adaptive detection model designed to evolve with changing IoT botnet attack patterns
- Comprehensive evaluation with accuracy, F1-score, precision, recall, and confusion matrix analysis
- Structured CSV dataset processing with feature engineering for network traffic characterization
- Scalable architecture supporting high-throughput packet analysis for production IoT networks

**Results:**
- Real-time — Kafka stream detection
- 3 models — Ensemble classification
- Adaptive — Evolving threat detection

---

# SecureNet IDS

**Subtitle:** Deep Learning Intrusion Detection & Prevention System

**Date:** Mar 2021

**Client:** Academic Research — Cybersecurity

**Problem:** Complex network intrusions including DoS, Probe, User-to-Root (U2R), and Remote-to-Local (R2L) attacks require sophisticated detection mechanisms that go beyond traditional signature-based approaches. Standard ML classifiers struggle with the high dimensionality and class imbalance inherent in network intrusion datasets, particularly for rare but critical attack types like U2R and R2L.

**Solution:** Built a network security solution combining Autoencoders for unsupervised anomaly detection with ANN classifiers and traditional ML algorithms to detect complex intrusion types. The autoencoder learns normal traffic patterns and flags anomalies, while the ANN classifier categorizes detected intrusions into specific attack types. Trained and validated on industry-standard datasets (UNSW-NB15 and NSL-KDD) from the University of New Brunswick, with performance evaluation using accuracy, precision, and confusion matrix analysis to optimize detection reliability across all attack categories.

**Tech:** TensorFlow, Keras, scikit-learn, Pandas, NumPy, Python, Deep Learning

**Features:**
- Hybrid architecture: Autoencoder for anomaly detection + ANN classifier for attack categorization
- Multi-class intrusion detection: DoS, Probe, User-to-Root (U2R), and Remote-to-Local (R2L) attacks
- Trained on industry-standard benchmarks: UNSW-NB15 and NSL-KDD datasets
- Autoencoder-based unsupervised feature learning for capturing normal traffic distribution patterns
- Comprehensive evaluation with accuracy, precision, recall, and confusion matrix across all attack types
- Traditional ML baseline comparison (SVM, Random Forest) for rigorous performance benchmarking

**Results:**
- Hybrid — Autoencoder + ANN
- 2 benchmarks — UNSW-NB15 & NSL-KDD
- Multi-class — 4 attack categories
