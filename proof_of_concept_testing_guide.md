# Proof of Concept: Testing Your Academic RAG Pipeline

Before investing weeks into building a full software application (backend, frontend, vector DBs), you must validate that the "Brain" of your application works exactly as expected. We will do this using a rapid prototype in a **Jupyter Notebook**.

## Phase 1: Environment Setup

Start by isolating your test environment and getting the necessary tools. 

```bash
mkdir rag_poc_test
cd rag_poc_test
python -m venv venv
# On Linux/Mac:
source venv/bin/activate  
# On Windows: 
# venv\Scripts\activate

# Install Jupyter and the minimal required libraries for testing
pip install jupyter requests langchain-groq
```

Set your API key exactly where you start the notebook:
```bash
export GROQ_API_KEY='your_free_groq_api_key_here'
jupyter notebook
```
Create a new Notebook named `rag_api_and_prompt_test.ipynb`.

---

## Phase 2: Testing the Free Academic API (Cell 1)

**Goal:** Ensure Semantic Scholar actually returns high-quality abstracts and metadata for your chosen niche. If the data retrieved here is bad, your AI's writing will be bad.

**Paste and run this in Cell 1:**
```python
import requests
import json

def fetch_sample_papers(query="IoT in agriculture", limit=3):
    """Hits the Semantic Scholar API to retrieve real papers."""
    url = f"https://api.semanticscholar.org/graph/v1/paper/search?query={query}&limit={limit}&fields=title,authors,year,abstract"
    response = requests.get(url)
    
    if response.status_code != 200:
        print(f"API Failed! Status: {response.status_code}")
        return None
        
    papers = response.json().get('data', [])
    return papers

# Test the function with a specific query
papers = fetch_sample_papers("IoT in smart agriculture architecture")

print(f"Found {len(papers)} papers.\n")
for i, p in enumerate(papers):
    print(f"--- Paper {i+1} ---")
    print(f"Title: {p.get('title')}")
    print(f"Year: {p.get('year')}")
    print(f"Abstract Snippet: {str(p.get('abstract'))[:200]}...\n")
```

**What to verify:** 
- Did it return actual, relevant papers?
- Are the abstracts present? *(Note: Sometimes abstracts are null if the paper isn't open access; if this happens often in your niche, you might need to adjust search parameters or test the OpenAlex API).*

---

## Phase 3: Testing the LLM & Citation Accuracy (Cell 2)

**Goal:** Provide the LLM with the raw text from Cell 1 and force it to write a paragraph citing ONLY that text. We must prove the LLM won't hallucinate fake citations before we trust it to automate full documents.

**Paste and run this in Cell 2:**
```python
import os
from langchain_groq import ChatGroq
from langchain_core.prompts import PromptTemplate

# Initialize the Groq LLM (Llama 3 70B runs blazingly fast here)
llm = ChatGroq(
    api_key=os.environ.get("GROQ_API_KEY"),
    model="llama3-70b-8192"
)

# Convert our retrieved papers into a context string (simulating what the Vector DB will do later)
context_string = ""
for p in papers:
    if p.get('abstract'):
        context_string += f"[Source: {p.get('title')}, Year: {p.get('year')}]\nAbstract: {p.get('abstract')}\n\n"

# The Strict "Writer Agent" Prompt
prompt_template = """
You are an expert academic writer. Your task is to write a short 'State of the Art' paragraph regarding the topic.
CRITICAL RULES:
1. You MUST ONLY use facts found in the CONTEXT below. Do not use outside knowledge.
2. If the context does not contain enough information, state "Insufficient information retrieved."
3. You MUST cite your sources strictly using the format: [Title, Year] inline within your text. Do not use footnotes.

CONTEXT:
{context}

Drafted Paragraph:
"""

prompt = PromptTemplate.from_template(prompt_template)
chain = prompt | llm

# Fire the test!
print("Generating academic paragraph based strictly on retrieved facts...\n")
result = chain.invoke({"context": context_string})
print(result.content)
```

**What to verify (The crucial moment of truth):**
1. Read the generated text carefully. 
2. **Check the citations:** Did it insert `[Title, Year]` directly beside the claims?
3. **Check for Hallucinations:** Did it mention any concepts or facts that *were not* present in the printed abstracts from Cell 1? If it hallucinated, your prompt needs to be stricter before moving to software dev.

---

## Phase 4: Final Evaluation Before Building

**Pass Criteria to proceed to software development:**
- [ ] The API successfully pulls 3-5 relevant abstracts for your target topics.
- [ ] The LLM successfully formats inline citations dynamically.
- [ ] The LLM did not hallucinate extra facts outside of the provided abstracts.

If all three boxes are checked, your core technology is validated. You can confidently close the notebook and begin building the multi-agent infrastructure outlined in the `zero_cost_implementation_guide.md`.
