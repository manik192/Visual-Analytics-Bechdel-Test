## Embedding Plotly in WordPress (Astra) — Why It Was Painful, How I Fixed It, and the Preprocessing I Did

**Project:** *Who Really Makes Inclusive Films?*  
**Codebase:** `project_1.ipynb` (I generated every figure here)  
**Stack:** WordPress (Astra theme, block editor), Hostinger

I assumed “export Plotly HTML → paste into WordPress” would take five minutes. It didn’t. Below is exactly what broke, how I fixed it, and the preprocessing decisions that made my charts render fast and update reliably.

---

## What I Expected vs What Happened

- **Expectation:** Paste full Plotly HTML into a block and publish.
- **Reality:** WordPress sanitizes `<script>`, optimization plugins shuffle/minify JavaScript, caching hides updates, and cross-origin rules block embeds. I had to rework **how** I publish charts and **what** I ship inside each HTML.

---

## The Painful Parts (and How I Solved Them)

### 1) Gutenberg strips `<script>` tags
- **Symptom:** Pasting the full Plotly HTML (with `<script>`) into a “Custom HTML” block resulted in nothing or half-rendered charts.
- **Why:** WordPress sanitizes scripts for security.
- **Fix (what worked for me):** Use an **`<iframe>`** that points to a standalone chart HTML hosted on my domain.

    <iframe src="https://mydomain.com/charts/decade_trend.html"
            width="100%" height="560" loading="lazy" style="border:0;"></iframe>

### 2) Caching hid my updates
- **Symptom:** I overwrote `decade_trend.html` on the server, but the page still showed the old chart.
- **Why:** Browser cache + WordPress/Hostinger cache (and sometimes Cloudflare).
- **Fixes I used:**
  - Add a quick **cache buster** once: `decade_trend.html?v=2025-09-14`.
  - Or disable caching for the charts folder with a tiny `.htaccess`:

        <FilesMatch "\.(html|json)$">
          Header set Cache-Control "no-cache, no-store, must-revalidate"
          Header set Pragma "no-cache"
          Header set Expires 0
        </FilesMatch>

  - Clear plugin/server cache after overwriting files.

### 3) Optimization plugins broke interactivity
- **Symptom:** Blank or half-initialized charts, especially on mobile.
- **Why:** “Defer/Combine/Minify JS” options sometimes reorder or wrap Plotly code.
- **Fix:** **Exclude** the page or the `/charts/` path from optimization. Once excluded, charts loaded consistently.

### 4) Mixed content / cross-origin
- **Symptom:** Embeds failed when I used another host or HTTP.
- **Why:** HTTPS vs HTTP mismatch, or host disallowed iframes.
- **Fix:** Keep everything **HTTPS** and **same-origin** when possible (I hosted charts on my own domain).

### 5) Iframe sizing & responsiveness
- **Symptom:** Scrollbars or squished charts.
- **Fix:** `width="100%"` and a fixed `height` (560–640px). Plotly is responsive within that container.

---

## What Finally Worked (My Stable Publishing Pipeline)

1. In **`project_1.ipynb`**, I export each figure as a *standalone* HTML:

        fig.write_html(
            "charts/decade_trend.html",
            include_plotlyjs="cdn",
            full_html=True
        )

2. I upload/overwrite `charts/*.html` to `/public_html/charts/` on my Hostinger site.
3. In WordPress, I embed with an **iframe** (Custom HTML block):

        <iframe src="https://mydomain.com/charts/decade_trend.html"
                width="100%" height="560" loading="lazy" style="border:0;"></iframe>

4. When I update a chart, I **overwrite the same file**. If I don’t see changes, I add `?v=<date>` once or clear cache.

**Result:** No more fights with sanitization or page scripts. Updating a chart takes seconds.

---

## Preprocessing I Did (and Why It Helped the Embeds)

I wasn’t just cleaning data for accuracy—the choices below made chart files **lighter, faster, and more reliable** inside WordPress.

### A) Pre-aggregate before plotting
I computed small, tidy frames in the notebook so each chart only shipped the data it needed:
- `decade_stats` — Bechdel pass % and “≥1 Oscar nomination %” per decade  
- `studio_pass` — pass % by normalized production company (with a minimum N)  
- `g` — average ROI vs pass % per studio (bubble size = #titles)  
- `genre_trend` — pass % by (exploded) genre  
- `corr` — pairwise correlations across selected numerics

**Why it helped:** Smaller JSON inside each Plotly figure → faster loads and fewer plugin timing issues.

### B) Round values and prune columns
I rounded percentages to 2 decimals and dropped unused columns from each chart’s DataFrame.

**Why it helped:** The HTML payload shrank; tooltips stayed clean.

### C) `include_plotlyjs="cdn"`
Each chart references Plotly from a CDN instead of embedding the entire library per file.

**Why it helped:** The chart HTMLs got smaller and loaded faster on repeat views.

### D) Avoid mega-traces
I filtered by minimum N for categories and avoided giant traces.

**Why it helped:** Less browser work, fewer chances for a plugin to choke.

### E) One chart per HTML
I exported one figure per HTML file.

**Why it helped:** Cache-busting is simple; a broken figure can’t cascade to others.

---

## The Sequence of Steps

All of this lives in **`project_1.ipynb`**:

1. **Handle schema drift**  
   I used a helper to pick the *first available* column among variants (e.g., `year_int | year | year_final`).  
   *Why:* My sources differed a bit; I didn’t want to hand-rename every time.

2. **Year → Decade**  
   Coerced year to numeric and computed `(year // 10) * 10` → strings like `"1990s"`.  
   *Why:* Decades matched the story granularity and reduced noise.

3. **Unified Bechdel flag**  
   Started with `binary` if present; otherwise mapped `bechdel_pass | test | clean_test` to `bechdel_pass_bin ∈ {0,1}` (e.g., `pass/ok/yes/1 → 1`, `fail/men/nowomen/notalk/dubious/0 → 0`).  
   *Why:* I wanted one consistent field for grouping and percentages.

4. **Oscars normalization**  
   Mapped nomination variants to `_osc_noms_num`, win variants to `_osc_wins_num`, coerced numeric, filled missing with `0`.  
   *Why:* For “% with ≥1 nomination” by decade and studio.

5. **Production company normalization**  
   Stripped brackets/quotes from list-like strings, split on commas/pipes, and kept the **first** company as `production_company_norm`.  
   *Why:* A pragmatic studio key for group-bys (I acknowledge the co-production limitation).

6. **ROI from money fields**  
   Preferred adjusted columns (`*_2013$`) when available; computed `world = domestic + international`; `ROI = world / budget` only when `budget > 0`, else `NaN`.  
   *Why:* Gives an economic axis for exploratory plots without fabricating values.

7. **Female cast/crew ratios**  
   Normalized `female_cast_ratio|female_cast_pct` and `female_crew_ratio|female_crew_pct` to numeric on a **0–1** scale (downscaled if they arrived as percentages).  
   *Why:* To check associations between representation and outcomes.

8. **Genres → explode**  
   Split comma-separated genres, trimmed whitespace, and **exploded** to one `(title, genre)` per row.  
   *Why:* Enables clean per-genre pass-rate charts.

9. **Standard numeric fields**  
   Built `_imdb_rating`, `_imdb_votes`, `_runtime`, `_cast_count`, `_crew_count`, `_budget`, `_domgross`, `_intgross`.  
   *Why:* Predictable inputs for correlations and quick EDA.

**Light QA I ran**
- `bechdel_pass_bin ∈ {0,1}` after mapping  
- Ratios in `[0,1]` (downscale if I spot `>1`)  
- ROI only if `budget > 0`  
- Count thresholds for categories to avoid tiny-N “winners”

---

## My Update Checklist (What I Actually Do Now)

1. Edit chart cell(s) in `project_1.ipynb`; re-run.
2. Export:

        fig.write_html("charts/<name>.html", include_plotlyjs="cdn", full_html=True)

3. Upload/overwrite `/public_html/charts/<name>.html`.
4. Hard refresh the post. If stale, add `?v=<new-date>` once or clear caches.

---

## How I Knew It Was WordPress (Not Plotly)

- The **same HTML** worked perfectly in a browser tab but failed inside the post → sanitization/optimization.  
- The chart **worked on a blank page** but not on the blog home → a theme/optimization setting was colliding.  
- Changing **only** the iframe `src` to a different domain flipped success/failure → cross-origin or mixed-content.

---
