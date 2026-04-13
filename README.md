# MIDI Melodic Similarity

Graph-based melodic similarity system evaluated on songs from the Lakh MIDI Dataset. Encodes melodies as DAGs with sequential and hierarchical edges, evaluates 11 similarity metrics against 32 labeled pairs across parodies, covers, samples, and copyright lawsuits. CS 6795 final project, Georgia Tech.

---

## Overview

This project investigates whether graph-based melodic representations outperform feature vector representations for detecting musical similarity. Each song is encoded two ways:

- **Vector**: 12-dimensional statistical summary (mean, std, min, max of interval, contour, and weight distributions), compared via cosine similarity
- **Graph**: Directed acyclic graph (DAG) where nodes are notes and edges capture sequential and hierarchical melodic relationships, compared via 11 distinct similarity metrics

The evaluation uses a hand-labeled test suite of 32 song pairs across five categories (parodies, covers, samples, interpolations, lawsuits) plus 50 randomly sampled true negatives.

---

## Repository Structure

```
├── database_uploader.py          # Core similarity functions (all 11 methods)
├── song_analyzer.py       # Full analysis cell — figures, significance tests, ablation
├── midi_project.db        # SQLite database (not included — too large - over 35 gigabytes)
└── README.md
```

---

## Database Schema

The SQLite database (`midi_project.db`) contains four tables:

| Table | Key Columns |
|-------|-------------|
| `songs` | `id`, `artist`, `title`, `length_seconds` |
| `vector_features` | `song_id`, `features` (JSON) |
| `graph_data` | `song_id`, `nodes` (JSON), `edges` (JSON) |
| `similarities` | `song_id`, `song2_id`, `vector_sim`, `graph_sim` |

Each note in `vector_features` is stored as `{note, beat, interval, contour, weight}`. Each edge in `graph_data` is stored as `{from, to, type, interval, contour}` where `type` is `sequential` or `hierarchical`.

---

## Similarity Methods

| Method | Description |
|--------|-------------|
| Vector cosine | Cosine similarity on 12-dim statistical summary |
| Graph (original) | Jaccard over edge fingerprint multisets |
| Hierarchical | Jaccard restricted to hierarchical edges only |
| Normalized | Size-normalized Jaccard |
| 2-gram / 3-gram | Jaccard over consecutive edge fingerprint n-grams |
| Contour | Jaccard over contour-direction-only fingerprints |
| LCS | Longest common subsequence on contour sequences |
| Sliding window | Fraction of contour windows from shorter song appearing in longer |
| Interval cosine | Cosine on 11-dim interval histogram |
| Chroma | Cosine on 12-dim pitch class profile |
| Ensemble | 0.40 × hierarchical + 0.40 × contour + 0.20 × chroma |

---

## Usage

All code is designed to run in Google Colab with the database mounted from Google Drive. Set `DB_PATH` to your database location and run cells in order.

**Query a single song against the full database:**
```python
query_song_vs_db(song_id=7301, top_n=20)  # Amish Paradise
```

**Check a specific pair across all 11 methods:**
```python
check_pair(7301, 5776)  # Amish Paradise vs Gangsta's Paradise
```

**Run the full analysis (figures + significance tests + ablation):**
```python
# Run analysis_cell.py as a Colab cell after loading similarity functions
```

---

## Key Results

| Method | Parody Mean | Negative Mean | Gap |
|--------|-------------|---------------|-----|
| Contour | 0.877 | 0.420 | +0.457 |
| Hierarchical | 0.767 | 0.330 | +0.437 |
| Graph (orig) | 0.749 | 0.350 | +0.399 |
| Interval Cosine | 0.902 | 0.880 | +0.022 |
| Vector | 0.993 | 0.991 | +0.002 |

Contour-based similarity achieves the best discrimination between melodically related songs (parodies) and unrelated songs (random negatives), consistent with Dowling's (1978) cognitive model that melodic contour is the primary cue for melody recognition.

---

## Dataset

[Lakh MIDI Dataset](https://colinraffel.com/projects/lmd/) — 176,581 MIDI files. 

The database is not included in this repo due to size. To reproduce, download the Lakh MIDI Dataset and run the ingestion pipeline to populate `midi_project.db`.

---

## Dependencies

```
numpy
scipy
sklearn
matplotlib
sqlite3 (stdlib)
```

---

## References

- Raffel, C. (2016). Learning-based methods for comparing sequences. PhD dissertation, Columbia University.
- Dowling, W. J. (1978). Scale and contour: Two components of a theory of memory for melodies. *Psychological Review*, 85(4), 341–354.
- Gentner, D. (1983). Structure-mapping: A theoretical framework for analogy. *Cognitive Science*, 7(2), 155–170.
