
# Document Q&A Agent (RAG)

An agentic RAG app that answers questions from a PDF with page citations. It grades its own retrieval, rewrites weak queries, and says "I don't know" when the document doesn't contain the answer.

## How it works

Upload PDF -> chunk (800 chars, 100 overlap) -> embed (all-MiniLM-L6-v2) -> ChromaDB -> retrieve top 4 chunks -> LLM grades relevance (YES/NO) -> if NO, rewrite the query and retry (max 2 times) -> answer with [Page X] citations, or "I don't know based on the provided document."

## Tech stack

Python, ChromaDB, sentence-transformers, Claude API, Gradio, Google Colab

## Run it

1. Open `Doc_Agent.ipynb` in Google Colab (use the "Open in Colab" badge in the notebook).
2. Add `ANTHROPIC_API_KEY` to Colab Secrets and turn on notebook access.
3. Run all cells, then upload a text-based PDF in the Gradio app.

## Test document

Public IBM Redbook: "IBM z17 Technical Introduction" (SG24-8580-01, 104 pages), available from IBM Redbooks. The PDF is not included in this repo.

## Evaluation

Test set: 20 questions (15 answerable from the document, 5 not in the document).

Results: in progress. Metrics will be added here: answer accuracy, correct page citations, and correct "I don't know" refusals.

## Limitations

- Scanned PDFs are not supported (no OCR)
- The index is stored in memory and resets when the app restarts
- Answers that depend on tables can be incomplete
- Citations use PDF page numbers, not the book's printed page numbers
