# DocInsight – Multimodal Document Question Answering System

## 📘 Overview

DocInsight is a research-oriented prototype that extracts, indexes, and queries information from educational PDFs using a **multimodal RAG (Retrieval-Augmented Generation)** pipeline. It integrates:

- Text extraction
- Table parsing
- Chart/flowchart image captioning
- Semantic embeddings
- A local offline LLM for answering questions

This system enables natural language querying of document content, grounded with retrieved evidence chunks, demonstrating a working approach to document intelligence using only local CPU resources.

---

## 🚀 Key Features

- **Local & offline** – No cloud APIs or paid model dependencies  
- **Text extraction** from PDF via PyMuPDF  
- **Table extraction** into structured DataFrames & textual chunks  
- **Page-level image captioning** using BLIP model  
- **Semantic embeddings** using SentenceTransformers  
- **FAISS vector search** for retrieving relevant chunks  
- **Local LLM QA** using TinyLlama  
- **Answers grounded in evidence**, with chunk metadata:
  - Source page  
  - Chunk type (`text` / `table` / `chart`)  
  - Similarity score / rank  

---

## 🧠 Project Methodology

The system performs:

1. **PDF ingestion**  
2. **Text block extraction**  
3. **Table detection and parsing**  
4. **Image captioning for charts/flowcharts**  
5. **Unified chunk construction** (text + tables + chart captions)  
6. **Vector embeddings for semantic search**  
7. **RAG answering using a local LLM**  
8. **Qualitative demonstration & conclusion**

This roughly mirrors ideas from modern document intelligence models such as DocVQA, LayoutLM, and ChartQA, but implemented as a lightweight local pipeline tailored to a single educational PDF.

---

## 📁 Repository Structure

```text
docinsight/
│
├── notebooks/
│   ├── 01_text_extraction.ipynb
│   ├── 02_embeddings_retrieval.ipynb
│   ├── 03_rag_qa.ipynb
│   ├── 04_table_extraction.ipynb
│   ├── 05_chart_extraction.ipynb
│   ├── 06_demo_and_conclusion.ipynb
│
├── data/
│   ├── raw/                      ← (not included in repo)
│   │    └── tables-charts.pdf    ← place here manually
│   ├── processed/
│        ├── tables-charts_chunks.csv
│        ├── tables-charts_master_chunks.csv
│        ├── demo_outputs.csv     ← (optional, from demo notebook)
│
├── README.md
├── .gitignore
├── requirements.txt              ← dependencies (optional)
```
## 🧪 Notebooks Explained

### `01_text_extraction.ipynb`
- Extracts text from `tables-charts.pdf` using PyMuPDF.  
- Groups low-level text blocks (with coordinates) into meaningful chunks.  
- Saves chunks to `data/processed/tables-charts_chunks.csv`.

---

### `02_embeddings_retrieval.ipynb`
- Loads text chunks from `tables-charts_chunks.csv`.  
- Builds semantic embeddings using `all-MiniLM-L6-v2`.  
- Uses FAISS for similarity search.  
- Prototypes `retrieve_similar_chunks(question)` for text-only retrieval.

---

### `03_rag_qa.ipynb`
- Loads the master chunk file (`tables-charts_master_chunks.csv`) once tables and charts are integrated.  
- Rebuilds embeddings & FAISS over all chunk types.  
- Loads a local TinyLlama model.  
- Implements:
  - `retrieve_similar_chunks(query)`  
  - `build_context_from_results(results)`  
  - `answer_question_rag(question)`  
- Allows interactive Q&A with **answers + evidence chunks**.

---

### `04_table_extraction.ipynb`
- Extracts tables from the PDF (via Camelot / pdfplumber).  
- Converts each table into a textual representation (DataFrame → string).  
- Wraps each as a chunk with `chunk_type="table"`.  
- Merges table chunks with text chunks into a unified `df_master`.  
- Saves updated `tables-charts_master_chunks.csv`.

---

### `05_chart_extraction.ipynb`
- Renders each PDF page to an image (`page0.png`, `page1.png`, …).  
- Uses the BLIP image captioning model to describe each page.  
- Treats each caption as `chunk_type="chart"` (page-level visual chunk).  
- Appends these to `df_master` and updates `tables-charts_master_chunks.csv`.

---

### `06_demo_and_conclusion.ipynb`
- Loads the final `tables-charts_master_chunks.csv` (text + tables + charts).  
- Builds embeddings and FAISS index over **all** chunks.  
- Uses TinyLlama with RAG to:
  - Answer curated questions  
  - Show the answer  
  - Show supporting evidence chunks with type, page, and similarity  
- Includes a small qualitative “evaluation” section and a **final conclusion summary**.

---

## 🔧 Installation & Setup

### 1. Clone the repo
git clone https://github.com/<yourusername>/docinsight.git
cd docinsight

### 2. Create a Python virtual environment
python -m venv .venv
.\.venv\Scripts\activate   # on Windows

# or on Linux/macOS:
source .venv/bin/activate

### 3. Install dependencies

# If you have a requirements.txt:
pip install -r requirements.txt

# If not, typical key dependencies include:
pip install pymupdf sentence-transformers faiss-cpu transformers pillow camelot-py[cv] pdfplumber

### 4. Place PDF in the proper location

# create the raw data folder (if not already)
mkdir -p data/raw

# Then copy your educational PDF into:
data/raw/tables-charts.pdf

# The project expects this filename and path by default.

------------------------------------------------------------

## ▶️ Running the System

# Recommended notebook execution order:
1. 01_text_extraction.ipynb
2. 04_table_extraction.ipynb
3. 05_chart_extraction.ipynb
4. 06_demo_and_conclusion.ipynb

# 01 → builds initial text chunks.
# 04 → adds table chunks and builds the master chunk file.
# 05 → adds chart/page-image caption chunks.
# 06 → runs the final demo and qualitative evaluation.

------------------------------------------------------------

### Example output (from notebook 06)

QUESTION:
What is the main function of tables, charts and graphs in written communication?

ANSWER:
They help summarize and highlight key information clearly so that readers can compare and understand data more easily.

EVIDENCE CHUNKS:
rank  chunk_type  chunk_id  page_number  similarity  text
1     text         13          2         0.89      "Tables, charts and graphs..."
2     text         14          2         0.83      "They help present information..."
...
## 📊 Evaluation Approach

This project focuses on qualitative, output-based evaluation rather than strict numeric accuracy:

✔ Does the system retrieve the correct region (table / paragraph / chart) of the document?  
✔ Does the answer roughly match the semantics of the document content?  
✔ Are explanations grounded with visible evidence chunks?  
✔ How does the system behave on different question types (text, table, chart, flowchart)?  

This interpretability-first view is especially useful when running small local models, where exact numerical accuracy is less reliable than conceptual understanding and retrieval quality.

------------------------------------------------------------

## 🧭 Future Improvements
- Implement rule-based / symbolic table querying to handle numeric questions deterministically  
  (e.g., "How many calories does a 6-year-old need?")  
- Improve chart handling via bounding-box detection and region-level captioning instead of whole-page rendering  
- Explore larger or quantized on-device language models for more accurate reasoning while remaining offline  
- Integrate layout-aware models such as LayoutLMv3 or Donut for richer document structure understanding  
- Evaluate on public benchmarks such as DocVQA, ChartQA, and InfographicsVQA for quantitative comparison  

------------------------------------------------------------

## 📌 Conclusion

DocInsight successfully demonstrates an end-to-end local multimodal document question answering system. By integrating text extraction, table parsing, and visual captioning into a unified embedding and retrieval pipeline, it provides a practical offline approach to document intelligence for educational PDFs. Despite limitations in precise numeric reasoning due to the use of a small local LLM, the project shows that meaningful, grounded QA over complex documents is achievable on CPU-only hardware.

This work forms a strong foundation for future research on hybrid symbolic–neural PDF reasoning, multimodal retrieval-augmented generation, and document understanding in academic and educational settings.

