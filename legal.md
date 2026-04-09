# 1. Experimental Setup: Model Families and What Each Tests

## A. Fine-tuned GPT only

**What it is.**

Models trained on domain-specific pairs derived from Saudi labor law (see `finetuning_data/` and the generator in `data_generation.py`). At inference, they read the contract clause and decide **“النقطة القانونية صحيحة”** (legally sound) or **“النقطة القانونية مخالفة للمادة …”** (violation + article) using a fixed legal prompt (no retrieval).

**Concrete models in this project (seen in outputs):**

- `ft-gpt-4.1-mini` → results in `Results_ft-gpt4.1-mini.csv`
- `ft-gpt-4.1` → results in `Results_ft-gpt4.1.csv`

**Why we run it.**

To measure the benefit of **domain specialization** alone (no external context). This isolates how much the fine-tune learned the Saudi labor-law decision boundary from curated positive/negative samples.

**How it runs in code.**

- In `evaluate_contracts.py`, generation calls `ft_model_answer(...)`, which wraps the system prompt **`ANALYZE_CHUNK_PROMPT`** from `contract_review/prompt.py` and passes each **clause** (`chunk`) to the model with `temperature=0`.
- Gold labels come from `final_eval.jsonl` (`ex["messages"][1]["content"]`).
- Evaluation uses Ragas (e.g., `ExactMatch`, `BleuScore`, `SemanticSimilarity`, `FactualCorrectness`, `AnswerAccuracy`) with a **single evaluator LLM** (`gpt-4o-mini`) via `LangchainLLMWrapper`.
- Per-sample and aggregate scores are written to `evaluation_outputs/Results_ft-*.csv` and combined into the aggregate workbook(s).

---

## B. RAG-enhanced GPT

**What it is.**

General models are paired with a **FAISS** retriever built from the Saudi labor law corpus (`faiss_index/`), so each clause is judged with **in-prompt legal citations** (article number + chapter) injected via `rag_analyzer.py`.

**Concrete models in this project (seen in outputs):**

- `rag-gpt-3.5-turbo` → `Results_rag-gpt-3.5-turbo.csv`
- `rag-gpt-4.1-mini` → `Results_rag-gpt-4.1-mini.csv`
- `rag-gpt-4.1` → `Results_rag-gpt-4.1.csv`
- `rag-gpt-4o` → `Results_rag-gpt-4o.csv`
- `rag-gpt-5-mini` → `Results_rag-gpt-5-mini.csv`
- `rag-gpt-5-nano` → `Results_rag-gpt-5-nano.csv`
- `rag-o4-mini` → `Results_rag-o4-mini.csv`

**Why we run it.**

To test whether **grounding** with authoritative law text improves **factuality** and **faithfulness**, especially on clauses that look plausible but are subtly non-compliant.

**How it runs in code.**

- `rag_analyzer.py` loads the FAISS index, sets `retriever = vectorstore.as_retriever(k=settings.RAG_TOP_K)`, and builds a **metadata-rich prompt** with article numbers and chapter names (`format_with_metadata(...)`).
- In `evaluate_contracts.py`, RAG variants call `rag_model_answer(...)`, which retrieves, formats, and queries the base model.
- **Context-sensitive metrics** such as `ResponseRelevancy` are enabled (and can be extended with `LLMContextRecall` / `LLMContextPrecision*` when `retrieved_contexts` are logged).
- Results are saved under `evaluation_outputs/Results_rag-*.csv`.

---

## C. Baseline GPT

**What it is.**

Zero/few-shot prompting of general models without fine-tuning and **without** retrieval. This is the **control group**.

**Concrete models in this project (seen in outputs):**

- `gpt-3.5-turbo` → `Results_gpt-3.5-turbo.csv`
- `gpt-4.1-mini` → `Results_gpt-4.1-mini.csv`
- `gpt-4.1` → `Results_gpt-4.1.csv`
- `gpt-4o` → `Results_gpt-4o.csv`
- `gpt-5-mini` → `Results_gpt-5-mini.csv`
- `gpt-5-nano` → `Results_gpt-5-nano.csv`
- `o4-mini` → `Results_o4-mini.csv`

**Why we run it.**

To establish **baseline difficulty** of the task and quantify **delta gains** from fine-tuning and/or retrieval.

**How it runs in code.**

- Same `ft_model_answer(...)` pathway as the fine-tuned family, but the **model** is a base model (no RAG).
- Prompts use `ANALYZE_CHUNK_PROMPT`; `temperature=0`.
- Evaluation and logging identical to other families.

---

## D. Hybrid (RAG + Fine-tuned GPT)

**What it is.**

Fine-tuned models receive **retrieved legal context** before answering. This is the **synergy** setup: domain specialization + explicit grounding.

**Concrete models in this project (seen in outputs):**

- `rag+ft-gpt-4.1` → `Results_rag+ft-gpt4.1.csv`
- `rag+ft-gpt-4.1-mini` → `Results_rag+ft-gpt4.1-mini.csv`

**Why we run it.**

To test whether a specialized classifier still **benefits from explicit law snippets** (e.g., fewer hallucinations, better article coding, clearer justifications).

**How it runs in code.**

- Retrieval and metadata injection from `rag_analyzer.py`, but the **answering LLM** is the **fine-tuned** variant.
- Same Ragas evaluation; per-sample + aggregates stored to `evaluation_outputs/`.

---

# 2. Evaluation Data & Pipeline

## **Dataset**

- `final_eval.jsonl` is the gold-labeled evaluation set. Each item has:
    - `messages[0].content` → the **contract clause** (Arabic)
    - `messages[1].content` → the **gold verdict** (“صحيحة” vs “مخالفة …”)

**Generation & Scoring loop.**

- `evaluate_contracts.py` sets `sample_data_mode = "new"` and loads all lines from `final_eval.jsonl`.
- For each experiment entry in `experiments: Dict[str, object]`, the script:
    1. **Generates** model answers (`ft_model_answer` for non-RAG; `rag_model_answer` for RAG).
        - Shared system prompt: `ANALYZE_CHUNK_PROMPT` (Arabic legal reviewer).
        - `temperature=0` for determinism.
        - A 180-second **timeout wrapper** skips outliers without crashing the run.
    2. **Builds** a Ragas `EvaluationDataset` from triples (question, answer, reference).
    3. **Evaluates** with:
        - String metrics: `ExactMatch`, `BleuScore`, `SemanticSimilarity`.
        - LLM metrics: `FactualCorrectness`, `AnswerAccuracy`.
        - Context metric used here: `ResponseRelevancy` (others can be toggled when logging retrieved contexts).
        - **Evaluator LLM** is fixed across all runs: `gpt-4o-mini` via `LangchainLLMWrapper`.
    4. **Writes outputs**:
        - Per-model, per-sample CSV → `evaluation_outputs/Results_<model>.csv`
        - Aggregated table across models → `evaluation_outputs/aggregate_scores_base_models.csv`
        - (You also keep combined Excel summaries: `all_experiments_aggregated.xlsx`, `all_experiments_aggregated_all.xlsx`, plus a narrative `experiments results.docx/xlsx`.)

**What the CSVs represent (from your folder listing):**

- **Fine-tuned only:** `Results_ft-gpt4.1-mini.csv`, `Results_ft-gpt4.1.csv`
- **Baseline (no RAG, no FT):** `Results_gpt-3.5-turbo.csv`, `Results_gpt-4.1-mini.csv`, `Results_gpt-4.1.csv`, `Results_gpt-4o.csv`, `Results_gpt-5-mini.csv`, `Results_gpt-5-nano.csv`, `Results_o4-mini.csv`
- **RAG-enhanced:** `Results_rag-gpt-3.5-turbo.csv`, `Results_rag-gpt-4.1-mini.csv`, `Results_rag-gpt-4.1.csv`, `Results_rag-gpt-4o.csv`, `Results_rag-gpt-5-mini.csv`, `Results_rag-gpt-5-nano.csv`, `Results_rag-o4-mini.csv`
- **Hybrid RAG+FT:** `Results_rag+ft-gpt4.1.csv`, `Results_rag+ft-gpt4.1-mini.csv`

## **Notes helpful for the paper.**

- **Prompting is identical** across families (except the RAG variants which prepend retrieved law text with article metadata). This keeps the comparison fair.
- **Evaluator kept constant** (same model + embeddings) to reduce evaluator drift.
- **Deterministic decoding** (`temperature=0`) ensures repeatable runs.
- **Timeout guard** avoids run starvation from API stalls; skipped items are logged (your script prints `[ALERT] Skipped item … due to timeout`), so N per model may differ slightly; report N_used per run.
- Because your labels are **short categorical strings** in Arabic, `ExactMatch` directly captures label-level correctness; `BLEU`/`SemanticSimilarity` supplement when answers include **justifications** (particularly in RAG/Hybrid)

## **Evaluator**

A consistent evaluator model was used to **standardize scoring** across all experiment types. Without this, comparing fine-tuned GPT models against RAG-enhanced and baseline models would introduce bias, since different evaluators could interpret responses differently.

- **Evaluator LLM:**
    - `gpt-4o-mini` was selected as the sole evaluator.
    - Wrapped inside Ragas (`LangchainLLMWrapper` in `evaluate_contracts.py`).
    - Fixed temperature = 0, with retries and timeout safeguards (to prevent dropped evaluations due to API instability).
- **Rationale:**
    - Using one evaluator ensures that all models — whether fine-tuned, baseline, or RAG-augmented — are judged under the **same standard of interpretation**.
    - This eliminates variability caused by evaluator creativity or style drift.
- **Implementation Detail:**
    
    ```python
    EVALUATOR_LLM = LangchainLLMWrapper(
        ChatOpenAI(model="gpt-4o-mini", temperature=0, timeout=3000, max_retries=4)
    )
    ```
    
    - The evaluator is paired with **OpenAI embeddings** (`OpenAIEmbeddings`) for metrics like *Semantic Similarity*.
    - All results are stored in `evaluation_outputs/` as both per-sample CSVs and aggregated summary tables.

## **Evaluation Metrics**

The evaluation pipeline consistently applied **six metrics** across all experiments — whether fine-tuned, baseline, RAG-enhanced, or hybrid. This ensures comparability across models and experiment types.

## **1. Exact Match**

- **Definition:** A strict binary metric that checks if the model’s output matches the gold reference **character by character**.
- **Purpose in contract review:** Very useful when the expected answers are short categorical outputs such as:
    - `"النقطة القانونية صحيحة"`
    - `"النقطة القانونية مخالفة للمادة 93"`
- **Limitation:** Overly rigid. It penalizes outputs that are semantically correct but worded differently (e.g., `"النقطة صحيحة"` vs. `"النقطة القانونية صحيحة"`).
- **Interpretation:**
    - **Higher values →** Strong alignment with the gold labels, especially for fine-tuned models trained on these exact phrases.
    - **Lower values →** Does not always mean the model is “wrong” — it may simply be using different but correct wording.

---

## **2. BLEU Score**

- **Definition:** Measures **n-gram overlap** between model output and gold reference.
- **Purpose:** Captures *partial correctness*, rewarding outputs that share overlapping word sequences even if the entire output isn’t identical.
- **Application to legal text:** Useful in compliance detection, since Saudi labor law phrases are often rigid, and clause-based rules benefit from precise wording.
- **Limitation:** Fails to capture meaning if synonyms or paraphrasing are used.
- **Interpretation:**
    - **Higher values →** Model outputs closely mirror the surface form of legal phrasing in the gold standard.
    - **Lower values →** Model either paraphrased too much, or diverged from canonical legal language.

---

## **3. Semantic Similarity**

- **Definition:** An embedding-based metric that measures whether two sentences express the same meaning, regardless of wording. Uses OpenAI embeddings.
- **Purpose:** Checks if outputs are semantically aligned with gold references even when phrased differently.
- **Relevance to Arabic contracts:** Essential, since equivalent compliance judgments can be phrased differently.
    - Example: `"صحيح"` vs. `"متوافق مع النظام"` → different wording, same meaning.
- **Interpretation:**
    - **Higher values →** Strong semantic alignment (ideal for fine-tuned models and RAG explanations).
    - **Lower values →** Model diverged semantically, producing misleading or irrelevant content.

---

## **4. Factual Correctness**

- **Definition:** LLM-based evaluation where the evaluator (here, *gpt-4o-mini*) checks whether **legal claims are factually valid**.
- **Purpose:** Ensures that references to Saudi labor law are not hallucinated and correspond to actual articles.
- **Application:** Especially important in RAG and Hybrid setups, where retrieval should ground responses.
- **Interpretation:**
    - **Higher values →** The model not only predicts the correct compliance verdict but also ties it to the correct legal reasoning.
    - **Lower values →** Indicates hallucinations, fabricated article references, or misinterpretation of retrieved content.

---

## **5. Answer Accuracy**

- **Definition:** Direct metric comparing the model’s **final verdict** (Correct vs. Violation) with the gold compliance label.
- **Purpose:** Core classification metric. Penalizes cases where the model flips the legal judgment.
- **Example:**
    - Gold: `"مخالفة للمادة 77"`
    - Model: `"النقطة القانونية صحيحة"` → marked as inaccurate.
- **Interpretation:**
    - **Higher values →** Reliable compliance classification — the model consistently agrees with human-labeled ground truth.
    - **Lower values →** Dangerous in real-world use, since it means the model often fails to detect violations correctly.

---

## **6. Response Relevancy**

- **Definition:** Measures whether the output is **focused and relevant** to the input clause.
- **Purpose:** Detects verbose, generic, or off-topic outputs.
- **Application in RAG:** Critical — ensures the model doesn’t drift into unrelated retrieved articles or hallucinate outside the contract context.
- **Interpretation:**
    - **Higher values →** Concise, on-topic answers grounded in the contract clause and retrieved from Saudi law.
    - **Lower values →** Indicates “noise” in answers — e.g., generic legal filler, irrelevant details, or misuse of retrieved context.

---


# 1. Introduction | المقدمة

### What is the Legal Advisor Chatbot?

The Legal Advisor Chatbot is your intelligent assistant for understanding Saudi Labor Law. Think of it as having a knowledgeable legal consultant available 24/7 to answer your questions about employment regulations, worker rights, and employer obligations under Saudi law.

**المستشار القانوني الذكي** هو مساعدك الشخصي لفهم نظام العمل السعودي. تخيله كمستشار قانوني متوفر على مدار الساعة للإجابة على أسئلتك حول قوانين التوظيف وحقوق العمال والتزامات أصحاب العمل.

### What Makes It Special?

Unlike general AI chatbots that might give you information from anywhere on the internet, this chatbot:

✅ **Exclusively uses Saudi Labor Law** - Every answer is based on official law articles

✅ **Shows its sources** - You can verify every claim by reading the actual law text

✅ **Remembers your conversation** - Understands context from previous questions

✅ **Speaks Arabic naturally** - Designed for Arabic speakers with professional legal terminology

✅ **Never makes things up** - If it doesn’t find the answer in the law, it tells you honestly

### Scope of Knowledge

The chatbot is trained on:
- **Saudi Labor Law (نظام العمل السعودي)** - Complete articles with chapter organization
- **Over 240+ legal articles** covering employment relationships, rights, and obligations
- **Topics include:**
- Employment contracts (عقود العمل)
- Working hours and overtime (ساعات العمل والعمل الإضافي)
- Wages and compensation (الأجور والتعويضات)
- Leave and vacations (الإجازات)
- Termination and notice periods (إنهاء العقد وفترات الإشعار)
- Probation periods (فترات التجربة)
- Employee rights and employer obligations (حقوق العمال والتزامات أصحاب العمل)
- Workplace safety (السلامة في مكان العمل)
- Disciplinary actions (الإجراءات التأديبية)

---

# 2. Core Capabilities | القدرات الأساسية

## 2.1 Dual Mode Interaction | التفاعل المزدوج

The chatbot is smart enough to distinguish between two types of conversations:

### 🤝 Casual Chat Mode (الدردشة العامة)

When you greet or thank the chatbot:
- **You say:** “مرحبا” or “صباح الخير” or “شكراً”
- **Chatbot responds:** Friendly greeting + offer to help with legal questions
- **No references shown** - Just a warm, professional interaction

### ⚖️ Legal Consultation Mode (الاستشارة القانونية)

When you ask a legal question:
- **You ask:** “ما هي مدة فترة التجربة؟”
- **Chatbot responds:** Detailed answer + article numbers + chapter names + full law text
- **Full transparency** - You see exactly where the answer comes from

## 2.2 Context-Aware Conversations | المحادثات الذكية

The chatbot doesn’t treat each question in isolation. It maintains conversation memory:

**Example of Context Awareness:**

```
👤 المستخدم: "ما هي أقصى مدة لفترة التجربة؟"
🤖 الروبوت: [يشرح أن الحد الأقصى هو 180 يومًا، ويستشهد بالمادة كذا]

👤 المستخدم: "هل يمكن تمديدها؟"
🤖 الروبوت: [يفهم أن الضمير "ها" يعود إلى فترة التجربة من السؤال السابق]
        [يجيب عن قواعد التمديد دون الحاجة لإعادة ذكر الموضوع]

👤 المستخدم: "ماذا يحدث إذا تجاوز صاحب العمل هذه المدة؟"
🤖 الروبوت: [لا يزال يعلم أننا نتحدث عن فترة التجربة]
        [يشرح عواقب تجاوز مدة 180 يومًا]

```

This makes conversations feel natural, like talking to a real legal advisor who remembers what you discussed.

## 2.3 Reference-Backed Answers | إجابات مدعومة بالمراجع

Every legal answer includes three levels of information:

### Level 1: Direct Answer (الإجابة المباشرة)

Clear, concise response to your question in everyday language.

### Level 2: Sources Summary (ملخص المصادر)

- **Article Number** (رقم المادة): e.g., “Article 53”
- **Chapter Name** (اسم الفصل): e.g., “Working Hours and Leave”
- **Key Excerpt** (مقتطف رئيسي): First 400 characters of the relevant article

### Level 3: Full Context (السياق الكامل)

- Complete text of all retrieved law articles
- Allows you to read the full legal text yourself
- Verify the chatbot’s interpretation
- Use for official reference or documentation

---

# 3. How to Use the Chatbot | كيفية استخدام المستشار القانوني

## 3.1 Starting a Conversation | بدء محادثة

### Types of Questions You Can Ask:

**✅ Direct Questions (أسئلة مباشرة)**
- “ما هي مدة الإجازة السنوية؟” (What is the annual leave duration?)
- “هل يحق لصاحب العمل فصل الموظف بدون إشعار؟” (Can employer terminate without notice?)

**✅ Scenario-Based Questions (أسئلة سيناريو)**
- “لدي موظف يريد الاستقالة خلال فترة التجربة، ما الإجراء؟”

(I have an employee wanting to resign during probation, what’s the procedure?)

**✅ Comparison Questions (أسئلة مقارنة)**
- “ما الفرق بين عقد محدد المدة وعقد غير محدد المدة؟”

(What’s the difference between fixed-term and indefinite contracts?)

**✅ Conditional Questions (أسئلة شرطية)**
- “إذا تجاوز العامل ساعات العمل المحددة، ما حقوقه؟”

(If worker exceeds specified working hours, what are their rights?)

**✅ Follow-Up Questions (أسئلة متابعة)**
- After initial answer: “هل هناك استثناءات؟” (Are there exceptions?)
- “ماذا عن…” (What about…)
- “وفي حالة…” (And in case of…)

### Example Questions in Arabic:

```
💬 "هل يجوز للعامل أخذ إجازة بدون أجر؟"
💬 "ما هي ضوابط العمل الإضافي؟"
💬 "كم يوم إجازة مرضية للموظف في السنة؟"
💬 "متى يحق للموظف مكافأة نهاية الخدمة؟"
💬 "هل يمكن إنهاء عقد العمل أثناء فترة التجربة؟"
💬 "ما هي حقوق المرأة العاملة في نظام العمل؟"
💬 "كيف يتم احتساب ساعات العمل في رمضان؟"
```

## 3.2 Understanding Responses | فهم الإجابات

When you ask a legal question, the response is organized into three sections:

### 📝 Section 1: Main Answer

This appears first and contains:
- Direct response to your question
- Written in clear, accessible Arabic
- Professional legal terminology explained simply
- Typically 2-5 paragraphs

**What to look for:**
- Clear statement of the law
- Practical implications
- Any important conditions or exceptions

### 🔍 Section 2: View References (عرض المراجع)

Click to expand this section to see:
- **Article numbers** that support the answer
- **Chapter names** where these articles appear
- **Brief excerpts** (first 400 characters) of each article

**Why this matters:**
- Verify the chatbot’s answer
- Note specific article numbers for official purposes
- Understand which chapter of the law applies

**Example display:**

```
📚 Article 53 | Chapter: Working Hours
Excerpt: "ساعات العمل للعاملين في المنشآت التجارية والفنادق والحراسة..."

📚 Article 107 | Chapter: Leave and Vacations
Excerpt: "للعامل الحق في إجازة سنوية لا تقل عن واحد وعشرين يوماً..."
```

### 📚 Section 3: Retrieved Context (السياق المسترجع)

Click to expand for complete details:
- **Full text** of every article the chatbot consulted
- **Article number, chapter, and source** for each
- Allows you to read the complete law, not just excerpts

**When to use this:**
- You need the complete legal text for official documentation
- You want to verify interpretation
- You’re preparing legal arguments or reports
- You need to cite the exact law

## 3.3 Following Up | المتابعة في الحوار

The chatbot excels at multi-turn conversations. Here’s how to make the most of it:

### ✅ Natural Follow-Ups

You don’t need to repeat context:

```
❌ Don't say: "في السؤال السابق سألت عن فترة التجربة، والآن أريد أن أعرف..."
✅ Just say: "هل يمكن تمديدها؟" (Can it be extended?)
```

The chatbot remembers what you were discussing.

### ✅ Ask for Clarification

If something isn’t clear:

```
💬 "هل يمكن شرح هذا بشكل أبسط؟"
💬 "ما معنى [مصطلح قانوني]؟"
💬 "هل يمكن إعطاء مثال؟"
```

### ✅ Explore Related Topics

Build on the conversation:

```
💬 First: "ما هي حقوق العامل عند إنهاء العقد؟"
💬 Then: "وماذا عن التزامات صاحب العمل؟"
💬 Then: "هل هناك استثناءات في حالة الفصل التأديبي؟"
```

### ✅ Request Specifics

Narrow down general answers:

```
💬 After general answer: "ماذا عن العقود محددة المدة تحديداً؟"
💬 Or: "هل ينطبق هذا على القطاع الخاص؟"
```

---

# 4. Example Conversations | أمثلة على المحادثات

### Scenario 1: Simple Direct Question

### سيناريو ١: سؤال مباشر بسيط

![image.png](attachment:47afdf59-3581-444a-ad64-5b5d27a80256:image.png)

**What Happens Behind the Scenes:**
1. 🔄 System refines question: “مدة الإجازة السنوية المستحقة للعامل وفق نظام العمل السعودي”
2. 🔍 Searches law database for articles about annual leave
3. 📚 Retrieves Article 109 (and related articles)
4. 💬 Generates answer based exclusively on retrieved articles

**If User Clicks “View References”:**

![image.png](attachment:fda54c9a-15b2-438c-8170-6b92b2682edd:image.png)

![image.png](attachment:55741ebb-b115-41ce-9bff-ecf2df3fd648:image.png)

---

### Scenario 2: Multi-Turn Conversation with Context

### سيناريو ٢: محادثة متعددة الدورات مع سياق

**Turn 1:**

![image.png](attachment:06a24a86-8e82-459d-bacb-21a3cbf6439d:image.png)

**Turn 2 (Follow-up):**

![image.png](attachment:55098916-ba33-45f4-8f50-5ce51ed513e7:image.png)

**Turn 3 (Deeper dive):**

![image.png](attachment:05ebac56-86f0-4a21-86f1-417a48f9a341:image.png)

**Notice how:**
- The chatbot maintains context across three questions
- Answers build progressively without repeating basic information
- References accumulate (Article 53, then Article 54)
- User doesn’t need to restate “probation period” each time

---

### Scenario 3: Greeting vs Legal Query

### سيناريو ٣: التمييز بين التحية والسؤال القانوني

**Example A: Casual Greeting**

**Example B: Thank You**

![image.png](attachment:5b7e0eaa-8457-43d6-b29e-41e9a1f41073:image.png)

**Example C: Legal Question Immediately After**

![image.png](attachment:623ee5f7-4b8e-44fe-adb7-b2fe714dcbf1:image.png)

**The Intelligence:**
The chatbot uses context clues to determine intent:
- Short phrases like greetings → Casual mode
- Questions with “ما/هل/كيف” + legal terms → Legal mode
- Thanks/acknowledgments → Casual mode
- Everything else legal → Legal mode with full references

---

### Scenario 4: When the Chatbot Doesn’t Know

### سيناريو ٤: عندما لا يجد المستشار الإجابة

**What Happens:**
1. System searches for articles about “تأمين صحي” (health insurance)
2. No relevant articles found in Saudi Labor Law database
3. System honestly reports inability to answer

**Chatbot Response:**

![image.png](attachment:27a842e8-ae07-453b-8553-4ef00302c8b8:image.png)

**Why This Is Important:**
- ✅ Honest about limitations
- ✅ Doesn’t invent information
- ✅ Suggests alternative resources
- ✅ Maintains trust and reliability

---

# 5. What Makes Answers Trustworthy | ما يجعل الإجابات موثوقة

### 5.1 Exclusive Use of Official Law

**The Golden Rule:** The chatbot ONLY uses Saudi Labor Law articles in its database.

**What this means:**
- ❌ No internet search results
- ❌ No general knowledge from training data
- ❌ No opinions or interpretations not grounded in law
- ✅ Only official law articles from the JSONL database
- ✅ Every claim traceable to specific article

**How it’s enforced:**
The system’s instruction explicitly states:
> “Answer EXCLUSIVELY based on the retrieved context below. If you don’t find a matching article, say: ‘لم أعثر على مادة مطابقة…’ and do not resort to external sources.”

### 5.2 Transparent Source Citation

**Every legal answer includes:**

1. **Article Numbers (أرقام المواد)**
    - Example: “المادة 53”
    - Allows you to look up the article independently
2. **Chapter Names (أسماء الفصول)**
    - Example: “الفصل: عقد العمل”
    - Gives context within the broader law structure
3. **Article Text (نص المادة)**
    - Full text available on demand
    - Verify the chatbot’s interpretation yourself

**Example of Full Transparency:**

```
Answer: "فترة التجربة لا تتجاوز 180 يوماً"

Source shown:
📚 Article 53 | Chapter: Employment Contract
Full text: "يجوز عند التعاقد تعيين العامل تحت التجربة بشرط أن يكون
ذلك ثابتاً بالكتابة ولا تزيد مدة التجربة على مائة وثمانين يوماً..."
```

### 5.3 Honesty When Uncertain

The chatbot is programmed to:

**✅ Admit when it doesn’t know**
- “لم أعثر على مادة مطابقة في قاعدة المراجع”
- (I didn’t find a matching article in my reference database)

**✅ Stay within scope**
- Won’t answer questions outside Saudi Labor Law
- Won’t speculate beyond what articles state

**✅ Suggest alternatives**
- Recommends consulting specialists for out-of-scope questions
- Points to other relevant regulations if appropriate

**Example:**

```
Question: "ما عقوبة مخالفة نظام العمل؟"

Honest Answer: "قد تتطلب تفاصيل العقوبات والغرامات الرجوع إلى
لوائح تنفيذية وقرارات وزارة الموارد البشرية، والتي قد لا تكون
متضمنة بالكامل في المواد الأساسية لنظام العمل المتاحة لدي."
```

### 5.4 Consistency Guaranteed

**Temperature Setting: 0.0**

The AI model is configured with “temperature zero,” meaning:
- **Same question = Same answer** every time
- No randomness or creativity in legal responses
- Deterministic, repeatable results
- Critical for legal advice where consistency matters

**Why this matters:**
- If you ask the same question tomorrow, you’ll get the same answer
- Multiple users asking the same question get identical responses
- No risk of contradictory advice

### 5.5 Multi-Layer Verification

You can verify answers at multiple levels:

**Layer 1: Read the answer**
- Human-friendly explanation

**Layer 2: Check sources**
- See which articles were cited

**Layer 3: Read full articles**
- Complete legal text available

**Layer 4: Independent verification**
- Article numbers allow you to cross-reference with official government sources

---

*تم تصميم هذا المستشار القانوني بتقنيات الذكاء الاصطناعي المتقدمة لخدمة المجتمع وتسهيل الوصول إلى المعرفة القانونية. استخدمه بحكمة واستشر المختصين للأمور الحساسة.*

*This Legal Advisor is designed with advanced AI technology to serve the community and facilitate access to legal knowledge. Use it wisely and consult professionals for sensitive matters.*

# Overview

The Legal Advisor Chatbot evaluation system is a comprehensive, two-stage evaluation framework designed to assess the quality, accuracy, and faithfulness of Arabic legal responses generated by various AI models. The system evaluates responses to questions about Saudi Labor Law (نظام العمل السعودي) using both automated metrics and LLM-based judging.

### Key Objectives

- **Accuracy Assessment**: Measure how well model responses align with authoritative legal references
- **Semantic Quality**: Evaluate the semantic similarity and relevance of responses
- **Factual Correctness**: Verify that legal claims match the official regulations
- **Faithfulness**: Detect hallucinations and ensure responses stay grounded in source material
- **Comparative Analysis**: Compare RAG (Retrieval-Augmented Generation) models against non-RAG baselines across different question typesEvaluation Architecture

---

## The evaluation pipeline consists of four main components:

### 1. Data Generation Layer

- **Purpose**: Automatically generate question-answer pairs from the labor law corpus
- **Input**: Structured JSONL file containing organized labor law articles
- **Output**: Synthetic Q&A pairs for human review and curation
- **Script**: `Data generation/data_generation for chatbot.py`

### 2. Golden Set Curation Layer

- **Purpose**: Human-verified test dataset with reference answers
- **Format**: CSV file with questions, gold-standard answers, and metadata
- **File**: `evaluate_chatbot/inputs/GenAI-Law ChatBot Human Review Golden set.csv`

### 3. Automated Metrics Layer (RAGAS)

- **Purpose**: Compute objective quality metrics using the RAGAS framework
- **Metrics**: BLEU, Semantic Similarity, Factual Correctness, Answer Accuracy, Response Relevancy
- **Script**: `evaluate_chatbot.py`

### 4. LLM Judge Layer

- **Purpose**: Deep faithfulness analysis using structured LLM evaluation
- **Focus**: Point-by-point legal accuracy, hallucination detection, contradiction identification
- **Script**: `evaluate_chatbot/final generation model results legal advisor/evaluate_llm_metrics.py`

---

# Input Data Generation

## Source Material

The evaluation data originates from the **Saudi Labor Law corpus**, which is stored in a highly structured JSONL format at:

```
source_data/labor_law_final_version_after_organization.jsonl
```

Each record in this corpus contains:
- **Article Number** (رقم الماده): Unique identifier for the legal article
- **Chapter Name** (اسم الفصل): The legal chapter/section the article belongs to
- **Article Text** (الماده): The full legislative text, cleaned and organized

## Generation Process

### Step 1: Article Preprocessing

Before generating questions, the system applies several text cleaning operations:
- **Markdown Removal**: Strips code fences, heading markers, and blockquotes
- **Bullet Point Extraction**: Identifies definition lists and enumerated items
- **Text Trimming**: Limits excessively long passages to maintain prompt quality

### Step 2: Prompt Construction

For each labor law article, the system constructs a specialized prompt that instructs an LLM (default: `gpt-4.1-mini` at temperature 0.2) to generate exactly two questions:

**Direct Question (مباشر/تطبيقي)**
- Application-focused
- Asks about specific conditions, rights, or obligations
- Example: “What are the requirements for a worker to be eligible for end-of-service benefits?”

**Explanatory Question (تفسيري/تفصيلي)**
- Interpretation-focused
- Asks about the reasoning, scope, or implications of a regulation
- Example: “Why does the law require specific documentation for employment contracts?”

### Step 3: Structured Response Format

The generator enforces strict JSON output using OpenAI’s `response_format` parameter. The required schema is:

```
{
  "qas": [
    {
      "type": "direct",
      "question": "<question text in Arabic>",
      "answer": "<answer text in Arabic>"
    },
    {
      "type": "explanatory",
      "question": "<question text in Arabic>",
      "answer": "<answer text in Arabic>"
    }
  ]
}
```

### Step 4: Quality Constraints

The generation prompt enforces several critical requirements:
- **No Article Numbers in Questions**: Questions must not explicitly mention the article number (e.g., “according to Article 47…”)
- **Article Citation in Answers**: Every answer MUST explicitly reference the article number it’s based on
- **Exclusive Source Grounding**: Answers must derive exclusively from the provided article text, with no external references
- **Arabic Language**: All content must be in Modern Standard Arabic

### Step 5: Retry Mechanism

If the LLM produces invalid JSON or fails schema validation, the system:
1. Sends a follow-up strict correction prompt
2. Attempts extraction of JSON from markdown-wrapped responses
3. Retries up to a configured maximum (default: 2 retries)
4. Logs failures for manual review

### Step 6: Output Storage

Successfully generated Q&A pairs are written line-by-line to:

```
Data generation/eval_qa_gold_test_samples.jsonl
```

Each line contains:
- `article_no`: The source article identifier
- `chapter`: The legal chapter name
- `article_text`: The cleaned article content
- `qas`: Array of two question-answer objects

---

## Golden Test Set Structure

### File Location

```
evaluate_chatbot/inputs/GenAI-Law ChatBot Human Review Golden set.csv
```

### Required Columns

### Core Evaluation Fields

1. **Question** (السؤال)
    - The legal query posed to the chatbot
    - Must be clear, unambiguous, and answerable from the labor law
    - Written in Arabic
2. **answer** (الإجابة المرجعية)
    - The gold-standard reference answer
    - Reviewed and verified by legal experts
    - Serves as ground truth for all metric calculations
    - Must include explicit article citations
3. **Type** (نوع السؤال)
    - Classification of question style
    - Valid values: `direct` or `explanatory`
    - Used to segment evaluation results by question type

### Contextual Metadata (Optional)

1. **chapter** (اسم الفصل)
    - Legal chapter for reference context
2. **labor law number** (رقم المادة)
    - Original article number
3. **labor law content** (نص المادة)
    - Full text of the referenced article
4. **Reviewer Decision** (قرار المراجع)
    - Human review approval status
5. **Reviewer Comments** (ملاحظات المراجع)
    - Quality notes from human reviewers

### Data Preprocessing

Before evaluation, the script performs several cleaning steps:

1. **Unnamed Column Removal**: Drops any columns with names like `Unnamed: 8`, `Unnamed: 9` (artifacts from Excel exports)
2. **Empty Row Filtering**: Removes rows where all values are NaN or empty strings
3. **Required Field Validation**: Ensures `Question`, `answer`, and `Type` columns exist and contain valid data
4. **Type Normalization**: Converts the `Type` field to lowercase for consistent filtering
5. **Dictionary Conversion**: Each valid row becomes a dictionary:
    
    ```
    {
      "question": "<cleaned question text>",
      "gold": "<cleaned reference answer>",
      "type": "direct" | "explanatory"
    }
    ```
    

### Subset Selection

The evaluation system supports:
- **Full Dataset Evaluation**: Process all questions together
- **Type-Specific Evaluation**: Separate evaluation runs for `direct` and `explanatory` questions
- **Optional Limiting**: `QUESTION_LIMIT` parameter can restrict evaluation to first N questions for smoke testing

---

# Evaluation Models

The evaluation framework compares multiple model configurations across two architectural paradigms:

## Non-RAG Baseline Models

These models answer questions using only their pre-trained knowledge without accessing external documents.

### Architecture

- **Input**: Question text + system prompt
- **System Prompt**: Arabic instructions emphasizing Saudi Labor Law expertise, conciseness, and compliance with Islamic legal principles
- **Output**: Direct answer from model parameters

### Available Models

1. **GPT-4.1** (`gpt-4.1`)
    - Latest GPT-4 series model
    - High capability baseline
    - Temperature: 0.0 (deterministic)
2. **GPT-4o** (`gpt-4o`)
    - Optimized variant of GPT-4
    - Balance of quality and speed
3. **GPT-4.1-mini** (`gpt-4.1-mini`)
    - Smaller, faster variant
    - Cost-effective baseline
4. **GPT-5-mini** (`gpt-5-mini`)
    - Next-generation small model
    - Enhanced reasoning capabilities
5. **GPT-5.1** (`gpt-5.1`)
    - Mid-tier GPT-5 series
    - Advanced comprehension
6. **GPT-5** (`gpt-5`)
    - Flagship GPT-5 model
    - State-of-the-art performance

### Non-RAG System Prompt Design

The shared prompt instructs models to:

- Act as a legal consultant specializing in Saudi Labor Law
- Provide concise answers with simplified explanations when needed
- Avoid citing external sources
- Maintain formal Arabic legal terminology

## RAG Pipeline Models

These models retrieve relevant labor law articles before generating answers, grounding responses in authoritative source material.

### Architecture

1. **Query Processing**: User question is analyzed and potentially reformulated
2. **Retrieval**: Semantic search over labor law knowledge base (FAISS vector index)
3. **Context Assembly**: Retrieved articles are formatted into context
4. **Generation**: LLM generates answer using both the question and retrieved context
5. **Post-Processing**: Answer is validated and formatted

### Available Models

1. **RAG-GPT-5** (`chatbot-rag-gpt-5`)
    - GPT-5 with retrieval augmentation
    - Highest accuracy configuration
2. **RAG-GPT-4.1-mini** (`chatbot-rag-gpt-4.1-mini`)
    - Fast, cost-effective RAG pipeline
    - Good accuracy-to-cost ratio
3. **RAG-GPT-5-mini** (`chatbot-rag-gpt-5-mini`)
    - GPT-5 mini with retrieval
    - Enhanced reasoning over retrieved context
4. **RAG-GPT-4.1** (`chatbot-rag-gpt-4.1`)
    - Classic RAG configuration
    - Reliable baseline
5. **RAG-GPT-4o** (`chatbot-rag-gpt-4o`)
    - Optimized RAG variant
    - Balance of speed and accuracy
6. **RAG-GPT-5.1** (`chatbot-rag-gpt-5.1`)
    - Advanced RAG pipeline
    - Superior context utilization

### RAG System Design

The RAG pipeline leverages:

- **Vector Store**: FAISS index built from labor law corpus
- **Embeddings**: OpenAI `text-embedding-3-large` for semantic matching
- **Retrieval Strategy**: Top-k most relevant articles (configurable)
- **Context Injection**: Retrieved content is formatted with article numbers and chapter markers

### Model Selection Logic

The evaluation script defines models through the `experiments` dictionary, which maps:

```
experiment_label → answer_generation_function

```

To add or remove models:

1. Non-RAG: Add entry in `non_rag_models` dictionary
2. RAG: Call `make_rag_answer_for_model(model_name)` with desired model
3. Comment out models to exclude them from evaluation runs

---

# RAGAS Metrics Evaluation

### Overview

RAGAS (Retrieval-Augmented Generation Assessment) is a framework that evaluates RAG pipelines using both reference-based and reference-free metrics. The evaluation script uses RAGAS to compute objective quality scores.

### Evaluator Configuration

### Judge LLM

- **Model**: `gpt-4.1-nano-2025-04-14`
- **Purpose**: Powers reference-based metrics that require LLM reasoning
- **Timeout**: 3000 seconds per evaluation batch
- **Max Retries**: 4 attempts for failed requests
- **Temperature**: 1.0 (allows diverse evaluation reasoning)

### Embedding Model

- **Model**: `text-embedding-3-large`
- **Purpose**: Computes semantic similarity vectors
- **Provider**: OpenAI
- **Dimensionality**: 3072 dimensions

### Active Metrics

### 1. BLEU Score

**Type**: Reference-Based, Lexical

**Description**: Modified Precision metric that measures n-gram overlap between candidate and reference answers.

**How It Works**:
- Computes precision of unigrams, bigrams, trigrams, and 4-grams
- Applies brevity penalty to discourage overly short responses
- Scores range from 0.0 (no overlap) to 1.0 (perfect match)

**Interpretation**:
- **High Score (>0.5)**: Strong lexical similarity; answer uses similar wording to reference
- **Medium Score (0.2-0.5)**: Partial overlap; may paraphrase correctly
- **Low Score (<0.2)**: Different wording or missing key terms

**Legal Domain Relevance**:
- Useful for detecting whether specific legal terminology is preserved
- Sensitive to Arabic morphology and word forms

### 2. Semantic Similarity

**Type**: Reference-Based, Embedding-Based

**Description**: Measures cosine similarity between embedding vectors of candidate and reference answers.

**How It Works**:
- Embeds both answers using `text-embedding-3-large`
- Computes cosine similarity: (A · B) / (||A|| × ||B||)
- Scores range from -1.0 to 1.0 (normalized to 0.0-1.0 in practice)

**Interpretation**:
- **High Score (>0.85)**: Answers are semantically equivalent
- **Medium Score (0.70-0.85)**: Similar meaning with different phrasing
- **Low Score (<0.70)**: Different core meanings or missing information

**Legal Domain Relevance**:
- Captures paraphrasing while preserving legal intent
- Language-agnostic (works well with Arabic)
- Robust to synonym usage

### 3. Factual Correctness

**Type**: Reference-Based, LLM-Judged

**Description**: Uses an LLM to extract claims from both answers and verify if candidate claims are supported by reference claims.

**How It Works**:
1. **Claim Extraction**: Judge LLM identifies discrete factual claims in both answers
2. **Claim Matching**: Determines which candidate claims align with reference claims
3. **Scoring**: Ratio of supported claims to total candidate claims

**Interpretation**:
- **High Score (>0.8)**: Most legal facts are correctly stated
- **Medium Score (0.5-0.8)**: Some facts correct, some missing or wrong
- **Low Score (<0.5)**: Significant factual errors or hallucinations

**Legal Domain Relevance**:
- Critical for legal applications where factual precision is mandatory
- Detects misstatements of law, incorrect article citations, wrong penalties

### 4. Answer Accuracy

**Type**: Reference-Based, Token-Level Comparison

**Description**: Measures how much of the reference answer’s information is preserved in the candidate answer.

**How It Works**:
- Computes token-level recall: what fraction of reference content appears in candidate
- May use fuzzy matching for Arabic token variants
- Penalizes omissions more than additions

**Legal Domain Relevance**:
- Ensures completeness: legal advice must not omit critical conditions or exceptions
- Useful for detecting overly terse responses that skip important details

### Evaluation Workflow

### For Each Model Configuration:

1. **Subset Selection**: Filter golden set by question type (`direct` or `explanatory`)
2. **Answer Generation**: Invoke model’s generation function for each question
3. **Error Handling**: Catch exceptions; record empty string on failure
4. **Sample Construction**: Build `SingleTurnSample` objects with `user_input`, `response`, `reference`
5. **Dataset Assembly**: Create `EvaluationDataset` from all samples
6. **RAGAS Execution**: Call `evaluate()` with configured metrics, LLM, and embeddings
7. **Result Export**:
    - Per-sample CSV: `evaluation_outputs_chatbot_Rag_results/Results_chatbot_{model}_{subset}.csv`
    - Aggregate CSV: Mean scores across all samples

### Execution Configuration

- **RunConfig**: Timeout 350s per sample, max 4 parallel workers
- **Progress Display**: tqdm progress bars for visibility
- **Encoding**: UTF-8-sig for Excel compatibility with Arabic text

### Output Structure (RAGAS Layer)

Each per-sample CSV contains:
- `user_input`: Original question
- `response`: Model-generated answer
- `reference`: Gold-standard answer
- `question_type`: `direct` or `explanatory`
- `type`: Additional type metadata from golden set
- `bleu_score`: BLEU metric value
- `semantic_similarity`: Semantic similarity value
- `factual_correctness`: Factual correctness value
- `answer_accuracy`: Answer accuracy value

The aggregate CSV (`aggregate_scores_chatbot_models_by_type.csv`) shows mean scores per model-subset combination.

---

# LLM-Based Faithfulness Evaluation

### Purpose

While RAGAS metrics assess alignment with references, they don’t deeply analyze **legal faithfulness**: whether a response contains hallucinated legal rules, misquotes articles, or introduces critical contradictions. The LLM Judge layer addresses this gap.

### Architecture

### Input

- Per-model CSV files from RAGAS evaluation (located in `final generation model results before llm evaluation/new_results_folder`)
- Each file must contain: `user_input`, `response`, `reference`, `question_type`

### Output

- Augmented CSV files with four new columns:
    1. `faithfulness_score`: Numeric score in [0.0, 1.0]
    2. `faithfulness_verdict`: Categorical label
    3. `faithfulness_notes`: Arabic explanation from judge
    4. `faithfulness_model`: Judge model name for traceability

## Judge LLM Configuration

### Model Selection

- **Default**: `gpt-4.1`
- **Configurable**: Can specify via `-model` CLI argument (e.g., `gpt-4o-mini` for faster evaluation)
- **Temperature**: 0.0 (deterministic scoring)
- **Max Retries**: 3 attempts with 4-second delays
- **Timeout**: 180 seconds per request

### Structured Output Mode

The judge uses OpenAI’s **Structured Outputs** feature (JSON Schema mode) to guarantee machine-parsable responses. The required schema enforces:

```
{
  "key_points_total": integer (1-8),
  "key_points_covered": integer (≥0),
  "key_points_incorrect": integer (≥0),
  "has_critical_contradiction": boolean,
  "hallucination_level": "none" | "minor" | "major",
  "explanation": string
}
```

### System Prompt (Arabic)

The system message establishes the judge as a legal quality auditor with expertise in Saudi Labor Law. It provides detailed instructions:

**Step-by-Step Process**:
1. **Read Question**: Understand the legal context and scope
2. **Extract Key Points**: Identify 1-5 core legal rulings from the **reference answer only**
- Each ruling should be a distinct legal determination (condition, right, obligation, exception)
- Avoid splitting one ruling into multiple micro-points
3. **Compare Point-by-Point**: For each reference point, determine if the candidate:
- **Covers Correctly**: Expresses the same ruling, even with different wording
- **Covers Incorrectly**: Distorts the ruling (e.g., makes prohibited action permissible, cites wrong article)
- **Omits Entirely**: Does not mention the point at all
4. **Verify Article Numbers**: Check if article citations in candidate match reference; citing a different article with different rulings is a substantive error
5. **Detect Contradictions**: Flag explicit conflicts in legal interpretation or requirements
6. **Identify Hallucinations**: Note any fabricated legal information, article numbers, or penalties not in reference
7. **Assess Completeness**: Evaluate whether candidate covers all essential points from reference

### Evaluation Philosophy

- **Meaning Over Wording**: Semantic equivalence counts as correct, even with different phrasing
- **Strict on Substance**: Changing legal outcomes, article numbers, or conditions is incorrect
- **Context Matters**: Examples and ordering differences are acceptable if core rulings match
- **Neutral and Precise**: Judge must provide concise rationale for scoring decisions

### User Prompt (Arabic)

For each question-reference-candidate triplet, the prompt includes:
- Original question text
- Reference answer (marked as authoritative)
- Candidate answer (to be evaluated)
- Explicit instructions to fill structured fields only (no free-form text outside JSON)

### Scoring Algorithm

The final faithfulness score is computed in Python using the structured data from the judge:

### Base Score Calculation

1. **Coverage Ratio**:
    
    ```
    coverage_ratio = key_points_covered / key_points_total
    ```
    
    Measures what fraction of reference points are mentioned.
    
2. **Correctness Ratio**:
    
    ```
    correct_raw = key_points_covered - key_points_incorrect
    correctness_ratio = max(0, correct_raw / key_points_total)
    ```
    
    Measures what fraction of reference points are mentioned **and correct**.
    
3. **Weighted Base Score**:
    
    ```
    base_score = 0.7 × coverage_ratio + 0.3 × correctness_ratio
    ```
    
    Prioritizes coverage (70%) but penalizes errors (30%).
    

### Penalty Factors

**Hallucination Penalties**:
- `none`: 1.0 (no penalty)
- `minor`: 0.8 (20% reduction)
- `major`: 0.5 (50% reduction)

**Contradiction Penalty**:
- If `has_critical_contradiction == true`: multiply by 0.5
- Else: multiply by 1.0

### Final Score

```
final_score = base_score × hallucination_penalty × contradiction_penalty
```

Clipped to [0.0, 1.0].

### Verdict Labeling

Scores are mapped to categorical verdicts using fixed thresholds:

| Score Range | Verdict | Interpretation |
| --- | --- | --- |
| ≥ 0.80 | `correct` | High legal accuracy; minor or no errors |
| 0.50 - 0.79 | `partially_correct` | Some correct points but notable gaps or errors |
| < 0.50 | `hallucinated` | Major errors, fabrications, or contradictions |
| N/A | `unscored` | Missing data (empty reference or candidate) |
| N/A | `error` | LLM evaluation failed after retries |

**Important**: The verdict is derived **solely from the numeric score**, not from the LLM’s explanation. This ensures consistency and prevents the judge from overriding the scoring formula.

### Execution Flow

### Command-Line Interface

The script supports flexible configuration via CLI arguments:

**Key Parameters**:
- `--input-dir`: Directory containing RAGAS CSV outputs
- `--pattern`: Glob pattern to select files (default: `*.csv`)
- `--output-dir`: Where to write augmented files
- `--output-suffix`: Filename suffix for outputs (default: `_with_llm_faithfulness.csv`)
- `--model`: Judge model name (default: `gpt-4.1`)
- `--temperature`: Judge temperature (default: 0.0)
- `--max-rows`: Limit evaluation to first N rows (for testing)
- `--skip-existing`: Skip files that already have outputs
- `--max-retries`: Retry attempts for failed LLM calls (default: 3)
- `--retry-delay`: Seconds between retries (default: 4.0)
- `--timeout`: Per-request timeout (default: 180s)

### Output Structure (Faithfulness Layer)

Augmented CSV files contain all original columns plus:

| Column | Type | Description |
| --- | --- | --- |
| `faithfulness_score` | float or None | Computed faithfulness score (0.0-1.0) |
| `faithfulness_verdict` | string | `correct`, `partially_correct`, `hallucinated`, `unscored`, or `error` |
| `faithfulness_notes` | string | Arabic explanation from judge LLM |
| `faithfulness_model` | string | Judge model name (e.g., `gpt-4.1`) |

---

# Results Structure

### Directory Layout

```
evaluate_chatbot/
├── inputs/
│   └── GenAI-Law ChatBot Human Review Golden set.csv
├── final generation model results legal advisor/
│   ├── evaluate_llm_metrics.py
│   └── final_results/
│       ├── non-rag-gpt-4.1-mini_direct_with_eval_with_llm_faithfulness.csv
│       ├── non-rag-gpt-4.1-mini_explanatory_with_eval_with_llm_faithfulness.csv
│       ├── non-rag-gpt-4.1_direct_with_eval_with_llm_faithfulness.csv
│       ├── non-rag-gpt-4.1_explanatory_with_eval_with_llm_faithfulness.csv
│       ├── non-rag-gpt-4o_direct_cleaned_with_eval_with_llm_faithfulness.csv
│       ├── non-rag-gpt-4o_explanatory_cleaned_with_eval_with_llm_faithfulness.csv
│       ├── non-rag-gpt-5-mini_direct_cleaned_with_eval_with_llm_faithfulness.csv
│       ├── non-rag-gpt-5-mini_explanatory_cleaned_with_eval_with_llm_faithfulness.csv
│       ├── non-rag-gpt-5.1_direct_cleaned_with_eval_with_llm_faithfulness.csv
│       ├── non-rag-gpt-5.1_explanatory_cleaned_with_eval_with_llm_faithfulness.csv
│       ├── rag-gpt-4.1-mini_direct_with_eval_with_llm_faithfulness.csv
│       ├── rag-gpt-4.1-mini_explanatory_with_eval_with_llm_faithfulness.csv
│       ├── rag-gpt-4.1_direct_with_eval_with_llm_faithfulness.csv
│       ├── rag-gpt-4.1_explanatory_with_eval_with_llm_faithfulness.csv
│       ├── rag-gpt-4o_direct_with_eval_with_llm_faithfulness.csv
│       ├── rag-gpt-4o_explanatory_with_eval_with_llm_faithfulness.csv
│       ├── rag-gpt-5-mini_direct_with_eval_with_llm_faithfulness.csv
│       ├── rag-gpt-5-mini_explanatory_with_eval_with_llm_faithfulness.csv
│       ├── rag-gpt-5.1_direct_with_eval_with_llm_faithfulness.csv
│       └── rag-gpt-5.1_explanatory_with_eval_with_llm_faithfulness.csv
└── evaluation_outputs_chatbot_Rag_results/
    ├── Results_chatbot_{model}_{subset}.csv  (per-model RAGAS outputs)
    └── aggregate_scores_chatbot_models_by_type.csv
```

### File Naming Convention

### Pattern

```
{architecture}-{base_model}_{question_type}_with_eval_with_llm_faithfulness.csv
```

**Components**:
- **architecture**: `rag` or `non-rag`
- **base_model**: Base LLM identifier (e.g., `gpt-4.1`, `gpt-5-mini`)
- **question_type**: `direct` or `explanatory`
- **with_eval**: Indicates RAGAS metrics are included
- **with_llm_faithfulness**: Indicates faithfulness scores are included

### Examples

- `rag-gpt-5.1_explanatory_with_eval_with_llm_faithfulness.csv`: RAG pipeline with GPT-5.1, explanatory questions, full metrics
- `non-rag-gpt-4o_direct_cleaned_with_eval_with_llm_faithfulness.csv`: Non-RAG GPT-4o, direct questions, cleaned dataset, full metrics

### Per-Sample CSV Schema

Each row in the final results CSVs represents one evaluated question-answer pair.

### Core Fields

| Column | Type | Source | Description |
| --- | --- | --- | --- |
| `user_input` | string | Golden Set | Original question text |
| `reference` | string | Golden Set | Gold-standard reference answer |
| `response` | string | Model Generation | Model-generated answer |
| `question_type` | string | Golden Set | `direct` or `explanatory` |
| `type` | string | Golden Set | Additional type metadata |

### RAGAS Metrics (5 columns)

| Column | Type | Range | Description |
| --- | --- | --- | --- |
| `bleu_score` | float | [0.0, 1.0] | N-gram overlap with reference |
| `semantic_similarity` | float | [0.0, 1.0] | Embedding cosine similarity |
| `factual_correctness` | float | [0.0, 1.0] | LLM-judged factual alignment |
| `answer_accuracy` | float | [0.0, 1.0] | Reference information coverage |
| `response_relevancy` | float | [0.0, 1.0] | Question-answer relevance |

### Faithfulness Metrics (4 columns)

| Column | Type | Range/Values | Description |
| --- | --- | --- | --- |
| `faithfulness_score` | float | [0.0, 1.0] or None | Computed legal faithfulness score |
| `faithfulness_verdict` | string | See table below | Categorical faithfulness label |
| `faithfulness_notes` | string | N/A | Arabic rationale from judge |
| `faithfulness_model` | string | N/A | Judge model identifier |

**Faithfulness Verdict Values**:
- `correct`: High accuracy (score ≥ 0.80)
- `partially_correct`: Moderate accuracy (0.50 ≤ score < 0.80)
- `hallucinated`: Low accuracy (score < 0.50)
- `unscored`: Missing required data
- `error`: Evaluation failed

### Aggregate CSV Schema

The `aggregate_scores_chatbot_models_by_type.csv` file summarizes performance across models.

### Structure

| Column | Type | Description |
| --- | --- | --- |
| `model_subset` | string | Composite key: `{model}__{question_type}` |
| `bleu_score` | float | Mean BLEU score across all questions in subset |
| `semantic_similarity` | float | Mean semantic similarity |
| `factual_correctness` | float | Mean factual correctness |
| `answer_accuracy` | float | Mean answer accuracy |
| `response_relevancy` | float | Mean response relevancy |

**Note**: Faithfulness metrics are not aggregated in this file; they require per-sample analysis.

### Using the Results

### For Model Comparison

1. **Load Aggregate CSV**: Quick overview of model performance by type
2. **Filter by Architecture**: Compare all RAG vs. all non-RAG models
3. **Filter by Question Type**: Assess which models excel at direct vs. explanatory questions
4. **Rank by Metric**: Sort by factual correctness or faithfulness score to identify best models

### For Deep Analysis

1. **Load Per-Sample CSV**: Examine individual question-answer pairs
2. **Filter by Verdict**: Find all `hallucinated` responses for error analysis
3. **Read Faithfulness Notes**: Understand specific legal errors from judge rationale
4. **Correlate Metrics**: Check if low RAGAS scores correlate with faithfulness issues
5. **Identify Patterns**: Group by question type, article number, or chapter to find systematic weaknesses

### For Reporting

1. **Export to Excel**: Use UTF-8-sig encoding for Arabic compatibility
2. **Create Dashboards**: Visualize metric distributions, model rankings, verdict proportions
3. **Highlight Failures**: Filter and report critical legal errors (contradictions, major hallucinations)
4. **Track Improvements**: Compare evaluation results across model versions or configuration changes

---

# Complete Evaluation Workflow

This section describes the end-to-end process for evaluating legal advisor chatbot models.

## Phase 1: Data Preparation

### Step 1.1: Corpus Organization

- **Verify Source**: Ensure `source_data/labor_law_final_version_after_organization.jsonl` is up-to-date
- **Review Structure**: Confirm each line has `رقم الماده`, `اسم الفصل`, `الماده`

### Step 1.2: Generate Synthetic Q&A

```bash
# Run from project rootpython "Data generation/data_generation for chatbot.py"
```

- **Output**: `Data generation/eval_qa_gold_test_samples.jsonl`
- **Review**: Manually inspect samples for quality

### Step 1.3: Human Review and Curation

- **Import**: Load JSONL into spreadsheet or review tool
- **Validate**: Check question clarity, answer accuracy, article citation correctness
- **Correct**: Fix errors, improve phrasing, ensure completeness
- **Classify**: Verify `type` field is correctly labeled
- **Export**: Save as `evaluate_chatbot/inputs/GenAI-Law ChatBot Human Review Golden set.csv`

### Step 1.4: Pre-Evaluation Checks

- **Encoding**: Verify UTF-8-sig encoding for Arabic compatibility
- **Required Columns**: Confirm `Question`, `answer`, `Type` exist
- **No Nulls**: Ensure core fields have no missing values
- **Type Values**: Verify `Type` column contains only `direct` or `explanatory`

## Phase 2: RAGAS Evaluation

### Step 2.1: Configure Models

Edit `evaluate_chatbot.py`:
- **Enable/Disable Models**: Comment/uncomment entries in `experiments` dictionary
- **Set Limits**: Adjust `QUESTION_LIMIT` if doing smoke tests
- **Verify API Keys**: Ensure `OPENAI_API_KEY` is set in environment or `config/keys.py`

### Step 2.2: Run Evaluation

```bash
# Run from project rootpython evaluate_chatbot.py
```

- **Duration**: Varies by model count and question count (typically 30-120 minutes)
- **Progress**: Monitor tqdm progress bars for each model and subset
- **Output Directory**: `evaluation_outputs_chatbot_Rag_results/`

### Step 2.3: Review RAGAS Results

- **Per-Model Files**: Check `Results_chatbot_{model}_{subset}.csv` for individual responses
- **Aggregate File**: Review `aggregate_scores_chatbot_models_by_type.csv` for overall rankings
- **Quality Check**: Look for unexpected patterns (all zeros, identical answers, errors)

### Step 2.4: Organize for Faithfulness

- **Copy Files**: Move relevant CSVs to `final generation model results before llm evaluation/new_results_folder/`
- **Optional Cleaning**: Remove unnecessary metadata columns if desired
- **Verify Schema**: Confirm `user_input`, `reference`, `response`, `question_type` columns exist

## Phase 3: Faithfulness Evaluation

### Step 3.1: Configure Judge

Decide on judge model and parameters:
- **Fast**: Use `--model gpt-4o-mini` for quicker evaluation
- **Accurate**: Use `--model gpt-4.1` (default) for highest quality
- **Cost-Conscious**: Limit rows with `--max-rows 50` for testing

### Step 3.2: Run Faithfulness Scoring

```bash
# Run from project rootpython "evaluate_chatbot/final generation model results legal advisor/evaluate_llm_metrics.py" \    --model gpt-4.1 \    --skip-existing
```

- **Duration**: 2-5 minutes per 100 rows (varies by model and API rate limits)
- **Output Directory**: `final generation model results before llm evaluation/final_model_results_with_all_metrics_final_results/`

### Step 3.3: Incremental Updates

If evaluation is interrupted or you want to add models:
- **Use `--skip-existing`**: Avoids re-evaluating already-processed files
- **Add New CSVs**: Drop new RAGAS outputs into input directory
- **Re-run Command**: Script will only process new files

### Step 3.4: Move Final Results

```bash
# Copy augmented files to reporting directorymove "final generation model results before llm evaluation/final_model_results_with_all_metrics_final_results/*.csv" \     "evaluate_chatbot/final generation model results legal advisor/final_results/"
```

### Phase 4: Analysis and Reporting

### Step 4.1: Load Results

Using Python (pandas), Excel, or BI tool:

```python
import pandas as pd
# Load per-sample resultsdf = pd.read_csv(
    "evaluate_chatbot/final generation model results legal advisor/final_results/"    "rag-gpt-5.1_explanatory_with_eval_with_llm_faithfulness.csv",
    encoding="utf-8-sig")
# Load aggregate resultsagg_df = pd.read_csv(
    "evaluation_outputs_chatbot_Rag_results/"    "aggregate_scores_chatbot_models_by_type.csv",
    encoding="utf-8-sig")
```

### Step 4.2: Comparative Analysis

- **Best Overall Model**: Sort aggregate by factual correctness or faithfulness verdict distribution
- **RAG vs. Non-RAG**: Group by architecture; compute mean metrics
- **Question Type Performance**: Compare direct vs. explanatory subsets
- **Error Analysis**: Filter `faithfulness_verdict == 'hallucinated'`; read `faithfulness_notes`

### Step 4.3: Visualization

Create charts to illustrate:
- **Metric Distribution**: Box plots or histograms for each metric by model
- **Verdict Proportions**: Stacked bar chart showing correct/partial/hallucinated percentages
- **Correlation Matrix**: Heatmap of metric correlations
- **Question Type Comparison**: Side-by-side bar charts for direct vs. explanatory

### Step 4.4: Reporting

Generate final report including:
- **Executive Summary**: Best-performing models, key findings
- **Methodology**: Brief overview of golden set, metrics, judge model
- **Quantitative Results**: Tables of aggregate scores, verdict distributions
- **Qualitative Findings**: Examples of excellent answers and critical errors
- **Recommendations**: Which models to deploy, which need improvement

### Phase 5: Iteration and Improvement

### Improving Model Performance

Based on evaluation results:
- **Low Factual Correctness**: Improve retrieval precision; add more source articles
- **Low Response Relevancy**: Refine question understanding; adjust prompts
- **High Hallucination Rate**: Strengthen grounding instructions; use lower temperature
- **Poor Direct Question Performance**: Add more examples; fine-tune on application-oriented data

### Refining the Golden Set

- **Add Edge Cases**: Questions about rare provisions, ambiguous scenarios
- **Balance Distribution**: Ensure even coverage of law chapters, question types
- **Update References**: Keep golden answers aligned with latest legal amendments
- **Expand Size**: Generate more questions to increase statistical significance

### Adjusting Metrics

- **Remove Low-Value Metrics**: If BLEU shows no discriminative power, drop it
- **Add Custom Metrics**: Create domain-specific metrics (e.g., article citation accuracy)
- **Tune Thresholds**: Adjust faithfulness verdict thresholds based on risk tolerance

### Automating Workflow

- **CI/CD Integration**: Run evaluations automatically on model updates
- **Dashboard**: Build live dashboard for stakeholders to track metrics
- **Alerting**: Set up alerts for performance degradation or critical errors

---

## Conclusion

The Legal Advisor Chatbot evaluation framework provides a comprehensive, multi-layered assessment of Arabic legal question-answering systems. By combining:

1. **Structured Golden Set Generation**: Automated creation with human oversight ensures high-quality test data
2. **RAGAS Metrics**: Objective, reference-based evaluation across multiple quality dimensions
3. **LLM Judge Faithfulness**: Deep legal accuracy analysis with structured scoring
4. **Comparative Architecture Testing**: Direct comparison of RAG vs. non-RAG approaches

…the system enables rigorous, reproducible evaluation that meets the high standards required for legal advisory applications. The framework supports continuous improvement cycles and provides actionable insights for model selection and optimization.

---