# Technical Report: Visualization Choices & Reflection

**Project:** *Who Really Makes Inclusive Films?*  
**Codebase:** `project_1.ipynb` (I generated and exported every chart from this single notebook)

---

## My design principles (after building the visuals)

- **Comparability first:** shared percent scales, sorted categories, consistent axes.
- **Context matters:** tooltips include both the **value and N**; I filter or annotate tiny groups.
- **Be honest about limits:** ROI and correlations are descriptive, not causal.
- **Keep it readable:** one palette, clear titles/subtitles, minimal gridlines, and legible labels.

---

## 1) Decade trends — line chart

**Question I asked:** Is inclusivity trending up/down over time, and how does prestige track with it?

**Why this chart:** Lines communicate continuity and turning points better than columns. I plotted two series—**Bechdel pass %** and **% with ≥1 Oscar nomination**—on the same 0–100% y-axis to keep the comparison honest.

**Design choices I made:**
- Aggregation: per-decade means (pass %) and per-decade share with ≥1 nomination.
- Encodings: decade on x (ordered), percentage on y; two strokes with clear legend.
- Labels/tooltips: decade, percentage (rounded), and **N** for transparency.
- Smoothing: none—I preferred the true signal over smoothed curves.

**Trade-offs:** Decade bucketing hides year-to-year variance, but it reduced noise and matched the story scale I wanted.

---

## 2) Studios ranked by pass % — sorted bars (with min-N)

**Question I asked:** Which production houses consistently release inclusive films?

**Why this chart:** For categorical rankings, sorted bars are the clearest and least “clever.”

**Design choices I made:**
- Group key: `production_company_norm` (first listed company) with a **minimum N** (e.g., ≥10 titles).
- Encodings: studio on y (sorted), pass % on x; tooltips show pass % and N.
- Reference: a faint vertical line at the overall mean pass % for context.

**Trade-offs:** Using only the **first** production company simplifies co-productions and can be lossy; I called this out in the write-up.

---

## 3) ROI vs pass % — bubble scatter

**Question I asked:** Do studios with higher inclusivity also see better returns?

**Why this chart:** A scatter shows **relationship + dispersion**; bubble size gives volume context (#titles), which prevents over-reading tiny catalogs.

**Design choices I made:**
- Aggregation: per-studio **avg ROI** (world/budget) vs **pass %**, bubble size = N.
- Axes: linear ROI; when outliers dominated, I previewed a capped view for sanity.
- Tooltips: studio, pass %, avg ROI, N; I annotated notable outliers.

**Trade-offs:** ROI is noisy (inflation bases can mix). I framed this as exploratory, not causal.

---

## 4) Genres ranked by pass % — sorted bars (optional polar alt)

**Question I asked:** Which genres over- or under-perform on inclusivity?

**Why this chart:** Sorted bars make category comparison direct. I tried a polar view for variety, but bars tell the story cleaner.

**Design choices I made:**
- Preprocessing: **explode** comma-separated genres to one (title, genre) per row.
- Encodings: genre on y (sorted), pass % on x; N in tooltips; small-N categories filtered or grayed.
- Labels: concise axis labels and short genre names to avoid truncation.

**Trade-offs:** Exploding genres means a title can contribute to multiple categories by design; acceptable for per-genre rates, but I stated it explicitly.

---

## 5) Correlation heatmap — rounded values

**Question I asked:** What co-moves with what (runtime, ratings, votes, ROI, ratios)?

**Why this chart:** A heatmap gives a fast “scan” of linear associations without heavy modeling.

**Design choices I made:**
- Inputs: numeric columns (e.g., `_imdb_rating`, `_imdb_votes`, `_runtime`, `female_*_ratio`, `_roi`, awards).
- Encodings: symmetric matrix with a diverging color scale.
- Precision: values **rounded to 2 decimals** to avoid fake precision.

**Trade-offs:** Correlation ≠ causation; I used it as an EDA sense check.

---

## 6) Crew vs cast — scatter with parity line

**Question I asked:** Does behind-the-camera representation track on-screen representation?

**Why this chart:** A simple scatter with a **y = x parity line** makes “above/below parity” obvious at a glance.

**Design choices I made:**
- Points: each title (or grouped if too noisy) with `female_crew_ratio` on x and `female_cast_ratio` on y.
- Guides: faint 45° parity line; I only added a trend line if it was clearly justified.
- Tooltips: title (or group), the two ratios, and year/decade.

**Trade-offs:** Missing ratio fields shrink the sample; I reported coverage in the caption.

---

## What worked (after I shipped the visuals)

- **Simple encodings + sorted lists** gave me clear, legible rankings.
- **Min-N filters and N in tooltips** prevented misleading “top spots” from tiny groups.
- **Consistent percent scales** across charts made mental comparisons easier.
- **Standalone Plotly HTML exports** were painless to host and trivial to update (overwrite the file, done).

---

## What didn’t (or I had to manage)

- **Studio key**: the “first production company” heuristic flattens co-production nuance.
- **ROI noise**: mixed inflation bases can blur the signal; I kept it descriptive.
- **Small categories**: some studios/genres/decades had low counts—without N or CIs, ranks can overstate certainty.

---

## What I’d change next time

1. **Studio entity resolution**: merge aliases/subsidiaries and weight co-productions instead of “first only.”  
2. **Money consistency**: enforce a single inflation basis before computing ROI; consider winsorizing outliers or showing a log-scaled variant.  
3. **Show uncertainty**: display **N** by default and add **95% CIs** (or Bayesian intervals) on category bars.  
4. **Temporal detail**: add a 5-year rolling line alongside the decade view.  
5. **Richer inclusion signals**: go beyond Bechdel (crew role breakdowns, character-level metadata, script-level NLP—ethically sourced).

---

**Repro note:** Everything—cleaning, aggregation, and plotting—lives in **`project_1.ipynb`**. Run the notebook end-to-end to regenerate the figures and export them with `fig.write_html(...)`.
