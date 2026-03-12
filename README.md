# Movie Review Aggregator

A multi-source data pipeline that combines New York Times movie reviews with TMDB film metadata — demonstrating production-grade API integration, pagination handling, and data normalization in Python.

## Overview

This pipeline retrieves critical reviews from the NYT Movies API, enriches each title with cast, genre, and rating data from The Movie Database (TMDB), merges both sources into a clean unified dataset, and exports to CSV for downstream analysis.

## Pipeline Steps

1. **NYT API**: Query movie reviews by topic and date range (Jan 2013 - May 2023) with full pagination support
2. **TMDB API**: For each reviewed title, retrieve metadata (genres, runtime, vote average, cast)
3. **Merge & clean**: Join on title, normalize column types, drop duplicates and nulls
4. **Export**: Save to structured CSV for analysis or visualization

## Features

- Handles API rate limiting with configurable retry logic
- Environment variable-based credential management (.env)
- Modular functions for each pipeline stage — easy to extend with new sources
- Graceful error handling for titles not found in TMDB

## Stack

Python | requests | pandas | python-dotenv | NYT API | TMDB API

## Setup

Run: pip install -r requirements.txt
Copy .env.example to .env and add your NYT_API_KEY and TMDB_API_KEY
Then: jupyter notebook retrieve_movie_data.ipynb

## Output Schema

| Column | Description |
|--------|-------------|
| title | Movie title |
| nyt_review_summary | NYT critic summary |
| genres | TMDB genre list |
| vote_average | TMDB audience rating |
| runtime | Film runtime (minutes) |
