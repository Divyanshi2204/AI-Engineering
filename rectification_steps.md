# Error Rectification & Learning Guide

This document explains the steps and commands used to resolve the `ModuleNotFoundError` issues in the Jupyter Notebook [day1.ipynb](file:///c:/Users/hp/llm_engineering/day1.ipynb).

---

## 1. Resolving the `dotenv` Import Error

### **The Error**
```python
ModuleNotFoundError: No module named 'dotenv'
```

### **Explanation**
The notebook tries to load environment variables using:
```python
from dotenv import load_dotenv
```
This requires the `python-dotenv` package, which was not installed in the current Python environment (virtual environment `.venv`).

### **Action & Command**
We installed the `python-dotenv` package to the project's dependencies and virtual environment using **`uv`**:
```bash
uv add python-dotenv
```
* **Why `uv add`?** In `uv`-managed projects, running `uv add <package>` automatically updates your `pyproject.toml` dependencies list and installs the package into the local `.venv`.

---

## 2. Resolving the `scraper` Import Error

### **The Error**
```python
ModuleNotFoundError: No module named 'scraper'
```

### **Explanation**
The notebook tries to import a custom function:
```python
from scraper import fetch_website_contents
```
There was no `scraper.py` file or package in the workspace root, which caused Python to throw a `ModuleNotFoundError`.

### **Action & Commands**
To solve this, we did two things:

1. **Install Scraper Dependencies**
   A typical web scraper requires the `requests` library to fetch HTML from a URL, and `beautifulsoup4` to parse that HTML and extract text. We installed these using `uv`:
   ```bash
   uv add requests beautifulsoup4
   ```

2. **Create the local `scraper.py` Module**
   We created a new file named [scraper.py](file:///c:/Users/hp/llm_engineering/scraper.py) in the workspace root directory with the following implementation:
   ```python
   import requests
   from bs4 import BeautifulSoup

   def fetch_website_contents(url: str) -> str:
       """
       Fetches the HTML content of the given URL and returns the parsed text content.
       """
       headers = {
           "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
       }
       try:
           response = requests.get(url, headers=headers, timeout=15)
           response.raise_for_status()
           soup = BeautifulSoup(response.content, "html.parser")
           
           # Remove script, style, and metadata elements that don't contain content
           for element in soup(["script", "style", "noscript", "iframe", "head", "title", "meta"]):
               element.decompose()
               
           # Get clean text
           text = soup.get_text(separator="\n")
           
           # Clean up extra whitespace and blank lines
           lines = (line.strip() for line in text.splitlines())
           chunks = (phrase.strip() for line in lines for phrase in line.split("  "))
           clean_text = "\n".join(chunk for chunk in chunks if chunk)
           
           return clean_text
       except Exception as e:
           return f"Error fetching website contents: {str(e)}"
   ```

---

## 3. Verifying the Solution

To test that everything was resolved correctly without launching the full Jupyter Notebook, we ran a one-line verification script using `uv run python`:

```bash
uv run python -c "from scraper import fetch_website_contents; print(fetch_website_contents('https://example.com')[:100])"
```
* **Explanation of the command:** 
  * `uv run python -c "..."` runs a Python command string within the context of the virtual environment (`.venv`).
  * It imports our newly created `fetch_website_contents` function, fetches the contents of `https://example.com`, and prints the first 100 characters of the parsed text.
  * The command succeeded and returned the text, proving both imports now work flawlessly!

---

## 4. Resolving the `openai` Import Error

### **The Error**
```python
ModuleNotFoundError: No module named 'openai'
```

### **Explanation**
The notebook `day1_with_ollama.ipynb` imports `openai`:
```python
from openai import OpenAI
```
This requires the `openai` Python package to be installed in the virtual environment.

### **Action & Command**
We installed the `openai` package using:
```bash
uv add openai
```
This downloads and installs the official OpenAI library and adds it to your project dependencies in `pyproject.toml`.

---

## 5. Troubleshooting Ollama & Stuck Jupyter Kernels

If you encounter issues where cells hang or fail to run, follow these steps to verify and resolve them.

### **A. Checking and Running Ollama**

1. **Verify if Ollama is running:**
   Open a terminal and run:
   ```bash
   ollama list
   ```
   * If it successfully prints a list of models, Ollama is running.
   * If it shows `Warning: could not connect to a running Ollama instance`, the service is offline.

2. **Start the Ollama Service:**
   * **Desktop App:** Search for **Ollama** in the Windows Start menu and run it. This adds a llama icon to your system tray.
   * **Command Line:** Run the following command in a terminal:
     ```bash
     ollama serve
     ```

### **B. Clearing Hung Jupyter Notebook Kernels**

If basic cells (e.g., `print("hello world")`) hang indefinitely:

1. **Restart Kernel:** Use the **Restart Kernel** button in the notebook interface.
2. **Force-Kill Stale Python Processes:** If the IDE is unresponsive, force-kill all running Python kernel instances using PowerShell:
   ```powershell
   Stop-Process -Name python -Force
   ```
   *Note: This will terminate all running Python processes, allowing you to establish a fresh connection in your IDE.*
