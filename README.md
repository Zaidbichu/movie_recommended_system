# Movie Recommender System

A movie recommendation system that combines local TF-IDF similarity over a curated dataset with live TMDB (The Movie Database) metadata to serve personalized movie recommendations. Includes a FastAPI backend that exposes recommendation and TMDB helper endpoints, and a Streamlit frontend for interactive browsing.

## Features
- TF-IDF based similarity on a local dataset (df.pkl, tfidf_matrix.pkl, indices.pkl, tfidf.pkl)
- TMDB-powered metadata (posters, genres, details) and discovery endpoints
- Combined "bundle" endpoint returning details + TF-IDF recommendations + genre recommendations
- Streamlit UI for search, details, and recommendation browsing

## Stack
- Language: Python (100%)
- Frameworks/runtimes:
  - FastAPI (API)
  - Streamlit (UI)
  - uvicorn (ASGI server)
- Notable libraries:
  - numpy, pandas, scipy, scikit-learn (data + TF-IDF)
  - httpx (async HTTP to TMDB)
  - python-dotenv (env)
  - pydantic (models/validation)

## Repository layout
```
.
├── app.py                  # Streamlit frontend (single-file app)
├── main.py                 # FastAPI backend + TF-IDF logic + TMDB helpers
├── requirements.txt        # Python dependencies
├── .env                    # Environment variables (TMDB_API_KEY) — rotate if present in repo
├── df.pkl                  # Local dataframe used for TF-IDF (must exist)
├── indices.pkl             # Title -> index mapping used by TF-IDF
├── tfidf_matrix.pkl        # TF-IDF matrix (sparse/scipy)
├── tfidf.pkl               # TF-IDF vectorizer object
└── README.md               # (this file)
```

How it fits together:
- main.py runs a FastAPI app exposing TMDB helpers, home feed endpoints, TF-IDF recommendations, and a bundle endpoint that returns movie details + two types of recommendations.
- app.py is a Streamlit client that calls the backend to search, show details, and render recommendation posters. The Streamlit UI expects the backend to be reachable (API_BASE in app.py).

## Requirements
- Python 3.10+ recommended (check reqs for compatibility)
- Install dependencies:
```bash
python -m venv .venv
source .venv/bin/activate   # or .venv\Scripts\activate on Windows
pip install -r requirements.txt
```

## Environment
Create a .env file in the project root with:
```
TMDB_API_KEY="your_tmdb_api_key_here"
```
Notes:
- The FastAPI app will raise at startup if TMDB_API_KEY is missing.
- If a TMDB API key is already present in the repository `.env`, rotate it (do not commit long-lived secrets to a public repo).

## Data / Pickle assets
This project depends on the following pickle files being present in the repository root:
- df.pkl
- indices.pkl
- tfidf_matrix.pkl
- tfidf.pkl

The API loads these on startup (see `main.py` startup event). If these files are missing, TF-IDF routes will fail. There is no included script in the repo to regenerate them — you'll need to run your dataset preprocessing pipeline (not included) to produce them.

## Running locally

1) Start the FastAPI backend
```bash
# from repository root
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```
This will load the pickles at startup; make sure `.env` and the .pkl files are present.

2) Start the Streamlit UI (in a separate terminal)
```bash
streamlit run app.py
```
By default app.py points to an API base in `app.py`:
```py
API_BASE = "https://movie-rec-466x.onrender.com" or "http://127.0.0.1:8000"
```
If you run the backend locally, Streamlit will connect to `http://127.0.0.1:8000`. You can edit `API_BASE` in `app.py` or set a query param to route.

## API Endpoints (examples)

Base: http://127.0.0.1:8000

- GET /health
  - Returns basic health: {"status": "ok"}

- GET /home
  - Query params: category (popular|trending|top_rated|now_playing|upcoming), limit
  - Example:
    ```
    curl "http://127.0.0.1:8000/home?category=popular&limit=12"
    ```

- GET /tmdb/search
  - Use for TMDB keyword search (returns TMDB raw shape with `results`)
  - Example:
    ```
    curl "http://127.0.0.1:8000/tmdb/search?query=avengers&page=1"
    ```

- GET /movie/id/{tmdb_id}
  - Returns TMDB movie details (title, overview, poster_url, backdrop_url, genres)
  - Example:
    ```
    curl "http://127.0.0.1:8000/movie/id/299534"
    ```

- GET /recommend/genre
  - Params: tmdb_id, limit
  - Discovers movies from the first genre of the provided movie id.

- GET /recommend/tfidf
  - Params: title, top_n
  - Returns local TF-IDF similarity results (title + score) from the local dataset.

- GET /movie/search
  - Params: query, tfidf_top_n, genre_limit
  - Returns a bundle with movie details, TF-IDF recommendations (with attached TMDB posters when available), and genre recommendations.
  - Example:
    ```
    curl "http://127.0.0.1:8000/movie/search?query=Inception&tfidf_top_n=8&genre_limit=8"
    ```

## Common issues & troubleshooting
- "TMDB_API_KEY missing" on startup: ensure `.env` exists with TMDB_API_KEY.
- Errors loading pickles: verify df.pkl and other pickle files are present and readable; file corruption or incompatible Python/pandas/numpy versions may cause deserialization errors.
- Large pickle files: df.pkl and tfidf_matrix.pkl are large — check disk and memory when loading; streaming to a production service may need different handling.

## Security & best practices
- Do NOT commit real API keys to source control. If a key is present in `.env`, rotate it and remove it from the repo history.
- Consider storing large pickles in an artifact storage (S3, GCS) rather than in the repo for production deployments.

## Contributing
- Open an issue or PR. If adding data preprocessing scripts, include clear instructions to regenerate the `.pkl` artifacts.
- Follow the existing code style; tests are not included — adding tests around TF-IDF functions and API responses would be helpful.

## License
Add a LICENSE file appropriate for your project. (No license file is included in the repo at present.)

## Contact
Repository owner: @Zaidbichu
