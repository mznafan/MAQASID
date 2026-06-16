# MAQASID — Majmu'at Maqalat al-Islamiyyah al-Indunisiyyah

> **The Collection of Islamic Articles in Indonesian**  
> A closed-domain corpus for Indonesian Islamic NLP research
<!--
[![Paper](https://img.shields.io/badge/Paper-ETASR-blue)](https://doi.org/10.48084/etasr.XXXX)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

---
-->
## Overview

MAQASID is a closed-domain Indonesian Islamic corpus constructed from articles published by authoritative Indonesian Islamic organizations, including **Nahdlatul Ulama**, **Muhammadiyah**, **Bincangsyariah**, and others. It was developed to support the training and evaluation of word embeddings for Indonesian Islamic NLP tasks.

| Metric | Value |
|--------|-------|
| Total documents | 52,374 |
| Total sentences | 2,298,731 |
| Total words | 40,078,617 |
| Avg. sentences per document | 43.89 |
| Avg. words per document | 765.24 |
| Number of categories | 9 |

Documents are categorized into **9 Islamic subject headings** based on the classification scheme published by the National Library of the Republic of Indonesia (Perpustakaan Nasional RI).
<!--
---

## ⚠️ Copyright Notice

The full text of the MAQASID corpus **cannot be redistributed** publicly. All constituent articles remain under copyright of their respective publishing organizations. This repository therefore provides two reproducibility artefacts as an alternative:

1. **Metadata index** — URL + structured metadata for each article, enabling independent re-crawling
2. **Pre-trained embedding models** — derived artefacts that do not reproduce the source texts

---

## Repository Contents

```
MAQASID/
├── metadata/
│   └── maqasid_metadata.csv          # Article metadata index (see schema below)
├── embeddings/
│   ├── cbow_simple_prep.model        # Word2Vec CBOW — simple_prep
│   ├── cbow_simple_prep.model.wv.vectors.npy
│   ├── cbow_prep_no_stopword.model   # Word2Vec CBOW — prep_no_stopword
│   ├── cbow_prep_no_stopword.model.wv.vectors.npy
│   ├── cbow_prep_stemming.model      # Word2Vec CBOW — prep_stemming
│   ├── cbow_prep_stemming.model.wv.vectors.npy
│   ├── cbow_prep_stem_no_stopword.model   # Word2Vec CBOW — prep_stem_no_stopword
│   ├── cbow_prep_stem_no_stopword.model.wv.vectors.npy
│   ├── skipgram_simple_prep.model
│   ├── skipgram_simple_prep.model.wv.vectors.npy
│   ├── skipgram_prep_no_stopword.model
│   ├── skipgram_prep_no_stopword.model.wv.vectors.npy
│   ├── skipgram_prep_stemming.model
│   ├── skipgram_prep_stemming.model.wv.vectors.npy
│   ├── skipgram_prep_stem_no_stopword.model
│   ├── skipgram_prep_stem_no_stopword.model.wv.vectors.npy
│   ├── fasttext_simple_prep.model
│   ├── fasttext_simple_prep.model.wv.vectors.npy
│   ├── fasttext_prep_no_stopword.model
│   ├── fasttext_prep_no_stopword.model.wv.vectors.npy
│   ├── fasttext_prep_stemming.model
│   ├── fasttext_prep_stemming.model.wv.vectors.npy
│   ├── fasttext_prep_stem_no_stopword.model
│   └── fasttext_prep_stem_no_stopword.model.wv.vectors.npy
└── README.md
```

---

## Metadata Index

**File:** `metadata/maqasid_metadata.csv`

| Column | Description |
|--------|-------------|
| `url` | Original article URL on the source website |
| `title` | Article title |
| `date` | Publication date (YYYY-MM-DD) |
| `author` | Author name(s) |
| `category` | Original category label as assigned by the source portal/website |
| `norm_category` | Normalised category following the Islamic Subject Headings (*Tajuk Islam*) standard published by Perpustakaan Nasional RI; used as the ground-truth label in model training (9 classes) |
| `tags` | Article tags (pipe-separated) |

> **Note:** The metadata index does not include article body text. Researchers wishing to reconstruct the full corpus should retrieve text directly from the original URLs.

---

## Pre-trained Embedding Models

Twelve embedding models are provided, covering **3 model architectures × 4 preprocessing scenarios**, all trained using [Gensim](https://radimrehurek.com/gensim/).

### Model Architectures

| Model | Type | Description |
|-------|------|-------------|
| `cbow` | Word2Vec CBOW | Predicts target word from surrounding context |
| `skipgram` | Word2Vec Skip-Gram | Predicts surrounding context from a target word |
| `fasttext` | fastText | Extends Skip-Gram with character n-gram subword information |

### Preprocessing Scenarios

| Scenario Code | Steps Applied |
|---------------|--------------|
| `simple_prep` | Sentence tokenization → word tokenization → punctuation removal → non-alphabet removal → transliteration normalization |
| `prep_no_stopword` | `simple_prep` + stopword removal |
| `prep_stemming` | `simple_prep` + stemming |
| `prep_stem_no_stopword` | `simple_prep` + stopword removal + stemming |

Stopword removal and stemming were performed using the [PySastrawi](https://github.com/har07/PySastrawi) library, designed for Indonesian text.

### Training Hyperparameters

| Parameter | Value |
|-----------|-------|
| Embedding dimension | 300 |
| Window size | 5 |
| Min. word frequency | 5 |
| Training epochs | 5 |
| Negative samples | 5 |
| Hierarchical softmax | Disabled |

---

## Usage

### Load a pre-trained model

```python
from gensim.models import Word2Vec, FastText

# Load Word2Vec (CBOW or Skip-Gram)
model = Word2Vec.load("embeddings/skipgram_prep_stem_no_stopword.model")

# Load fastText
ft_model = FastText.load("embeddings/fasttext_simple_prep.model")

# Query: top-10 most similar words
model.wv.most_similar("shalat", topn=10)

# Word vector
vector = model.wv["zakat"]  # shape: (300,)
```

### Word analogy

```python
# "a is to a* as b is to b*"
result = model.wv.most_similar(
    positive=["jakarta", "presiden"],
    negative=["mekah"],
    topn=1
)
```

---

## Experimental Results

Best configurations from the benchmarking study:

| Model | Preprocessing | Macro F1 (Test) |
|-------|---------------|-----------------|
| Skip-Gram + CNN-static | `prep_stem_no_stopword` | **0.5746** |
| CBOW + CNN-static | `prep_stem_no_stopword` | 0.5569 |
| fastText + CNN-static | `prep_no_stopword` | 0.5341 |
| IndoBERT (fine-tuned) | `prep_stemming` | **0.6411** |

McNemar's test confirms that IndoBERT significantly outperforms the best non-contextual configuration (χ² = 7.26, *p* = 0.0071, α = 0.05).

Full results are reported in the accompanying paper.

---

## Citation

If you use MAQASID or the pre-trained models in your research, please cite:

```bibtex
@article{maqasid2025,
  title   = {Benchmarking {Word2Vec}, {FastText}, and {IndoBERT} on {MAQASID}:
             A Closed-Domain Religious Corpus},
  author  = {Nafan, Muhammad Zidan and others},
  journal = {Engineering, Technology \& Applied Science Research},
  year    = {2025},
  doi     = {10.48084/etasr.XXXX}
}
```

> Please update the DOI once the paper is formally published.

---
-->
## Contact

For questions about the dataset or models, contact the corresponding author:

**Muhammad Zidan Nafan**  
📧 [muhammadn@telkomuniversity.ac.id]  
🔗 [https://github.com/mznafan/MAQASID](https://github.com/mznafan/MAQASID)

---
<!--
## License

The metadata index and pre-trained embedding models in this repository are released under the [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) license.

The underlying article texts remain under copyright of their respective publishing organizations and are **not** included in this repository.
-->
