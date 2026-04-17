# Technical Architecture & Automation Pipeline for Academic RAG System

To ensure real citations and high-quality academic flow, this framework uses a Retrieval-Augmented Generation (RAG) pipeline integrated with actual academic databases and a multi-agent workflow.

## 1. The Research & Retrieval Engine (The Secret Sauce)
Instead of asking the AI to "write a section," the system automates the research phase first to ground the text in reality.

**Process:**
1. **Academic APIs:** Connect the backend to open scientific databases.
2. **Automation Step:** When a student inputs a topic (e.g., "L'impact de l'IoT sur l'agriculture"), the system automatically queries these APIs to download abstracts and metadata of real, existing papers.
3. **Vector Database:** Store these retrieved papers in a vector database so the AI can "read" them.

**Recommended Free Tools:**
* **Academic APIs:** 
  * **OpenAlex:** The most comprehensive completely open and free catalog of global research. 
  * **Semantic Scholar API:** Extremely generous free tier with great abstract coverage.
  * **Crossref / CORE:** Free APIs for academic metadata and open-access papers.
* **Vector Database:**
  * **ChromaDB:** Open-source and runs locally. Perfect for Python projects without cloud costs.
  * **FAISS:** A library completely free and created by Meta, optimal for fast similarity search locally.
* **Embedding Model (for Vector DB):**
  * **Hugging Face (`sentence-transformers`):** Allows you to generate embeddings locally for free (no API key needed).

---

## 2. The Multi-Step Drafting Workflow (Agentic AI)
Break the writing process into automated, isolated tasks to prevent generic "AI-speak" and ensure strict adherence to citations.

**Process:**
* **Agent 1 (The Outliner):** Generates the standard academic structure (Introduction, État de l'art, Méthodologie, Résultats, Conclusion).
* **Agent 2 (The Researcher):** Pulls facts from the Vector Database and maps them to the outline.
* **Agent 3 (The Writer):** Drafts the text strictly using the retrieved facts, ensuring in-text citations (e.g., [Smith et al., 2022]) are tied directly to the bibliography.
* **Agent 4 (The Editor):** Scans the text to remove generic AI transition phrases (e.g., "En conclusion, il est important de noter que...") and formats it strictly for academic tone.

**Recommended Free Tools:**
* **Orchestration / Multi-Agent Framework:** 
  * **CrewAI / LangGraph / AutoGen:** All are completely open-source Python frameworks. CrewAI is highly recommended for defining "roles" (Outliner, Researcher, Writer, Editor) very easily.
* **LLM Engine:**
  * **Ollama:** Run advanced open-source models (like Llama 3 or Mistral) locally on your own machine entirely for free.
  * **Groq API Cloud:** Offers a very generous free tier with incredibly fast inference if your local hardware isn't powerful enough.
  * **Hugging Face Inference API:** Free tier available for various models.

---

## 3. The Export Module
Convert the final agent-approved text into beautiful, structured documents.

**Recommended Free Tools:**
* **DOCX:** 
  * **`python-docx`:** Free Python library to generate structural Word documents containing proper heading styles (Crucial for auto table of contents).
* **LaTeX:** 
  * **`PyLaTeX`:** A seamless free Python library that helps you write `.tex` files dynamically. Highly attractive for STEM students.
* **PDF:** 
  * **`Pandoc`:** The universal document converter. It can take your markdown or LaTeX output and convert it into beautiful PDFs using `pdflatex` or `xelatex`.
  * **`WeasyPrint` / `ReportLab`:** Open-source Python libraries for generating PDFs dynamically.

---

### Suggested Tech Stack Summary (100% Free & Local-Friendly):
- **Language**: Python
- **APIs**: Semantic Scholar / OpenAlex
- **LLM / Embeddings**: Ollama (Llama 3) + HuggingFace
- **Agent Orchestration**: CrewAI
- **Vector DB**: ChromaDB
- **Document Generation**: `python-docx` + `PyLaTeX`
