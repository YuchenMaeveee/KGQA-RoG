# Can an LLM Follow the Evidence in a Graph?

**Author:** Yuchen Ma  
**Institution:** National University of Singapore  

## 1. Overview
This repository contains the implementation of a proof-of-concept Knowledge Graph Question Answering (KGQA) system. It compares a simple sub-graph neighborhood baseline (Method A) with a structured, "Plan-then-Retrieve" method inspired by the Reasoning on Graphs (RoG) framework (Method B). The evaluation is conducted on an 8-question subset of the RoG-WebQSP dataset.

## 2. Deliverables
* `code.ipynb`: The main Jupyter Notebook containing the full pipeline (data loading, linking, reasoning methods, evaluation, and edge-removal test).
* `results.jsonl`: The required output file containing the per-question results (linked_entities, answer, evidence_paths, status) for both methods.
* `report.pdf`: A 2-page report detailing the quantitative results, failure analysis, edge-removal test, and a brief paper note on RoG.
* `README.md`: This file.

## 3. Setup and Run Instructions
### Prerequisites
Ensure you have Python 3.8+ installed. You need the following libraries:
` ` `bash
pip install datasets google-genai
` ` `

### Run Instructions
1. Open the `code.ipynb` file in Jupyter Notebook or JupyterLab.
2. Locate the API initialization cell:
   ` ` `python
   API_KEY = "YOUR_GEMINI_API_KEY" 
   client = genai.Client(api_key=API_KEY)
   ` ` `
   *Note: Ensure your API key is valid. The code is currently configured to use `gemini-3.1-flash-lite-preview`.*
3. Execute the notebook cells sequentially from top to bottom.
4. The notebook will automatically download the dataset from Hugging Face, run the entity linker, execute Method A and Method B, perform the evaluation, and conclude with the Edge-Removal Test.

## 4. Model and AI Tools Used
* **Large Language Model (LLM):** `gemini-3.1-flash-lite-preview` (accessed via the `google-genai` SDK) with `temperature=0.0` for deterministic outputs.
* **AI Coding Assistants:** AI conversational agents (Gemini/ChatGPT) were utilized as pair-programming assistants to brainstorm graph traversal algorithms (BFS), debug Python `ValueError` / `AttributeError` exceptions during JSON parsing, and polish the grammatical structure of the final academic report. All logical implementations and architectural choices were manually verified and aligned with the RoG framework.

## 5. Implementation Details

### Part A: Entity Linker
The entity linker extracts candidate name phrases from the natural language question and ranks nodes from the question's specific graph. It utilizes **exact string matching** followed by **fuzzy string matching** (token overlap) to score candidates. It successfully returns the Top-3 candidates, achieving a Hit@1 and Hit@3 of 87.5% on the test set.

### Part B: Reasoning Methods
* **Method A (Neighborhood Baseline):** 
  Selects the Top-1 linked entity and extracts a 2-hop neighborhood (capped at 50 triples). The raw sub-graph is formatted into a string and fed directly into the LLM alongside the question to request an answer and cited evidence.
* **Method B (RoG-Inspired Method):** 
  Implements a rigorous 4-step pipeline:
  1. **Planning:** The LLM generates up to two semantic relation sequences (paths) based on the question.
  2. **Retrieval:** A constrained Breadth-First Search (BFS) is executed starting from the Top-3 linked entities, strictly following the planned relations.
  3. **Grounding:** Only exact, physically existing graph paths are retained.
  4. **Answering:** The surviving paths are presented to the LLM to deduce the final answer. If no paths survive, it triggers a deterministic `"abstain"`.

### Evaluation
Both methods are evaluated on the exact same 8 test questions using the identical LLM and decoding settings. Metrics reported include Link Hit@1, Link Hit@3, generated answers, strictly supported answers, and the average number of triples shown to the LLM. 

### Part 6: Edge-Removal Test
To evaluate "Faithful Reasoning", we identified a multi-hop question that Method B answered correctly (*"who was the president after jfk died"*). We manually removed the bridge triple `['John F. Kennedy', 'government.us_president.vice_president', 'Lyndon B. Johnson']` from the graph. 
* Method B successfully registered the broken graph topology, failed the grounding step, and correctly fell back to an `"abstain"` status, proving it resists relying on parametric memory (hallucination) when physical evidence is absent.
