# Document Q&A Agent (RAG)

An agentic RAG app that answers questions from a PDF with page citations. It grades its own retrieval, rewrites weak queries, and says "I don't know" when the document does not contain the answer.

## How it works

Upload PDF -> chunk -> embed (all-MiniLM-L6-v2) -> ChromaDB -> retrieve top 8 chunks -> LLM grades relevance (YES/NO) -> if NO, rewrite the query and retry (max 2 times) -> answer with [Page X] citations, or "I don't know based on the provided document."

## Tech stack

Python, ChromaDB, sentence-transformers, Claude API, Gradio, Google Colab

## Run it

1. Open `Doc_Agent.ipynb` in Google Colab (use the "Open in Colab" badge).
2. Add `ANTHROPIC_API_KEY` to Colab Secrets and turn on notebook access.
3. Run all cells, then upload a text-based PDF in the Gradio app.

## Test document

Public IBM Redbook: "IBM z17 Technical Introduction" (SG24-8580-01, 104 pages), available from IBM Redbooks. The PDF is not included in this repo.

## Evaluation

Test set: 19 questions on the Redbook. 14 are answerable from the document and 5 are not (3 off-topic and 2 near-miss traps: an unreleased chip and product pricing).

| Metric | Baseline (k=4) | After change (k=8) |
|---|---|---|
| Answer accuracy | 9/14 (64%) | 10/14 (71%) |
| Correct page citations | 12/14 (86%) | 13/14 (93%) |
| Correct "I don't know" refusals | 5/5 (100%) | 5/5 (100%) |

Page numbers are PDF page numbers (cover = page 1). Full per-question results and notes: [`evaluation_results.xlsx`](evaluation_results.xlsx)

### What I changed
I made two changes together: raised the number of retrieved chunks from 4 to 8, and increased the chunk size from [old] to [new] characters. I then re-tested Q3 to Q7. This fixed one failure: the ME1 vs ME2 memory comparison now returns 64 TB vs 17.96 TB. Because both settings changed at once, I cannot say which one caused the improvement.

### What still fails and why
- **Spec-table questions (processor chip, clock speeds):** the values sit in a table on page 17. PDF tables split into chunks that lose their column labels, so the agent finds the numbers but cannot tell which belongs to which model.
- **Multi-part questions:** the agent sometimes answers only part of the question, for example listing one of the two AI accelerators (missed the Spyre Accelerator) or naming only one of the I/O features that use the DPU.
- More context and larger chunks did not fix these, which points to table structure and answer completeness rather than missing context.

### What worked
The relevance check and query rewrite correctly refused all 5 out-of-scope questions, including the near-miss traps.

## Limitations

- Scanned PDFs are not supported (no OCR)
- The index is stored in memory and resets when the app restarts
- Answers that depend on tables can be incomplete or mapped to the wrong model
- The test set is small (19 questions), so percentages are indicative, not statistically robust
- Grading was done manually by the author

## Next improvements

- Table-aware PDF extraction so column labels stay attached to values
- A final check that every part of a multi-part question is answered
- A larger test set, with chunk size and k changed one at a time
