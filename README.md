# Telegram RAG Bot

A lightweight Retrieval-Augmented Generation (RAG) bot built on Telegram that answers questions from a local knowledge base using sentence embeddings and LLaMA 3.3.

**Author:** Sambit Sahoo

---

## Features

- `/ask <query>` — retrieves relevant answers from the knowledge base
- `/help` — shows usage instructions
- `/history` — shows your last 3 questions
- `/image` — placeholder for image support
- Query caching — identical queries skip the LLM call
- Source snippets — every answer shows which document it came from
- Per-user message history — last 3 interactions tracked per user

---

## System Design

```
User (Telegram)
      │
      ▼
Telegram Servers
      │  polling
      ▼
python-telegram-bot (Colab)
      │
      ├── /help, /start, /history, /image
      │         (handled directly)
      │
      └── /ask <query>
                │
                ▼
         Query Cache (dict)
         hit? ──► return cached answer
         miss?
                │
                ▼
         Sentence Transformer
         (all-MiniLM-L6-v2)
         encode query → vector
                │
                ▼
         sqlite-vec (rag.db)
         native vector search
         top-3 similar chunks
                │
                ▼
         Prompt Builder
         context + question
                │
                ▼
         LLaMA 3.3 (OpenRouter API)
         generate answer
                │
                ▼
         Reply to user
         answer + 📄 sources
```

---

## Tech Stack

| Component     | Technology                              |
|---------------|-----------------------------------------|
| Bot           | python-telegram-bot v21                 |
| Embeddings    | sentence-transformers (all-MiniLM-L6-v2)|
| Vector DB     | sqlite-vec (native vector search)       |
| LLM           | LLaMA 3.3-70b via OpenRouter (free)     |
| Runtime       | Google Colab                            |
| Tunnel        | pyngrok (for webhook, optional)         |

### Model Justification

The assignment recommends Ollama (LLaMA 3, Mistral, Phi-3) or OpenAI API. Ollama requires a local installation and cannot run on Google Colab. We use **LLaMA 3.3-70b** served via OpenRouter's free API tier — functionally identical to running LLaMA 3 locally via Ollama, but cloud-hosted for Colab compatibility. No credit card required.

`all-MiniLM-L6-v2` was chosen for embeddings because it is small (80MB), fast on CPU, and produces high-quality semantic embeddings for retrieval tasks.

---

## Knowledge Base

The bot answers questions from 4 documents in the `/docs` folder:

| File            | Contents                        |
|-----------------|---------------------------------|
| `faq.md`        | General FAQ (hours, shipping)   |
| `policies.md`   | Refund, privacy, account policy |
| `products.md`   | Pricing plans and free trial    |
| `technical.md`  | API docs, auth, rate limits     |

---

## How to Run Locally

### Prerequisites

- Python 3.8+
- A Telegram bot token from [@BotFather](https://t.me/BotFather)
- A free OpenRouter API key from [openrouter.ai](https://openrouter.ai)

### Installation

```bash
git clone https://github.com/your-username/telegram-rag-bot
cd telegram-rag-bot
pip install python-telegram-bot[webhooks] sentence-transformers sqlite-vec openai pyngrok nest_asyncio
```

### Configuration

Open `bot.py` and set your tokens:

```python
BOT_TOKEN = "your-telegram-bot-token"
OPENROUTER_API_KEY = "your-openrouter-api-key"
```

### Run

```bash
# Step 1 — ingest documents
python ingest.py

# Step 2 — start the bot
python bot.py
```

### Run on Google Colab

1. Run Cell 1 — install dependencies
2. Run Cell 2 — create knowledge base documents
3. Run Cell 3 — ingest, chunk, embed, store in sqlite-vec
4. Run Cell 4 — load retriever + LLM
5. Run Cell 5 — start ngrok tunnel (optional, for webhook)
6. Run Cell 6 — start the Telegram bot

> Cell 6 will hang indefinitely — this is normal. It means the bot is live and listening.

---

## Optional Enhancements Implemented

- **Query caching** — repeated queries return instantly without calling the LLM
- **Source snippets** — every response shows which document(s) were used
- **Message history** — `/history` shows the last 3 questions per user

---

## Evaluation Notes

| Criteria       | Implementation                                              |
|----------------|-------------------------------------------------------------|
| Code Quality   | Modular cells: ingest, retrieve, LLM, bot all separated     |
| System Design  | Clear RAG pipeline with cache and vector search             |
| Model Use      | LLaMA 3.3 via OpenRouter, all-MiniLM-L6-v2 for embeddings  |
| Efficiency     | sqlite-vec native search, query cache avoids redundant calls|
| UX             | Source snippets, history command, clear error messages      |
