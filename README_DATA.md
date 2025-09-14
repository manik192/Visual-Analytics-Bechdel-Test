# Dataset: Origin & What I Worked With

After I scoped the project, I combined a Bechdel-style outcome with general movie metadata so I could study **how inclusivity patterns evolved by decade, studio, and genre**, and how those patterns related to **ratings, awards, and ROI**.

## Where the data came from
- **Bechdel outcome (pass/fail variants):** [add exact source link]
- **Movie metadata (title, year, genres, production companies, runtime, ratings/votes):** [add exact source link]
- **Awards (Oscars nominations/wins):** [add exact source link]
- **Economics (budget, domestic & international gross):** included with the metadata or merged from a companion file.

> I followed each source’s license/terms of use. I included links above so anyone can re-download exactly what I used.

## What the table looked like to me
- **Core fields used:** `title`, `year` (or `year_int` / `year_final`), `genres`, `production_companies`, `runtime`, `imdb_rating/avg_vote`, `imdb_votes`, `osc_*`, `budget`, `domgross`, `intgross`, `female_cast_*`, `female_crew_*`, and one or more **Bechdel outcome** columns (`binary`, `bechdel_pass`, `test`, `clean_test`).
- **Messiness I found:** the same concept had multiple column names; production companies were list-like strings; genres were comma-separated; some money columns were adjusted (e.g., `*_2013$`) while others weren’t; and the Bechdel outcome appeared in different formats.

## Why this dataset fit my question
I wanted to move from “did a film pass?” to **“which production houses consistently put out inclusive films, and when?”** For that, I needed:
- a clean **pass/fail flag**,
- **time** bucketing (decades),
- **studio/genre** group-bys,
- and some **context** (ratings, awards, ROI) to frame the story.

## Reproducing my setup
I did everything in **`project_1.ipynb`**. To re-run:
1. Create a Python env and install packages (`pandas`, `numpy`, `plotly`, `pyarrow`, `jupyterlab`).
2. Put the raw CSV(s) under `data/raw/`.
3. Open `project_1.ipynb` and **Run All**. The notebook reads raw data, cleans it, and builds the tables/figures I show in the write-up.
