# `bot_does_things`

A collection of useful Python functions that can be used as tools for LangChain/LangGraph agents. The focus of this package is on tools which can be run locally - all functionalities relating to running models locally (i.e. image interpretation) are set up to require a local LLM server like Ollama, but this can be altered to use other OpenAI API-compatible providers.

## Installation

This repo is set up as a package for `uv`.

```bash
uv sync
```

## Configuration

### Serper (web search)

`web_search()` uses Serper via LangChain’s `GoogleSerperAPIWrapper`. Requires setting the `SERPER_API_KEY` environment variable.

```bash
export SERPER_API_KEY="..."
```

### Ollama (image interpretation)

`interpret_image()` uses an OpenAI-compatible endpoint (defaults to Ollama).

```bash
export OLLAMA_BASE_URL="http://localhost:11434/v1"
export OLLAMA_IMAGE_INTERPRETATION_MODEL="gemma3:4b"
```

`OPENAI_API_KEY` is read but defaults to `ollama`.

### Download directory

Some tools save files under:

- `DOWNLOAD_DIR = "./data/downloads"`

## Available tools

All tools are plain Python callables and can be imported from `bot_does_things.tools`.

### `read_file(file_path: str) -> str`

Reads a text file.

```python
from bot_does_things.tools import read_file

text = read_file("./notes.txt")
```

### `read_table(file_path: str) -> str`

Reads a table file and returns a string representation.

Supported formats:

- `.xlsx`
- `.csv`
- `.tsv`
- `.json`
- `.parquet`

```python
from bot_does_things.tools import read_table

print(read_table("./data.csv"))
```

### `web_search(query: str, max_results: int = 10) -> list[dict]`

Performs a web search (Serper/Google) and returns the standard Serper `organic` results list.

```python
from bot_does_things.tools import web_search

results = web_search("Atalanta 41 0335-0347 pdf", max_results=5)
for r in results:
    print(r["title"], r["link"])
```

### `retrieve_webpage(url: str, only_text: bool = True) -> str`

Fetches a webpage and converts it.

- If the page contains a `<main>` element, only that content is converted.
- If `only_text=True` (default), returns plain text only.
- If `only_text=False`, returns Markdown and downloads images into `DOWNLOAD_DIR`, rewriting image URLs to local paths.

```python
from bot_does_things.tools import retrieve_webpage

text = retrieve_webpage("https://www.wikipedia.org/", only_text=True)
md = retrieve_webpage("https://www.wikipedia.org/", only_text=False)
```

### `download_and_read_pdf(pdf_url: str) -> str`

Downloads a PDF from a URL into `DOWNLOAD_DIR` and returns extracted text.

```python
from bot_does_things.tools import download_and_read_pdf

txt = download_and_read_pdf("https://www.zobodat.at/pdf/Atalanta_41_0335-0347.pdf")
```

### `load_pdf(file_path: str) -> str`

Loads a local PDF and returns extracted text (via `PyPDFLoader`).

```python
from bot_does_things.tools import load_pdf

txt = load_pdf("./data/downloads/paper.pdf")
```

### `interpret_image(image_path: str, query: str) -> str`

Sends an image + prompt to an OpenAI-compatible vision model (defaults to Ollama) and returns the model response.

```python
from bot_does_things.tools import interpret_image

answer = interpret_image("./image.png", "What is shown in this image?")
```

## Quick sanity check

`bot_does_things.tools` contains a minimal `__main__` runner that calls each tool once.

```bash
uv run python -m bot_does_things.tools