# Training Design Agent
**Oracle University | AI-Powered Instructional Design**

An AI agent that generates complete, Oracle-branded Training Design Documents from structured inputs and source content. Powered by Groq (LLaMA-3.3-70B) and built with Streamlit.

---

## Features

| Feature | Details |
|---|---|
| Multi-step UI | 3-screen wizard: Course Info → Source Content → Generate |
| LLM Backend | Groq API, LLaMA-3.3-70B-Versatile |
| Labs toggle | Auto-selects Yes-Labs or No-Labs instructional design template |
| File ingestion | PDF, DOCX, PPTX, XLSX, TXT, CSV — with OCR for scanned docs |
| URL crawler | Crawls base URL + child links (2 levels deep) |
| Copyright scanner | Flags protected content before sending to AI |
| Golden Standard | Optional reference doc to match tone/depth |
| Quality check | Post-generation section completeness audit |
| Reliability audit | Source tag count + Section → Source map |
| Traceability map | Every claim tagged with [FILE:] or [URL:] or [INPUT] |
| Feedback loop | Human-in-the-loop regeneration without starting over |
| Oracle branding | DOCX export with red header, colour-banded headings, formatted tables |
| PDF export | Via LibreOffice (requires system install) |
| Session persistence | All state preserved across steps |

---

## Setup

### 1. Clone / Download

```bash
git clone <your-repo>
cd training_design_agent
```

### 2. Install Python Dependencies

```bash
pip install -r requirements.txt
```

### 3. System Dependencies (for OCR and PDF conversion)

**Tesseract OCR** (for scanned PDFs):
```bash
# Ubuntu/Debian
sudo apt-get install tesseract-ocr poppler-utils

# macOS
brew install tesseract poppler
```

**LibreOffice** (for PDF export):
```bash
# Ubuntu/Debian
sudo apt-get install libreoffice

# macOS
brew install --cask libreoffice
```

> ⚠️ On Streamlit Community Cloud, Tesseract and LibreOffice require a `packages.txt` file.

### 4. Create `packages.txt` (Streamlit Cloud only)

```
tesseract-ocr
poppler-utils
libreoffice
```

### 5. Configure Secrets

Create `.streamlit/secrets.toml`:
```toml
GROQ_API_KEY = "gsk_your_actual_groq_key_here"
```

On Streamlit Cloud: go to **App Settings → Secrets** and paste:
```
GROQ_API_KEY = "gsk_your_actual_groq_key_here"
```

### 6. Run Locally

```bash
streamlit run app.py
```

---

## Project Structure

```
training_design_agent/
├── app.py                        # Entry point + router
├── requirements.txt
├── packages.txt                  # (create for Streamlit Cloud)
├── .streamlit/
│   ├── config.toml               # Oracle theme colours
│   └── secrets.toml.template     # Key template (don't commit actual keys)
├── pages/
│   ├── screen1.py                # Course Information & Target Audience
│   ├── screen2.py                # Source Content Inputs
│   └── screen3.py                # Generate, Display, Audit, Export
├── utils/
│   ├── session.py                # Session state initializer
│   ├── ui_helpers.py             # Oracle CSS theme + sidebar
│   ├── file_utils.py             # File extraction + copyright scanner
│   ├── crawler.py                # URL crawler (2 levels deep)
│   ├── llm_client.py             # Groq API client + prompt builder
│   └── export.py                 # DOCX/PDF generation with Oracle branding
└── prompts/
    ├── no_labs_prompt.py         # System prompt: courses without labs
    └── yes_labs_prompt.py        # System prompt: courses with hands-on labs
```

---

## Known Limitations

### LLM / Generation
| Limitation | Detail |
|---|---|
| Context window | LLaMA-3.3-70B supports ~128K tokens, but Groq's max_tokens is capped at 8,000 output tokens. Very large source docs will be truncated (first 8,000 chars per file). For very long docs, summarise content before uploading. |
| Hallucination risk | The system prompt instructs the model not to invent features. However, if source content is sparse, the model may fill gaps. Always review the Assumptions & Open Questions section. |
| Generation time | Expect 30–120 seconds depending on Groq load and input complexity. A progress bar is shown throughout. |
| One document at a time | Each generation is a single API call. No multi-turn memory beyond the feedback regeneration loop. |

### File Upload
| Limitation | Detail |
|---|---|
| File size | Streamlit Cloud default is 200MB per file (configured in config.toml). |
| Scanned PDFs | OCR requires Tesseract system install. Without it, image-based PDFs will not extract text. A clear warning is shown. |
| PPTX images | Text inside embedded images in PPTX slides is not extracted (only text shapes). |
| Password-protected files | Encrypted PDFs/DOCX will fail extraction. Remove passwords before uploading. |

### URL Crawler
| Limitation | Detail |
|---|---|
| Auth-gated pages | Pages behind login walls (Oracle Help Center authenticated pages) will not be crawled. |
| JavaScript-rendered pages | The crawler fetches raw HTML only. Pages that require JS to render content will return minimal text. |
| Rate limiting | Some sites may block the crawler after repeated requests. The agent uses a polite User-Agent string. |
| Depth cap | Crawls 2 levels deep, max 15 links per level. Very large documentation sites will not be fully indexed. |

### Export
| Limitation | Detail |
|---|---|
| PDF conversion | Requires LibreOffice installed on the server. On Streamlit Cloud, add `libreoffice` to `packages.txt`. |
| Complex markdown tables | Very wide tables (6+ columns) may overflow page margins in DOCX. |
| Images | No images are embedded in the DOCX export (text-only output). |

### Concurrency
| Limitation | Detail |
|---|---|
| Session isolation | Each browser session has its own state. Multiple concurrent users are supported by Streamlit's session model. |
| Groq rate limits | Groq free tier has rate limits. Under heavy concurrent load, requests may be queued or throttled. |

---

## Deployment on Streamlit Community Cloud

1. Push this repo to GitHub (ensure `.streamlit/secrets.toml` is in `.gitignore`).
2. Go to [share.streamlit.io](https://share.streamlit.io) → New App.
3. Connect your repo, set `app.py` as the main file.
4. Under **Advanced Settings → Secrets**, paste your `GROQ_API_KEY`.
5. Create `packages.txt` in the repo root with system dependencies.
6. Deploy.
