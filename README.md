# QA Benchmark: Ranking Experimental Results from Research Papers

This repository provides a **QA benchmark** for evaluating systems that answer **ranking-style questions** using **experimental results reported in research papers** (typically from tables inside PDFs).

The benchmark is stored in an Excel file **`qa_annotated.xlsx`**, in the worksheet **`qa_pair`**. Each row corresponds to one QA item.

---

## 1) Task Overview

Given:
- a **Question** (ranking query), and
- one paper (**single-doc**) or multiple papers (**multi-doc**),

the system should output the **Answer** grounded in the paper(s)’ reported evaluation results.

This benchmark focuses on ranking queries such as:
- best method variant (within one paper),
- best dataset,
- best method or top-*k* methods (across methods and/or papers).

---

## 2) Data File

- **File**: `qa_annotated.xlsx`
- **Sheet name**: `qa_pair`

Each row in `qa_pair` is a QA example.

---

## 3) Schema (Columns in `qa_pair`)

| Column | Description |
|---|---|
| `Topic` | `research topic` |
| `Query type` | `single-doc` or `multi-doc` |
| `Query plan type` | `Rank method variants`, `Rank datasets`, or `Rank methods` |
| `Paper` | Paper identifier (recommended to map to a PDF filename, e.g., `chase-sql` → `chase-sql.pdf`) |
| `Question` | Natural-language question |
| `Answer` | Manually annotated ground-truth answer |


---

## 4) How to Use the Benchmark

### 4.1 Load the QA pairs (Python)
```python
import pandas as pd

xlsx_path = "qa_annotated.xlsx"
df = pd.read_excel(xlsx_path, sheet_name="qa_pair")

```

### 4.2 Resolve `Paper` → PDF path(s)
A recommended convention is to store PDFs under `data/{topic}/` and resolve like:
- `data/{topic}/{Paper}.pdf`

```python
from pathlib import Path

PAPER_DIR = Path(f"data/{topic}")

def resolve_pdf(paper_key: str) -> Path:
    return PAPER_DIR / f"{paper_key}.pdf"

```

### 4.3 Multi-doc items (paper lists)
For multi-doc items, you should define how the `Paper` field encodes a list of papers. Common options:

- **delimiter-separated string**
  - Example: `paperA,paperB,paperC`
  - Parse by splitting on `,`

```python
def resolve_multi_pdf(paper_field: str):
    keys = [k.strip() for k in paper_field.split(",") if k.strip()]
    return [resolve_pdf(k) for k in keys]
```


### 4.4 Run evaluation
For each QA item:
1. Load the referenced PDF(s).
2. Provide the `Question` to your QA system.
3. Compare the predicted answer with the annotated `Answer`.

---

## 5) Query Templates (Preliminaries)

This benchmark is based on ranking query templates such as:

### Rank datasets
- `Which dataset has the best performance over the [metric]?`

### Rank method variants
- `Which method variant has the best performance over the [metric]?`

### Rank methods
- `Tell me the top-[k] methods over the [metric].`
- `Which method is the best over the [metric]?`
- `Which method is the worst over the [metric]?`

---

## 6) Benchmark Construction Process

The benchmark was constructed using the following pipeline:

1. **Pick well-studied topics**  
   Select topics with multiple well-known benchmarks and commonly used metrics.

2. **Collect papers**  
   For each topic, collect papers that report experimental results for the selected benchmark(s).  
   (If possible, ~10 papers per topic to support multi-doc comparisons.)

3. **Extract evaluation results**  
   Extract experimental results from each paper, based on their tables.

4. **Construct QA pairs + manual annotation**
   - Randomly choose a single paper or a paper list.
   - Construct questions using the templates (LLM-assisted).
   - **Manually annotate** the answer for each question to ensure correctness.

---

