# Small-Catalog Recommendation Explorer

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![Platform](https://img.shields.io/badge/Run%20on-Google%20Colab-orange)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Cost](https://img.shields.io/badge/Cost-Free-success)

Comparing three simple recommenders on a small book catalog, with a time-based train/test split, cold-start handling, explanations and acceptance tests.

## Table of Contents
1. [Problem](#problem)
2. [What I Built](#what-i-built)
3. [Results](#results)
4. [Sample Output](#sample-output)
5. [Acceptance Tests](#acceptance-tests)
6. [Tech Stack](#tech-stack)
7. [How to Run](#how-to-run)
8. [Limitations](#limitations)
9. [About Me](#about-me)

## Problem
A small study club has about 60 books and wants to recommend books to its members. Some members are brand new (no reading history) and some books are brand new (nobody has read them yet). Offline numbers also do not prove that people are happier, so the goal here is to compare simple methods honestly instead of picking the one that looks good.

## What I Built
- A fictional catalog of 60 books across 7 genres, with 150 users and 926 reads (fixed seed 42, no download needed)
- A **time-based split**: the oldest 80% of reads for training, the newest 20% for testing (cut day 156)
- Three recommenders:
  1. **Popularity**: most-read books in the training data
  2. **Content**: TF-IDF of title, genre and words, user profile = average of the books they read
  3. **Item-item collaborative filtering (CF)**: books read by the same people look similar. New books with zero reads fall back to the content score
- Metrics: **recall@5**, **catalog coverage** and a simple **diversity** score (distinct genres in the top 5 / 5)
- Cold-start handling for new users (popularity fallback) and a separate "new arrivals" shelf for new books
- Explanations for each suggestion, and a "clear my history" check
- Acceptance tests for leakage, repeated items and cold start

## Results
Evaluated on 88 users who have training history and at least one new read in the test period (train rows: 736, test rows: 190).

| Method | recall@5 | Coverage | Diversity |
|---|---|---|---|
| Popularity | 0.330 | 0.183 | 0.907 |
| Content | 0.333 | 0.983 | 0.343 |
| Item-item CF | **0.520** | 0.733 | 0.723 |

What the table says:
- Item-item CF has the best recall@5 (0.52 vs 0.33 for the other two).
- Popularity and Content are almost tied on recall, but behave very differently: popularity shows only 18% of the catalog, content shows 98% of it but repeats the same genre a lot (diversity 0.343).
- CF is the best balance of the three here.

**Cold-start users (4 users):** popularity fallback gave recall@5 of 0.458. This is only 4 users, so I treat it as a check that the fallback works, not as a result to quote.

**New books:** the CF fallback showed 0 new books to warm users, so new books get their own "new arrivals" shelf using the content score instead.

## Sample Output
```
cut day: 156.0 | train rows: 736 | test rows: 190
users with relevant test items: 92
cold-start users (no train history): 4

        method  recall@5  coverage  diversity
0   Popularity     0.330     0.183      0.907
1      Content     0.333     0.983      0.343
2  ItemItem CF     0.520     0.733      0.723

new arrivals shelf for user 5 : ['Summer Letters 60', 'Dragon Sword 57', 'Quantum Cyber 58']

- Promise Summer 53 | Romance | because you read 'Love Wedding 4' (shared words: romance, summer, tale, wedding)
- Magic Prophecy 50 | Fantasy | because you read 'Wizard Elf 29' (shared words: elf, fantasy, wizard)

after clearing history: same as popularity list: True
```

## Acceptance Tests
All of these pass in the run:

| Test | Result |
|---|---|
| Books a user already read never come back (all methods, including fallback) | 0 repeats, PASS |
| Cold-start users get the popularity baseline | PASS |
| Last training day (155) is before the first test day (156) | PASS |
| Interaction matrix is built from training rows only | PASS |
| Test-period reads inside the training matrix | 0, PASS |

## Tech Stack
| Area | Tools |
|---|---|
| Language | Python |
| Data | pandas, numpy |
| Modelling | scikit-learn (TF-IDF, cosine similarity) |
| Charts | matplotlib |
| Environment | Google Colab |

## How to Run
1. Open `Small_Catalog_Recommendation_Explorer.ipynb` in Google Colab
2. Click **Runtime > Run all**
3. No dataset or API key is needed, all data is generated inside the notebook (seed 42)

Or locally: `pip install -r requirements.txt` and open the notebook in Jupyter.

## Limitations
- The data is synthetic. It checks that the code and the evaluation are correct, it does **not** prove that real readers would be happier
- Offline recall is not the same as real satisfaction
- Some explanations are weak, because they use content similarity while the suggestion comes from CF (for example only one shared word)
- The cold-start result comes from 4 users only

## About Me
**Akshat Kesharwani** - Data Analyst / Data Scientist (fresher)

- Portfolio: https://akshatkesharwani-info.github.io/
- GitHub: https://github.com/akshatkesharwani-info

Idea source: "Top 50 Projects to Build in 2026" (Yasir Arafat Sharfi). The build, data and code here are my own.
