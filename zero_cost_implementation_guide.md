# Zero-Cost Implementation Guide: Academic RAG System

This guide outlines exactly how to build your academic multi-agent RAG workflow for exactly **$0**. We will use Python, local databases, and generous free-tier APIs to ensure this project costs you absolutely nothing.

## Step 1: The Zero-Cost Tech Stack

* **Language:** Python 3.10+
* **Vector Database:** `chromadb` (Runs entirely on your local machine, $0)
* **Embedding Model:** `sentence-transformers` (Runs locally via HuggingFace, $0)
* **LLM Engine:** Groq API (Provides Llama 3 70B with incredibly fast speeds and a generous free tier) or Ollama (Runs models completely offline, $0)
* **Multi-Agent Framework:** `crewai` ($0 open source)
* **Academic API:** Semantic Scholar Graph API (Provides public endpoints requiring no API key for basic rate limits, $0)
* **Exporting:** `python-docx` and `pylatex` ($0 open source)

## Step 2: Environment Initialization

First, create a virtual environment and install the required free packages:

```bash
mkdir zero_cost_rag
cd zero_cost_rag
python -m venv venv
source venv/bin/activate  # On Linux/Mac
# venv\Scripts\activate   # On Windows

pip install crewai langchain-groq chromadb sentence-transformers requests python-docx pylatex
```

## Step 3: Building the Research API Connector

We will use the free Semantic Scholar API to fetch metadata and abstracts to ground our AI in reality.

```python
import requests

def search_academic_papers(query, limit=5):
    """Fetches real papers from Semantic Scholar without requiring an API key."""
    url = f"https://api.semanticscholar.org/graph/v1/paper/search?query={query}&limit={limit}&fields=title,authors,year,abstract,url"
    response = requests.get(url)
    
    if response.status_code == 200:
        return response.json().get('data', [])
    return []

# Example Usage:
# papers = search_academic_papers("IoT agricultural impact")
```

## Step 4: The Local Vector Database ($0)

Instead of paying for cloud databases like Pinecone, we use ChromaDB with local HuggingFace embeddings.

```python
import chromadb
from chromadb.utils import embedding_functions

# Initialize local ChromaDB client (saves permanently to your hard drive)
client = chromadb.PersistentClient(path="./local_vector_db")

# Use HuggingFace's free local embedding model (downloads automatically the first time)
sentence_transformer_ef = embedding_functions.SentenceTransformerEmbeddingFunction(model_name="all-MiniLM-L6-v2")

# Create or connect to a collection
collection = client.get_or_create_collection(
    name="academic_papers", 
    embedding_function=sentence_transformer_ef
)

def store_papers(papers):
    """Stores fetched papers into the vector database."""
    for i, paper in enumerate(papers):
        abstract = paper.get('abstract')
        if abstract:
            collection.add(
                documents=[abstract],
                metadatas=[{
                    "title": paper['title'], 
                    "year": str(paper.get('year', 'N/A')), 
                    "url": paper.get('url', '')
                }],
                ids=[f"paper_{i}"]
            )
    print("Papers stored in ChromaDB.")

def search_vector_db(query, n_results=3):
    """Searches the database for content matching the AI's current context."""
    results = collection.query(
        query_texts=[query],
        n_results=n_results
    )
    return results
```

## Step 5: Setting Up The Brains (Groq API)

Groq provides blazing-fast Llama 3 models for free. 
1. Get a free API key at [console.groq.com](https://console.groq.com).
2. Configure it in your terminal: `export GROQ_API_KEY='your_key'`

```python
import os
from langchain_groq import ChatGroq

# Initialize the free Groq LLM
llm = ChatGroq(
    api_key=os.environ.get("GROQ_API_KEY"),
    model="llama3-70b-8192" # Massive 70 Billion parameter reasoning power for free
)

# Note: If you want 100% offline, you can swap this to use Ollama running locally.
```

## Step 6: Creating the Multi-Agent Workflow (CrewAI)

CrewAI allows you to automate the entire drafting process by creating isolated intelligent agents.

```python
from crewai import Agent, Task, Crew

# 1. Define the Agents
outliner_agent = Agent(
    role="Academic Outliner",
    goal="Create a structured outline for an academic paper based on a specific topic.",
    backstory="You are an expert academic planner who structures university thesis papers.",
    llm=llm
)

research_writer_agent = Agent(
    role="Academic Writer",
    goal="Write the academic paper rigorously using ONLY the provided vector DB search results. You must cite sources inline natively (e.g., [Title, Year]).",
    backstory="You are a strict academic author. You refuse to hallucinate. You only write facts provided in your context.",
    llm=llm
)

editor_agent = Agent(
    role="Humanizing Academic Editor",
    goal="Rewrite the drafted text to bypass AI detection algorithms by increasing burstiness and perplexity.",
    backstory="You are an expert human academic editor. Your task is to bypass AI detection. You heavily vary sentence lengths—using very short statements followed by complex, compound sentences. You use unconventional but appropriate vocabulary and avoid robotic transition phrases like 'In conclusion' or 'It is important to note.'",
    llm=llm
)

# 2. Integrate the Workflow
topic = "The use of IoT in smart agriculture"

# First, search our local vector DB for facts to feed the writer
retrieved_context = str(search_vector_db(topic)) 

outline_task = Task(
    description=f"Create a 3-section outline for: {topic}. Output in Markdown.",
    expected_output="A bulleted markdown outline.",
    agent=outliner_agent
)

drafting_task = Task(
    description=f"Using the outline created, and ONLY these retrieved facts: {retrieved_context}, draft the paper. Include citations.",
    expected_output="A well-cited academic text in Markdown format.",
    agent=research_writer_agent
)

editing_task = Task(
    description="Rewrite the drafted paper heavily to bypass AI detection. Increase burstiness and perplexity. Do NOT lose or alter the citations.",
    expected_output="A highly humanized, cited academic text in Markdown format.",
    agent=editor_agent
)

# 3. Form the Crew and Execute
academic_crew = Crew(
    agents=[outliner_agent, research_writer_agent, editor_agent],
    tasks=[outline_task, drafting_task, editing_task],
    verbose=True
)

# Fire the pipeline!
final_markdown = academic_crew.kickoff()
print("\n--- FINAL DRAFT ---")
print(final_markdown)
```

## Step 7: Exporting to Professional Formats ($0)

Converting the output into a DOCX so students can view headings and automatic table of contents.

```python
from docx import Document

def save_to_word(markdown_text, filename="Final_Academic_Paper.docx"):
    doc = Document()
    doc.add_heading('Academic Research Draft', 0)
    
    # Simple parser (split paragraphs by double newline)
    paragraphs = str(markdown_text).split('\n\n')
    for para in paragraphs:
        # Check if the paragraph is a markdown heading
        if para.startswith('# '):
            doc.add_heading(para.replace('# ', ''), level=1)
        elif para.startswith('## '):
            doc.add_heading(para.replace('## ', ''), level=2)
        else:
            doc.add_paragraph(para)
            
    doc.save(filename)
    print(f"\nDocument successfully saved as {filename}! ($0 cost)")

save_to_word(final_markdown)
```

## Full Automated Workflow (How it runs in production):
1. User provides a topic.
2. System calls `search_academic_papers` -> Downloads abstracts from Semantic Scholar.
3. System calls `store_papers` -> Saves abstracts into local ChromaDB.
4. System polls `search_vector_db` to get relevant context.
5. CrewAI agents take the context -> Draft Outline -> Draft Paper -> Humanize Output.
6. System calls `save_to_word` and outputs the DOCX for the user to download.
