# Dual Model Reviewer

A Streamlit chat app that routes every prompt through two — or all three — major AI providers in sequence: the first model answers, the second reviews and refines, and an optional third synthesizes the final answer.

---

## How it works

Each message triggers a multi-call pipeline:

1. **First model** — receives your prompt with full conversation history and generates an initial answer.
2. **Reviewer model** — receives the original question and the first model's response, then critiques and improves it. Stateless: sees only the current turn, not prior history.
3. **Synthesizer model** *(optional, "Run all three" mode)* — receives the question, the initial answer, and the review, then delivers the single best final answer.

The answer shown in the chat is always the last model's output. The initial response and the review are preserved in collapsible expanders beneath each turn.

---

## Providers and models

| Provider | Role indicator | Available models |
|---|---|---|
| Claude (Anthropic) | 🟣 | Opus 4.8, Sonnet 4.6, Haiku 4.5 |
| Gemini (Google) | 🔵 | 2.5 Flash, 2.5 Pro, 3.5 Flash |
| ChatGPT (OpenAI) | 🟢 | GPT-4o, GPT-4o mini, GPT-4.1, GPT-4.1 mini |

Any provider can fill any role. The sidebar lets you pick the answerer, reviewer, and (in three-model mode) the synthesizer independently.

---

## Prerequisites

- Python 3.10 or later
- Windows *(PDF export uses fonts from `C:\Windows\Fonts` — see [Limitations](#limitations))*
- API keys for each provider you want to use

---

## Installation

**1. Clone or copy this folder** to any location on your machine.

**2. Install Python dependencies:**

```
pip install streamlit anthropic google-genai openai fpdf2 python-docx
```

**3. Set up the API usage tracker** *(optional — logs token usage and cost to a local SQLite database):*

The file `dual_model_reviewer.py` imports `api_usage_tracker.py` from a separate folder. Update the path near the top of that file to match where you put it:

```python
# dual_model_reviewer.py, line ~14
sys.path.insert(0, r"C:\path\to\your\api_usage_tracker_folder")
```

If you don't want usage tracking at all, remove the three `log_from_*` import lines and their call sites in `dual_model_reviewer.py`.

**4. Set environment variables** for each provider:

```
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=AI...
OPENAI_API_KEY=sk-...
```

You only need keys for the providers you plan to use. Missing keys are flagged in the UI but don't break the other two providers.

---

## Running the app

Double-click `launch_reviewer.bat`, or from a terminal:

```
python -m streamlit run app.py --server.port 8502
```

Then open `http://localhost:8502` in your browser.

---

## Features

### Model and role selection (sidebar)

- Choose the specific model version for each provider independently.
- Pick which provider answers first and which reviews.
- Check **Run all three models** to enable the synthesizer pass. The third provider is auto-assigned as whichever of the three isn't already the answerer or reviewer.

### File attachments

- **Images** (JPG, PNG, GIF, WebP) — sent natively to the first model.
- **PDFs** — sent to Claude or Gemini as native documents. OpenAI does not receive PDFs in this integration; a warning appears if ChatGPT is selected as the first model when a PDF is attached.
- **TXT / DOCX** — text is extracted and prepended to your prompt.

Attachments apply to the current message only and clear automatically after sending.

### Conversation archive

- **Archive** saves the current conversation to `archives.json` in the project folder.
- Archived conversations are listed in the sidebar. Clicking an entry loads it back as the active chat.
- Each archive entry has a collapsible preview and its own PDF export button.

### PDF export

Exports the full conversation — including both the initial response and the reviewer's output — to a timestamped PDF file. The exporter strips emoji and unsupported Unicode characters automatically before rendering.

### Cost tracking

Each turn shows a per-call token count and estimated cost. A running session total appears in the sidebar. Rates are defined in the `PRICING` dict in `dual_model_reviewer.py` — update them there if provider pricing changes.

---

## Project structure

```
Dual Model Reviewer/
├── app.py                  # Streamlit UI — layout, session state, chat handler
├── dual_model_reviewer.py  # API dispatch layer — calls Claude, Gemini, OpenAI
├── archives.json           # Persisted conversation archive (created on first save)
├── launch_reviewer.bat     # Windows launcher (port 8502)
└── reviewer_icon.ico       # Taskbar icon
```

---

## Limitations

- **Conversation history** is maintained only for the first (answering) model. The reviewer and synthesizer are stateless — they see only the current turn, not previous exchanges.
- **PDF attachments** are not forwarded to OpenAI. Use Claude or Gemini as the first model when working with PDFs.
- **PDF export** uses Segoe UI from `C:\Windows\Fonts` and is Windows-only. To run on macOS or Linux, update the font paths in `generate_pdf()` in `app.py` to point to fonts available on that system.
- **Extended thinking** is enabled by default on all Claude calls. It cannot be disabled per-call from the UI.
