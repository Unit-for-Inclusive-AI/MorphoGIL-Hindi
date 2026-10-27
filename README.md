# MorphoGIL-Hindi

Resources for **gender-inclusive language generation in Hindi**: two Hindi datasets, a Hindi gender-bias Knowledge Graph (KG), and the code for a KG-guided generation pipeline.

This repository accompanies the paper *"Morphology-Enhanced KG-Guided Gender-Inclusive Language Generation for Hindi"* .

---

## Contents

| Resource | Size | Description |
|---|---|---|
| **GNP-Hindi** | 296 pairs | Gendered Hindi term → gender-neutral alternative |
| **CSP-Hindi** | 348 pairs | Biased Hindi sentence → inclusive rewrite (60 predicate rules, 7 bias categories) |
| **Hindi Gender-Bias KG** | 924 nodes, 2,077 edges | 9 entity types and 9 relation types built from GNP-Hindi and CSP-Hindi |

```
MorphoGIL-Hindi/
├── datasets/
│   ├── [GNP-Hindi.csv]            # 296 gendered-to-neutral term pairs
│   └── [CSP-Hindi.csv]            # 348 biased-to-inclusive sentence pairs
├── outputs/
│   ├── hindi_gender_kg.graphml           # the Knowledge Graph
│   ├── hindi_gender_kg_triplets.json     # the KG as (subject, relation, object) triplets
│   ├── kg_schema.json                    # entity and relation type definitions
│   ├── kg_guided_outputs.csv             # system outputs + Gemma judge scores, 17 test prompts
│   ├── human_evaluation_scores.csv       # scores from the two human evaluators
│   └── judge_prompts.md                  # prompts used by the Gemma-2-2b-it judge
├── Phase 1 - Datasets.ipynb
├── Phase 2 - Constructing KG.ipynb
├── Phase 3 - Neurosymbolic counterfactual Pipeline.ipynb
├── Phase 4 - Evaluation.ipynb
├── requirements.txt
└── LICENSE
```

---

## 1. GNP-Hindi (Gender-Neutral Pairs)

**Columns:** [column names, e.g. `original_term`, `neutral_term`, `category`]

**How it was built**
1. The 693 English gender-neutral pairs of Bartl and Leavy (2024) were translated into Hindi with ChatGPT-4.
2. 103 compound forms with no Hindi equivalent (e.g. *boyish*, *girldom*) were excluded, leaving 590.
3. Every translation was checked manually against the LBSNAA guide on gender-inclusive communication, the gender-neutral occupation names of Kirtane and Anand (2022), Joshi et al. (2024) and the Shabdkosh dictionary.
4. We removed duplicates created by translation, pairs whose Hindi translation is already gender-neutral, and pairs with no neutral alternative supported by the sources. Four exact duplicate rows were then removed, giving **296 unique pairs**.

**Categories:** Occupational (OCC), Representational (REP), Relational (REL). In the KG, categories are assigned by keyword matching; terms that match no keyword are labelled Unknown.

**Examples**

| Category | Gendered term | Neutral term |
|---|---|---|
| Occupational (OCC) | पुलिसवाला *pulisvālā* (policeman) | पुलिस अधिकारी *pulis adhikārī* (police officer) |
| | नौकरानी *naukarānī* (maidservant) | घरेलू कर्मचारी *gharelū karmachārī* (domestic worker) |
| Representational (REP) | जनशक्ति *janśakti* (manpower) | कार्यबल *kāryabal* (workforce) |
| | सामान्य आदमी *sāmānya ādmī* (common man) | सामान्य व्यक्ति *sāmānya vyakti* (common person) |
| Relational (REL) | प्रेमी/प्रेमिका *premī/premikā* (boyfriend/girlfriend) | साथी *sāthī* (partner) |
| | पति/पत्नी *pati/patnī* (husband/wife) | जीवनसाथी *jīvansāthī* (life partner) |

---

## 2. CSP-Hindi (Counterfactual Sentence Pairs)

**Columns:** [column names, e.g. `pair_id`, `biased_sentence`, `counterfactual_sentence`]

**How it was built**
- 60 predicate rules, each written as a triplet (masculine form, feminine form, inclusive form). The inclusive form uses the masculine plural, the Hindi convention for mixed or generic reference.
- Six gendered subjects: पुरुष, आदमी, लड़के (masculine) and महिलाएँ, औरतें, लड़कियाँ (feminine). All map to the neutral subject सभी लोग (*sabhī log*, all people).
- The four adult subjects received all 60 rules (240 pairs). The two child subjects received 54 rules each (108 pairs), excluding six rules about marriage and spousal obedience. Total: **348 pairs**.

| Bias category | Rules |
|---|---|
| Intelligence & Capability | 12 |
| Social Role & Domestic | 10 |
| Rights & Dignity | 10 |
| Emotion & Temperament | 8 |
| Physical & Sports | 8 |
| Behaviour & Character | 7 |
| Hindi Cultural | 5 |
| **Total** | **60** |

---

## 3. Hindi Gender-Bias Knowledge Graph

Built with NetworkX from GNP-Hindi and CSP-Hindi. Each node is a unique string, so node counts can be lower than dataset sizes (e.g. a gendered term with two neutral alternatives is one node).

| Entity type | Count | Purpose |
|---|---|---|
| BiasedSentence | 348 | A gender-biased CSP sentence |
| CounterClaim | 7 | A reusable inclusive statement for a stereotype dimension |
| CounterfactualRule | 59 | A unique inclusive rewrite in CSP-Hindi (two rules share one sentence) |
| GenderedSubject | 6 | A masculine or feminine subject form |
| GenderedTerm | 294 | An original term from GNP-Hindi |
| GrammarRule | 26 | An observed Hindi gender- or number-marked agreement form |
| NeutralTerm | 173 | A gender-neutral alternative from GNP-Hindi |
| StereotypeCategory | 4 | GNP categories OCC, REL, REP and UNKNOWN |
| StereotypeDimension | 7 | Profession, Family Roles, Leadership/Traits, Emotion, Intelligence/Learning, Behavior, General |

| Relation type | Count | Purpose |
|---|---|---|
| BELONGS_TO_CATEGORY | 295 | Gendered term → OCC, REL, REP or UNKNOWN |
| COUNTERED_BY | 7 | Stereotype dimension → inclusive counterclaim |
| COUNTERFACTUAL_OF | 348 | Biased sentence → its inclusive rewrite |
| HAS_GRAMMATICAL_GENDER | 27 | Clearly marked term → lexical gender label |
| HAS_STEREOTYPE | 354 | Biased sentence → one or more stereotype dimensions |
| HAS_SUBJECT | 348 | Biased sentence → its gendered subject |
| NEUTRAL_REPLACEMENT | 296 | Gendered term → neutral alternative |
| RELEVANT_TO_DIMENSION | 96 | Occupational GNP term → Profession dimension |
| TRIGGERS_MUTATION | 306 | Biased sentence → observed agreement form |

Stereotype dimensions are assigned by keyword matching and are separate from the seven CSP bias categories. A sentence can match more than one dimension.

---

## Quick start

```bash
pip install -r requirements.txt
```

**Load the datasets**
```python
import pandas as pd
gnp = pd.read_csv("datasets/[GNP-Hindi.csv]")
csp = pd.read_csv("datasets/[CSP-Hindi.csv]")
print(len(gnp), len(csp))   
```

**Look up a neutral replacement in the KG**
```python
import networkx as nx
G = nx.read_graphml("outputs/hindi_gender_kg.graphml")

term = "पुलिसवाला"
for _, target, data in G.out_edges(f"gendered:{term}", data=True):
    if data["relation"] == "NEUTRAL_REPLACEMENT":
        print(term, "→", G.nodes[target]["label"])
# पुलिसवाला → पुलिस अधिकारी
```

**Follow a stereotype to its counterclaim**
```python
sentence = next(n for n, a in G.nodes(data=True) if a["node_type"] == "BiasedSentence")
print(G.nodes[sentence]["label"])
for _, dim, d in G.out_edges(sentence, data=True):
    if d["relation"] == "HAS_STEREOTYPE":
        for _, claim, d2 in G.out_edges(dim, data=True):
            if d2["relation"] == "COUNTERED_BY":
                print(G.nodes[dim]["label"], "→", G.nodes[claim]["label"])
```

**Run the full pipeline:** open the notebooks in Google Colab in order (Phase 1 → 4). Phases 3 and 4 need a GPU (tested on a T4) and a Hugging Face token with access to `mistralai/Mistral-7B-Instruct-v0.3` and `google/gemma-2-2b-it`.

---

## Evaluation outputs

- `kg_guided_outputs.csv`: output of the KG-guided system for the 17 unseen test prompts (adapted from HinStereo-100), with retrieval type, method and Gemma-2-2b-it scores.
- `human_evaluation_scores.csv`: GA, GN and QR scores (0–2) from two native Hindi-speaking evaluators, who volunteered and scored the outputs independently without seeing the Gemma scores. No personal data is included.
- `judge_prompts.md`: the GA, GN and QR prompts given to the Gemma judge.

Scores are normalised as score % = mean × 50.

---

## Limitations

The resources cover binary gender only. GNP-Hindi translations were drafted with ChatGPT-4 and checked manually, so some errors may remain. Categories and stereotype dimensions are assigned by keyword matching. See the paper for full details.

---

## License
Datasets and KG: [CC BY 4.0]
Code: [MIT] (see LICENSE)
```

