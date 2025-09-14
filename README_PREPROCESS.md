# Preprocessing & Cleaning — What I Did and Why

All cleaning lives in **`project_1.ipynb`**. I wrote it to survive small schema changes so I could swap in similar CSVs without rewriting the notebook.

## My approach (in the order I executed it)

### 1) Handle schema drift
**What I did:** I used a small helper to pick the *first* existing column among name variants (e.g., `year_int | year | year_final`).  
**Why:** different drops named the same thing differently; I didn’t want to hand-rename columns each time.

### 2) Year → Decade
**What I did:** coerced the year to numeric, computed `(year // 10) * 10`, and stored strings like `"1990s"`.  
**Why:** decades smoothed noise and were the right granularity for trends.

### 3) One Bechdel flag
**What I did:** started from `binary` if present; otherwise mapped labels from `bechdel_pass | clean_test | test` (`pass/ok/yes/1 → 1`, `fail/men/nowomen/notalk/dubious/0 → 0`), then finalized `bechdel_pass_bin` as `0/1`.  
**Why:** I wanted a single, reliable field for grouping and percentages.

### 4) Oscars fields
**What I did:** normalized any nominations column to `_osc_noms_num` and any wins column to `_osc_wins_num`, coercing to numeric and filling missing with `0`.  
**Why:** I needed a clean “% with ≥1 nomination” by decade/studio and a simple prestige signal.

### 5) Production company normalization
**What I did:** stripped brackets/quotes from list-like strings, split on commas/pipes, and kept the **first** company as `production_company_norm`.  
**Why:** this gave me a pragmatic studio key for group-bys (I note the co-production limitation in the reflection).

### 6) ROI from money columns
**What I did:** preferred adjusted columns when available (e.g., `*_2013$`), coerced `budget`, `domgross`, `intgross`, computed `world = dom + int`, and `ROI = world / budget` only when `budget > 0`.  
**Why:** I wanted an economic axis for one of the exploratory charts without fabricating values.

### 7) Female cast/crew ratios
**What I did:** normalized `female_cast_ratio|female_cast_pct` and `female_crew_ratio|female_crew_pct` to numeric and kept them on a **0–1** scale (down-scaling if they came as percentages).  
**Why:** to inspect whether representation behind the camera tracked other signals.

### 8) Genres → explode
**What I did:** split comma-separated genres, trimmed whitespace, and **exploded** to one `(title, genre)` per row.  
**Why:** it made “pass % by genre” a straightforward group-by.

### 9) Standardize numeric fields
**What I did:** created consistent numeric columns like `_imdb_rating`, `_imdb_votes`, `_runtime`, `_cast_count`, `_crew_count`, `_budget`, `_domgross`, `_intgross`.  
**Why:** that consistency made correlation/EDA functions clean and reusable.

## Light QA I ran
- `bechdel_pass_bin ∈ {0,1}` after mapping.
- Ratios within `[0,1]` (rescaled if I detected values > 1).
- ROI only computed when `budget > 0`.
- Group sizes reviewed before ranking/plotting to avoid tiny-N traps.
