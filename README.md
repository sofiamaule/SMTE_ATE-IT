# ATE-IT — Automatic Term Extraction (Italian)
 Top-ranked ensemble system for Subtask A (Term Extraction) of the ATE-IT shared task https://nicolacirillo.github.io/ate-it/

IT utilizes a fine-tuned Italian BERT token classifier and a spaCy NER module, integrated with a vocabulary-based filtering mechanism. The system achieved strong results, significantly outperforming the official baseline on the test set.
 
**Author:** Sofia Maule — University of Padua (2025–2026)

#### Subtask A — Term Extraction Pipeline
## Overview

This repository contains my experiments and final system for the **ATE-IT shared task — Subtask A: Automatic Term Extraction** on Italian texts.

The goal is to automatically extract domain-specific terms from Italian documents.

The final system combines:
- a fine-tuned BERT token classifier
- a trained spaCy pipeline
- a vocabulary built from the training gold terms
- post-processing + LLM-based reranking

Most of the work is implemented in Jupyter notebooks inside the `src/` folder.

## Method Overview

The final ensemble for Subtask A is composed of:

### 1) BERT term extraction
Token classification with labels `B-TERM` / `I-TERM` / `O`, fine-tuned on the ATE-IT training set (and train+dev in the final run).

**Implemented in:**
- `src/bert_term_extraction.ipynb`
- `src/final_train_dev_training/bert_term_extraction_train_dev.ipynb` (final training on train+dev)

### 2) spaCy trained extractor
Custom spaCy pipeline trained as a sequence labeller / NER-like term tagger.  
Uses POS, morphology, and context from `it_core_news_md` as features.

**Implemented in:**
- `src/spacy_term_extraction.ipynb`
- `src/final_train_dev_training/spacy_term_extraction_train_dev.ipynb`

### 3) Vocabulary from training set
A single vocabulary is built from the gold terms in the training data.

Used to:
- normalise and canonicalise terms
- filter very noisy candidates
- lightly favour terms that appear in the training vocabulary

### 4) Post-processing & cleaning
Includes normalisation (lowercasing, trimming punctuation, collapsing spaces), de-duplication at sentence level, removal of clearly truncated spans, and removal of nested terms.

**Implemented in:**
- `src/postprocessing/clean_terms.ipynb`
- `src/postprocessing/clean_final_submission.ipynb`

### 5) LLM-based reranking (optional; not used in the final system)
A lightweight LLM is queried to:
- remove obviously wrong or overly generic terms
- drop subspans when a better, longer term is present

**Implemented in:**
- `src/llm_reranking.ipynb`
- `src/llm_reranking_refined.ipynb`

### 6) Few-shot and zero-shot LLM-based variants
Additional LLM baselines/variants were tested.

**Implemented in:**
- `src/llm_few_shot_term_extraction.ipynb`
- `src/llm_zero_shot_term_extraction.ipynb`

Predictions are saved in `src/predictions/` (JSON format consistent with the ATE-IT official evaluation script).

## Baselines and Ablations

Several baselines and ablation models are also included (later not considered due to lower performance):

- Vanilla term extraction: `src/vanilla_term_extraction.ipynb`
- TF-IDF + POS heuristics: `src/tfidf_POS_heuristic_extraction.ipynb`
- spaCy pattern-based extractor (rule-heavy): `src/spacy_term_extraction_patterns.ipynb`

### Results

Evaluation outputs and logs are stored in:
- `evaluation/ablation_results.csv`
- `evaluation/subtask_a_evaluation_results.csv`
- `evaluation/subtask_a_evaluation.ipynb`

### Installation

Create a virtual environment:

```python -m venv . ```

**Activate it:**
-  Linux / macOS
```source .venv/bin/activate```
-  Windows
```.venv\Scripts\activate```

**Install dependencies**
```pip install -r requirements.txt```

**Download spaCy Italian model**
```python -m spacy download it_core_news_md```

**LLM API keys**
Create a .env file (or export environment variables) with your LLM credentials, for example:
```
GROQ_API_KEY=...
GEMINI_API_KEY=...
```


### How to Reproduce the Final System (Subtask A)

All steps are done via notebooks (no command-line scripts yet).

1. **Train / load BERT model**  
   Open `src/bert_term_extraction_extended.ipynb`.

   - Run the notebook to:
     - load the ATE-IT training data
     - fine-tune BERT
     - generate BERT predictions for dev
   - Save outputs into `src/predictions/`

2. **Train spaCy pipeline**  
   Open `src/spacy_term_extraction.ipynb`.

   - Train the spaCy model on the same training data
   - Export spaCy predictions for dev sentences → `src/predictions/`

3. **Build training vocabulary + ensemble (BERT + spaCy)**  
   Open `src/bert_spacy_term_extraction.ipynb`.

   - Build the vocabulary from gold terms in `data/subtask_a_train.*`
   - Merge BERT and spaCy candidates using this vocabulary
   - Save the ensemble predictions to `src/predictions/`

4. **Post-processing / cleaning**  
   Open `src/postprocessing/clean_bert_terms.ipynb`.

   - Run the cleaning pipeline to:
     - normalise text
     - drop truncated spans
     - de-duplicate terms
     - remove nested terms
   - Save cleaned predictions (e.g., `src/predictions/subtask_a_dev_ensemble_clean.json`)

5. **Final submission formatting**  
   Open `src/postprocessing/clean_final_submission.ipynb`.

6. *(Optional)* **LLM reranking**  
   Open `src/llm_reranking.ipynb`.

   - Configure the API key via `.env`
   - Run the notebook to obtain the LLM-reranked predictions
   - Save predictions to `src/predictions/`

7. **Evaluation**  
   Open `evaluation/subtask_a_evaluation.ipynb`.

   - Set the path to the chosen prediction file in `src/predictions/`
   - Run all cells to compute:
     - Micro Precision / Recall / F1
     - Type Precision / Recall / F1

Final scores for the different variants are logged in:
- `evaluation/subtask_a_evaluation_results.csv`
- `evaluation/ablation_results.csv`




#### License
This project is for academic/research purposes.
You can adapt or reuse parts of the code and notebooks; please cite the ATE-IT shared task if you use the dataset.

#### Contact
Sofia Maule
Master’s student in Computer Engineering – University of Padua
