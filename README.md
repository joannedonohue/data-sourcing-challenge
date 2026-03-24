# Movie Review Aggregator

A multi-source data pipeline merging New York Times critical reviews with TMDB film metadata — demonstrating production-grade API integration, pagination handling, and data normalization in Python.

---

## Overview

This project builds an automated pipeline that retrieves NYT movie reviews via their API, enriches each title with supplementary metadata from The Movie Database (TMDB), and outputs a clean, unified CSV dataset. The pipeline handles pagination, API rate limiting, missing data, and credential management — addressing the full set of production concerns for a real-world data integration task.

---

## Pipeline Architecture

```
NYT Movies API
    │
    ├── Paginated review retrieval (Jan 2013 – May 2023)
    │
    └── Per-title title string extraction
              │
         TMDB API enrichment
              │
         ├── genres
         ├── runtime
         ├── vote_average
         └── cast info
              │
         Data merge & cleaning
              │
         ├── Title-based joins
         ├── Null / duplicate removal
         └── Column normalization
              │
         CSV export → output/
```

---

## Coverage

- **Date range:** January 2013 – May 2023 (10+ years of NYT critical reviews)
- **Enrichment source:** TMDB metadata for all matched titles

---

## Output Schema

| Column | Source | Description |
|---|---|---|
| title | NYT + TMDB | Movie name |
| nyt_review_summary | NYT API | Critic assessment text |
| genres | TMDB | Genre classification |
| vote_average | TMDB | Audience rating |
| runtime | TMDB | Film duration (minutes) |

---

## Engineering Features

- **Pagination handling:** Full NYT review archive retrieved across multiple API pages
- **Rate limit management:** Retry logic with back-off to avoid API throttling
- **Credential security:** API keys managed via `.env` files (not hardcoded)
- **Modular functions:** Each pipeline stage is independently callable and extensible
- **Error handling:** Graceful fallback for titles not found in TMDB

---

## Tech Stack

| Component | Tool |
|---|---|
| HTTP requests | requests |
| Data manipulation | pandas |
| Credential management | python-dotenv |
| APIs | NYT Movies API, TMDB API |
| Language | Python |

---

## Repository Structure

```
movie-review-aggregator/
├── retrieve_movie_data_final.ipynb   # Full pipeline implementation
├── output/                           # Exported CSV datasets
└── README.md
```

---

## Outcomes

- Built a complete end-to-end data pipeline integrating two external APIs across 10+ years of review history
- Implemented pagination, rate-limit retry logic, and secure credential handling — demonstrating production-ready API integration practices
- Produced a clean, normalized dataset combining critical sentiment with audience ratings and film metadata
- Modular architecture makes the pipeline straightforwardly extensible to additional data sources (Rotten Tomatoes, IMDb, etc.)

---

## Getting Started

```bash
pip install requests pandas python-dotenv
# Add NYT_API_KEY and TMDB_API_KEY to a .env file
jupyter notebook retrieve_movie_data_final.ipynb
```
