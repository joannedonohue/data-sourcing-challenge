# data-sourcing-challenge

# Movie Reviews and Details Collector

This project retrieves movie reviews from the New York Times (NYT) API and corresponding movie details from The Movie Database (TMDB) API, processes the data, and merges it into a single dataset.

## Requirements

- Python 3.x
- Requests library
- Pandas library
- Python-dotenv library

## Setup

1. Clone the repository.
2. Install the required libraries:

    ```sh
    pip install requests pandas python-dotenv
    ```

3. Create a `.env` file in the project root directory and add your API keys:

    ```env
    NYT_API_KEY=your_nyt_api_key
    TMDB_API_KEY=your_tmdb_api_key
    ```

## Usage

### Import Required Libraries and Set Up Environment Variables

```python
import requests
import time
from dotenv import load_dotenv
import os
import pandas as pd
import json

load_dotenv()

nyt_api_key = os.getenv("NYT_API_KEY")
tmdb_api_key = os.getenv("TMDB_API_KEY")


## Access the New York Times API
### Set up the parameters and query URL for the NYT API to retrieve movie reviews.
### In this example, the query is pulling movie reviews around the topic of "love" from Jan 1, 2013 to May 31, 2023 and is limited to the first 20 pages of results, 10 results per page. 

url = "https://api.nytimes.com/svc/search/v2/articlesearch.json?"
filter_query = 'section_name:"Movies" AND type_of_material:"Review" AND headline:"love"'
sort = "newest"
field_list = "headline,web_url,snippet,source,keywords,pub_date,byline,word_count"
begin_date = "20130101"
end_date = "20230531"

query_url = (url + "api-key=" + nyt_api_key + "&begin_date=" + begin_date + "&end_date=" + end_date
            + "&fq=" + filter_query + "&sort=" + sort + "&fl=" + field_list)
print(query_url)

### Retrieve and store the reviews in a list.

reviews_list = []
for page in range(0, 20):
    query_url_page = query_url + "&page=" + str(page)
    reviews = requests.get(query_url_page).json()
    time.sleep(12)
    
    for review in reviews["response"]["docs"]:
        try:     
            reviews_list.append(review)
            print(f"{review['headline']['print_headline']} found! Appending review. Page {page}")   
        except:
            print(f"No results for page {page}")
            break

### Convert the list of reviews to a DataFrame.

reviews_df = pd.json_normalize(reviews_list)
reviews_df["title"] = reviews_df["headline.main"].apply(lambda st: st[st.find("\u2018")+1:st.find("\u2019 Review")])
reviews_df["keywords"] = reviews_df["keywords"].apply(lambda keyword_list: ";".join([f"{item['name']}: {item['value']}" for item in keyword_list]))
titles = reviews_df['title'].to_list()

## Access The Movie Database API

### The Movie Database API provides additional information about these films, such as genre, language, filming locations and so on.

### Prepare and execute queries to TMDB API to retrieve movie details.

url = "https://api.themoviedb.org/3/search/movie?query="
tmdb_key_string = "&api_key=" + tmdb_api_key
tmdb_movies_list = []
request_counter = 1

for title in titles:
    if request_counter % 50 == 0:
        time.sleep(12)
    request_counter += 1
    
    movies = requests.get(url + title + tmdb_key_string).json()
    try:
        movie_id = movies["results"][0]["id"]
        url_2 = "https://api.themoviedb.org/3/movie/"
        results = requests.get(url_2 + str(movie_id) + "?api_key=" + tmdb_api_key).json()
        
        genres = [genre['name'] for genre in results.get('genres', [])]
        spoken_languages = [language['english_name'] for language in results.get('spoken_languages', [])]
        production_countries = [country['name'] for country in results.get('production_countries', [])]
        
        movie_data = {
            "title": title,
            "original_title": results.get("original_title"),
            "budget": results.get("budget"),
            "original_language": results.get("original_language"),
            "homepage": results.get("homepage"),
            "overview": results.get("overview"),
            "popularity": results.get("popularity"),
            "runtime": results.get("runtime"),
            "revenue": results.get("revenue"),
            "release_date": results.get("release_date"),
            "vote_average": results.get("vote_average"),
            "vote_count": results.get("vote_count"),
            "genres": genres,
            "spoken_languages": spoken_languages,
            "production_countries": production_countries
        }
        
        tmdb_movies_list.append(movie_data)
        print("Found " + title)
    except:
        print(title + " not found.")

### Convert the list of movie details to a DataFrame.

movies_df = pd.DataFrame(tmdb_movies_list)

## Merge and Clean the Data for Export
### Merge the DataFrames from NYT and TMDB, clean the data, and export it to a CSV file.

merged_df = pd.merge(movies_df, reviews_df, on="title", how="inner")

columns_to_fix = ["genres", "spoken_languages", "production_countries"]
characters_to_remove = ["[", "]", "'"] 

for column in columns_to_fix:
    merged_df[column] = merged_df[column].astype(str)
    for char in characters_to_remove:
        merged_df[column] = merged_df[column].str.replace(char, "")

merged_df.drop(columns=["byline.person"], inplace=True)
merged_df.drop_duplicates(inplace=True)
merged_df.reset_index(drop=True, inplace=True)

merged_df.to_csv("output/collected_data.csv", index=False)

## Output
### The final merged dataset is saved as collected_data.csv in the output directory.


This README provides an overview of the project, setup instructions, and usage details, making it easier for users to understand and run your code.
