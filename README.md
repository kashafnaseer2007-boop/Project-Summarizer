# T5 Summarizer

A clean, easy-to-use web app that summarizes long text using a fine-tuned **T5 (Text-to-Text Transfer Transformer)** model. Built with **FastAPI** on the backend and a sleek **HTML/Tailwind CSS** frontend.

---

## What Does This Do?

You paste any long text — articles, essays, news, research papers — and the app returns a short, concise summary. It runs entirely on your local machine using your own T5 model. No cloud services, no API keys needed.

---

## Features

- Clean, modern chat-style UI (inspired by Claude/ChatGPT layouts)
- Paste text and get a summary in seconds
- Shows input/output character count for each summary
- Runs 100% locally — your data never leaves your machine
- Built-in API so you can use it programmatically too
- Health check endpoint to verify the model is running

---

## Folder Structure

Before running anything, make sure your project folder looks like this:

```
t5-summarizer/
├── main.py              ← FastAPI backend (serves the API + frontend)
├── index.html           ← Web UI (the chat-style interface)
├── requirements.txt      ← Python dependencies
└── model/               ← Your fine-tuned T5 model files
    ├── config.json
    ├── pytorch_model.bin
    ├── tokenizer_config.json
    ├── vocab.json
    ├── spiece.model      (or tokenizer files you have)
    └── ...
```

> **Important:** The `model/` folder must contain a valid T5 model and tokenizer. This app does not download a model for you — you need to provide your own fine-tuned model.

---

## Prerequisites

Make sure you have these installed on your Windows machine:

| Software | What It Is | How to Check |
|----------|-----------|--------------|
| **Python** (3.8+) | The programming language | Open CMD and run: `python --version` |
| **pip** | Python package manager | `pip --version` |

If you don't have Python, download it from [python.org](https://www.python.org/downloads/) — **check the box that says "Add Python to PATH"** during installation.

---

## Installation

Open **Command Prompt** (search "cmd" in Start menu) and follow these steps:

### Step 1: Navigate to your project folder

```cmd
cd C:\path\to\your\t5-summarizer
```

Replace `C:\path\to\your\t5-summarizer` with the actual path to your project.

### Step 2: Create a virtual environment (recommended)

A virtual environment keeps your project's dependencies separate from other Python projects.

```cmd
python -m venv venv
```

### Step 3: Activate the virtual environment

```cmd
venv\Scripts\activate
```

You'll know it worked when you see `(venv)` at the start of your command prompt:

```
(venv) C:\path\to\your\t5-summarizer>
```

### Step 4: Install the required packages

```cmd
pip install fastapi uvicorn torch transformers pydantic
```

This installs everything your app needs. If you already have a `requirements.txt`, you can use:

```cmd
pip install -r requirements.txt
```

> **Note:** The `torch` package is large (~2 GB). If you have an NVIDIA GPU, install the CUDA version from [pytorch.org](https://pytorch.org/get-started/locally/) for faster inference. The CPU version works fine too — just slower.

---

## Running the App

With your virtual environment activated, run:

```cmd
uvicorn main:app --reload --port 8000
```

Here's what each part means:

| Part | What It Does |
|------|-------------|
| `uvicorn main:app` | Starts the server using the `app` object from `main.py` |
| `--reload` | Auto-restarts the server when you edit `main.py` (great for development) |
| `--port 8000` | Runs the app on port 8000 |

You should see output like this:

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [12345]
INFO:     Loading model from ./model on cuda (or cpu)...
INFO:     ✅ Model loaded successfully
INFO:     Started server process [12346]
INFO:     Application startup complete.
```

Now open your browser and go to:

```
http://localhost:8000
```

You should see the T5 Summarizer web interface. Paste some text and hit Enter!

---

## How to Use the Web UI

1. **Open** `http://localhost:8000` in your browser
2. **Paste** any long text into the input box at the bottom
3. **Press Enter** (or click the send button)
4. **Wait** a moment — the model will process your text
5. **Read** the summary that appears in the chat

> **Tip:** The model needs at least 10 characters to summarize. If you type something too short, the input box will flash red.

---

## API Endpoints

Besides the web UI, your app also exposes these API endpoints that you can use from scripts or other apps:

### `GET /`
Serves the web frontend (the HTML page).

### `POST /summarize`
The main endpoint — sends text and receives a summary.

**Request body (JSON):**

```json
{
  "text": "Your long article text goes here...",
  "max_length": 150,
  "min_length": 30
}
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `text` | string | Yes | — | The text to summarize (minimum 10 characters) |
| `max_length` | integer | No | 150 | Maximum summary length in tokens (20–512) |
| `min_length` | integer | No | 30 | Minimum summary length in tokens (10–200) |

**Example using curl:**

```cmd
curl -X POST http://localhost:8000/summarize -H "Content-Type: application/json" -d "{\"text\": \"The Apollo program was a human spaceflight program carried out by NASA...\"}"
```

**Response (JSON):**

```json
{
  "summary": "The Apollo program was NASA's effort to land humans on the Moon...",
  "input_length": 1234,
  "output_length": 87
}
```

### `GET /health`
Check if the server and model are running.

**Response:**

```json
{
  "status": "ok",
  "device": "cuda",
  "model_loaded": true
}
```

### `GET /docs`
Auto-generated interactive API documentation (Swagger UI). Great for testing endpoints directly in your browser.

---

## Configuration

You can tweak these settings in `main.py`:

| Setting | Where | Default | Description |
|---------|-------|---------|-------------|
| Model path | `MODEL_PATH` | `"./model"` | Folder containing your T5 model |
| Max input length | `tokenizer.encode(..., max_length=1024)` | 1024 | Max tokens the model processes at once |
| Num beams | `model.generate(..., num_beams=4)` | 4 | Higher = better quality but slower |
| No repeat n-grams | `no_repeat_ngram_size=3` | 3 | Prevents repeated phrases in the summary |

---

## Troubleshooting

### "ModuleNotFoundError: No module named 'fastapi'"

You forgot to install the packages, or you're not in the virtual environment. Run:

```cmd
venv\Scripts\activate
pip install fastapi uvicorn torch transformers pydantic
```

### "Failed to load model"

Make sure your `model/` folder is in the same directory as `main.py` and contains all the necessary files (`pytorch_model.bin`, `config.json`, tokenizer files, etc.).

### "Address already in use" (port 8000 is busy)

Something else is using port 8000. Either close that program, or use a different port:

```cmd
uvicorn main:app --reload --port 8080
```

Then go to `http://localhost:8080` instead.

### The page loads but summarization fails

1. Check the CMD window for error messages
2. Make sure the model loaded successfully — you should see `✅ Model loaded successfully`
3. Try the `/health` endpoint: `http://localhost:8000/health`
4. If using GPU, make sure you have the correct CUDA version of PyTorch installed

### The summary quality is poor

- Try a longer input text (the model works best with at least a few sentences)
- Adjust `max_length` and `min_length` in the API request
- Your model quality depends on the training data — make sure it was fine-tuned on a summarization dataset (like CNN/DailyMail)

### How to stop the server

Press `Ctrl + C` in the CMD window that's running the server.

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Backend | FastAPI (Python) |
| Model | T5 (Hugging Face Transformers) |
| Frontend | HTML, Tailwind CSS, Vanilla JavaScript |
| Server | Uvicorn (ASGI) |
| Deep Learning | PyTorch |

---

## License

This project is for personal/educational use. Check the license of your fine-tuned T5 model for any restrictions.
