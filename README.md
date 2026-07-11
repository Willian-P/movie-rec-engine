# Movie Recommendation Engine

An end-to-end Data Science and Engineering project focused on building a movie recommendation system using the MovieLens dataset.

## Project Progress & Pipeline

### Data Cleaning & Integrity (EDA)
* **Deduplication:** Resolved title collisions (e.g., *War of the Worlds*) by remapping conflicting `movieId` records, protecting historical rating distributions.
* **Quality Thresholding:** Filtered out low-interaction titles using a minimum threshold of 3 ratings per movie. This streamlined the catalog by 49.3% while preserving 93.8% of core user interactions.

### Feature Engineering
* **Temporal Noise Reduction:** Dropped the `timestamp` column from ratings to focus strictly on collaborative signals.
* **Text Processing:** Extracted release years from the text strings into a dedicated numerical `year` attribute and removed text noise from titles.
* **Categorical Transformation:** Expanded the piped `genres` column into 19 binary features via One-Hot Encoding, shaping a clean mathematical matrix for modeling.
* **Artifact Persistence:** Exported the final optimized data to `../datasets/processed/` to bypass upstream pipeline recalculations.

## Next Steps
* [ ] Phase 2: Model Research, Architecture & Evaluation.

## Tech Stack
* Python
* Pandas
* Jupyter Notebook