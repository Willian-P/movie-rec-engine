# Movie Recommendation Engine

An end-to-end Data Science and Engineering project focused on building a movie recommendation system using the MovieLens dataset.

## Current Stage
* **Data Integrity & Cleansing:** Resolved movie title duplication anomalies by merging overlapping IDs (`movieId`), preserving historical user ratings while ensuring referential integrity.
* **Quality Threshold:** Applied a minimum threshold of 3 ratings per movie (removed 49.3% of low-interaction movies while preserving 93.8% of user ratings).
* **Feature Engineering & Text Processing:** Extracted movie release years from titles, dropped temporal noise (`timestamp`), and converted the multi-label `genres` column into sparse binary features via One-Hot Encoding.

## Tech Stack
* Python
* Pandas
* Jupyter Notebook