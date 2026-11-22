# ![CI logo](https://codeinstitute.s3.amazonaws.com/fullstack/ci_logo_small.png)

# Mapping Short-Let Supply in Bristol

**Where are professional short-let operators concentrated in Bristol, and how do their listings differ from those of casual hosts?**

An independent data analytics project using Inside Airbnb listing data (snapshot of 22 September 2026), written for an imagined local authority policy and planning team. It is not commissioned by, or affiliated with, any council.

**Interactive dashboard (Tableau Public):** https://public.tableau.com:9443/views/Wheredoshort-letoperatorsconcentrateinBristol/Dashboard1?:language=en-US&:sid=&:display_count=n&:origin=viz_share_link

---

## 1. Purpose, goals and audience

**Purpose.** Short-term lets range from residents occasionally renting a spare room to businesses running many whole homes as full-time lettings. The platform does not distinguish between them. This project separates the two and shows where each is found.

**Audience.** A council policy and planning team that needs evidence to target limited resources and design proportionate responses. The dashboard is written for non-technical readers. The notebooks hold the full method for anyone who wants to check it.

**Goals**
1. Show where professional operators are concentrated across Bristol's 34 neighbourhoods.
2. Show how operator listings differ from casual listings (availability, minimum stay, ratings, price, property type).
3. Test whether those differences are real, how large they are, and whether they survive robustness checks.
4. Identify which listing characteristics best separate operator listings from casual ones, using interpretable machine learning.
5. Let a non-technical reader explore the findings, including changing the definition of "operator".

**Definitions.** An *operator listing* belongs to a host with 3 or more listings in the data (`calculated_host_listings_count >= 3`). A *casual listing* belongs to a host with 1 or 2. The threshold is a judgement call, so the key findings are re-tested at 5 or more, and the dashboard lets the reader switch between the two.

**What this project cannot show.** The data measures short-let *supply*. It cannot show housing pressure, displacement, rent effects or harm to residents, and no causal claims are made. It is a single snapshot, so it shows nothing about change over time.

---

## 2. Key findings

- **Scope:** 2,561 active listings (of 2,932). 1,075 (42%) belong to operators (31% at the stricter 5+ definition).
- **Where:** operator share runs from 10% (Bedminster) to 63% (Central). Four areas (Central, Clifton Down, Lawrence Hill, Hotwells & Harbourside) have shares above 55%. They hold 24% of active listings but 35% of operator listings.
- **Availability:** operator listings are open more days a year (median 281 against 195). The effect is small but stable across every robustness check.
- **Minimum stay:** no meaningful difference. The apparent tilt towards 1-night stays comes from one host with 127 active listings, 124 of them with a 1-night minimum.
- **Ratings:** operator listings are rated lower (median 4.75 against 4.92). This is the largest difference found and holds in every scenario tested.
- **Prediction:** listing characteristics separate the groups only moderately (test accuracy 70% to 73% against a 50.5% baseline). Review rating is the strongest separator in both models.

### How the analysis challenged my starting assumptions
I began by expecting operators to cluster in particular areas, to run listings that look commercial (more availability, different minimum stays), and for neighbourhood and price to be the main ways to tell the two groups apart. The first held. The second held for availability but not for minimum stay, where one host explained the whole difference. The third did not hold: once property size was in the model, price added nothing detectable, and guest rating, which I had tested but not expected to lead, was the strongest separator. Neighbourhood was left out of the models by design, so they cannot say how it compares with rating.

The analysis also changed how I worked. Because a few large hosts dominate the operator group, every key test was repeated without the largest host, effect sizes were reported alongside p-values (listings from the same host are not independent), and the models were validated on host-grouped splits. Dormant listings were flagged, not deleted, because removing them moves the operator share from 38.5% to 42.0%.

---

## 3. Hypotheses and validation

The hypotheses were written before analysis and are kept as originally stated in the ETL notebook. Full tests, robustness checks and interpretation are in the EDA notebook.

| Hypothesis | Test | Result | Effect size | Survives removing the largest host? |
|---|---|---|---|---|
| **H1:** operator share differs by neighbourhood | Chi-square (34 areas), Cramér's V | Supported (χ² = 215.6, 33 d.f., p < 0.001) | V = 0.29 (moderate) | Yes (V = 0.26) |
| **H2a:** operator listings have higher availability | Mann-Whitney U | Supported | Small (rank-biserial 0.18) | Yes |
| **H2b:** operator listings have different minimum stays | Mann-Whitney U | Not supported in practice | Negligible (−0.06) | No: disappears (p = 0.82) |
| **H3:** operator listings are rated differently | Mann-Whitney U | Supported, operators lower | Medium to large (−0.42) | Yes (−0.36) |

Mann-Whitney was used instead of a t-test because the measures are heavily skewed or bunched near a ceiling. P-values are likely overstated because listings from one host are not independent, so effect sizes are treated as the more reliable figures.

---

## 4. The dashboard

**Views** 
- Three headline figures: active listings, operator listings and operator share.
- **Map:** operator share by neighbourhood.
- **Ranking:** all 34 areas ranked by operator share, labelled with listing counts. [CONFIRM: areas with fewer than 30 listings are shown in grey.]
- **Dot plots and bar charts:** median availability and median review rating for operators and casual hosts. [CONFIRM: availability plot kept or removed.]
- **Model chart:** logistic regression coefficients. [CONFIRM: kept or removed.]

**Interaction**
- **Operator threshold:** switch between 3+ and 5+ listings. [CONFIRM options.] Everything except the model chart updates.
- **Room type filter:** limits every view except the model chart.
- **Click an area on the map:** filters the figures and dot plots to that area and highlights it in the ranking. Click an empty part of the map to clear.

Each business requirement and the visual that answers it:

| Requirement | Visual (plot type) | Where else it appears |
|---|---|---|
| Where are operators concentrated? | Choropleth map; ranked bar chart | EDA V2, V3 |
| How do operator listings differ? | Dot plots of medians | EDA V1 (stacked bar), V4 and V6 (histograms), V5 (banded bar), V7 (box plots), V8 (heatmap) |
| What separates the groups? | Bar chart of model coefficients | ML notebook |

### Design and accessibility decisions
- **Information hierarchy:** title and the question first, then headline figures, then the map and ranking (where), then the comparisons (how they differ), then caveats and attribution.
- **User control:** one threshold control and one filter, placed beside the title. The same controls drive every view where they apply.
- **Consistency:** purple always means operator and grey always means casual or "too few listings to trust". Labels use the same wording throughout.
- **Confirmation and feedback:** selecting an area outlines it on the map and highlights its bar. Tooltips show the listing count next to every share.
- **Accessibility:** a single-hue sequential palette that does not rely on red/green, and direct labels on the ranking (so the map's colour is not the only route to the figures).

**Decisions that depart from common UX practice**
- **Fixed-size layout, not responsive.** The ranking shows all 34 areas, which needs a tall, fixed canvas. The dashboard is designed for desktop and is not mobile-friendly.
- **The map does not grey out small areas.** An area with fewer than 30 listings is coloured like any other on the map, so its share can look more reliable than it is. The tooltip shows the count, and the ranking flags small areas. [CONFIRM against the published dashboard.]
- **The model chart does not respond to the controls.** The model was fitted once, on all active listings at the 3+ definition, so it cannot change when the viewer does. A caption says so. [Delete if the model chart was removed.]
- **Colour is the only encoding on the map.** The ranking and tooltips back it up, but a map-only user relies on colour alone.
- **External links.** GitHub's README renderer removes the `target` attribute (as far as I know), so links in this file open in the same tab and cannot be made to open in a new one.

---

## 5. Methodology and justification

### ETL (`bristol-short-let-analysis_ETL.ipynb`)
Raw files stay untouched. All work happens on a copy, and every change records its row and column counts. Dormant listings are flagged (`is_active`) and not deleted. Empty, constant and personal-data columns are dropped with a reason recorded for each (`docs/dropped_columns.csv`). Airbnb's host ID is replaced by an anonymous `host_key`. Missing values are classified by cause before they are treated. Averages are not used to fill gaps, because the gaps are not random.

### EDA and hypothesis testing (`..._EDA.ipynb`)
Distributions first (V0), then comparisons (V1 to V8), a robustness check across both operator definitions and three listing subsets (V9), then formal tests (H1 to H3).

| Choice | Why | Alternative considered |
|---|---|---|
| Medians and rank-based tests | Availability, minimum stay, price and reviews are skewed; ratings are bunched near 5.0 | t-test: compares means, which a few extremes distort |
| Effect sizes beside p-values | With groups of this size almost any difference is "significant"; listings from one host are not independent | p-values alone |
| Chi-square with Cramér's V for H1 | Two categorical variables; V gives the strength | Grouping small areas first (tested as a check: V = 0.289 against 0.29) |
| Active listings as the main subset | Dormant listings would change the headline share from 42.0% to 38.5% | Using all listings (kept as a sensitivity check) |

### Machine learning (`..._ML.ipynb`)
The target is `is_operator` on active listings. The aim is interpretation (*which characteristics separate the groups?*), not prediction for its own sake, so interpretable models were chosen.

| Option | Verdict | Reason |
|---|---|---|
| **Logistic regression** | Used | Coefficients and odds ratios can be read directly |
| **Decision tree** | Used | Rules can be drawn and read; handles non-linear patterns without scaling |
| Random forest / gradient boosting | Not used | Might score higher, but the project needs explanation, not accuracy, and the importances are harder to defend with 2,561 listings and a handful of dominant hosts |
| k-nearest neighbours, neural networks | Not used | Opaque for this purpose and unnecessary at this data size |
| Naive Bayes | Not used | Assumes inputs are independent; V8 shows the size variables overlap heavily |
| Clustering | Not used | The target is known, so a supervised method answers the question directly |

I did not choose one model over the other in advance. After tuning, the two were statistically tied (cross-validated F1 0.563 for the tree, 0.541 for the logistic regression), so both are reported. Two model families agreeing that rating matters most is stronger evidence than either alone. The logistic regression was the more stable: removing the largest host from training cut the tree's recall from 0.647 to 0.411 but barely moved the logistic regression.

**Safeguards:** an 80/20 split grouped by host (`RANDOM_STATE = 42`) so no host is in both sets; host-count columns excluded because they define the target; preprocessing inside pipelines so nothing leaks from held-out data; 5-fold host-grouped cross-validation; the test set used once; a comparison with and without `price` (no detectable difference, so it was dropped).

---

## 6. Evaluation, limitations and alternatives

**What worked.** The area pattern, the rating gap and the availability gap are robust: they hold under both operator definitions, across listing subsets and with the largest host removed.

**What is weaker than it looks**
- The models are only moderately accurate. Cross-validated F1 is about 0.55, and the test scores (about 0.70) are probably optimistic because a few large hosts supply over half of the test set's operator listings.
- One host holds about 12% of operator listings. Any result that disappears without that host (such as minimum stay) was treated as not real.
- Availability shows what a host leaves open, not how often the listing is booked.
- Small areas (fewer than 30 listings, five of them) give unstable shares. The 30-listing cut-off is a judgement.
- A host with one Bristol listing and a large portfolio elsewhere counts as casual here.
- Single snapshot only: no change over time and no delisted properties.

**Alternative approaches I did not take**
- **Analyse at host level** (one row per host) to remove the non-independence problem.
- **Mixed-effects or clustered models** to account for listings nested within hosts.
- **Add neighbourhood to the models** to test whether rating partly stands in for geography.
- **Use several snapshots** to see change over time.
- **Join external data** (housing, licensing) to move from supply towards pressure.
- **Try a random forest** as a comparison on accuracy.

---

## 7. Prototype question: which interactive tool could improve stakeholders' interactions?

An adjustable threshold. Policy teams will disagree about what counts as a "professional" operator, and any single threshold hides that disagreement. The dashboard lets them change the definition and see which findings survive. A natural next step, not built here, would be a repeating monthly snapshot with a view of areas where operator share is rising, which a single snapshot cannot show.

---

## 8. Data, licence and privacy

**Source.** Inside Airbnb, Bristol, compiled 22 September 2026 (downloaded 12 October 2026): <https://insideairbnb.com/get-the-data/>. Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

**Attribution.** Data: Inside Airbnb, Bristol, compiled 22 September 2026 (downloaded 12 October 2026). Licensed under CC BY 4.0. Changes made: listings filtered to active listings, columns removed, host IDs replaced with pseudonymous keys, and new fields derived. This is an independent project and is not endorsed by Inside Airbnb.

**Privacy.**
- The raw files are **not** committed (`data/raw/` is git-ignored), because they include host names, profile links and other personal data.
- The processed data drops host names, profile and photo links, free text and Airbnb's host ID.
- Pseudonymisation is not anonymisation. The processed file keeps the public listing `id` and the coordinates as published in the source, so a listing can be traced back to the source, which is itself public. [CONFIRM: if you drop `id` from the processed file, update this paragraph.]
- The dashboard files contain no listing or host identifiers and no coordinates.

---

## 9. Project structure and how to run

```
.
├── data/
│   ├── raw/           not committed; download from Inside Airbnb
│   ├── processed/     listings_clean.csv, neighbourhoods.geojson
│   └── tableau/       dashboard inputs written by the EDA and ML notebooks
├── docs/              dropped_columns.csv (every dropped column and why)
├── dashboard/         Tableau workbook (.twbx) and screenshot
├── jupyter_notebooks/ ETL, EDA and ML notebooks
├── .gitignore
└── README.md
```

**To reproduce**
1. Create a virtual environment and install the dependencies [CONFIRM: `requirements.txt`].
2. Download `listings.csv.gz`, `neighbourhoods.csv` and `neighbourhoods.geojson` for Bristol from Inside Airbnb and place them in `data/raw/`.
3. Run the notebooks in order: ETL, then EDA, then ML. Each runs from the project root and uses relative paths.
4. The EDA and ML notebooks write the three files in `data/tableau/`. Open the workbook in `dashboard/` with Tableau Public.

---

## 10. Version control

The history is organised as small, single-purpose commits (for example, one per notebook stage), with messages that say why each change was made. Raw data was never committed. [CONFIRM: `git log --oneline -- data/raw` prints nothing.]

---

## 11. Use of AI

AI (Claude) was used as a project planner, a coding assistant and a tutor for interpreting statistics and building the dashboard. I set it up to coach me through the work, not to do it for me. I checked outputs against the data. Two of its suggestions did not survive that check (a small-sample explanation for the rating gap, and a non-linearity explanation for the tree's advantage), and I corrected the notebooks. A fuller note is at the end of the ML notebook.

---

## 12. Reflection on the learning journey


- **Testing assumptions.** I started with a story (neighbourhood and price explain operators) and the data told a different one (rating does). Reporting robustness checks, including removing one host, kept me from presenting a minimum-stay difference that was really a single host.
- **Statistics.** Significance is not the same as importance. With groups this size tiny differences are "significant", so effect sizes and the independence problem mattered more than p-values.
- **Practical problems.** A hard-coded file path stopped the notebook running anywhere else, and I nearly published raw data containing host information. Both were caught during review and fixed. In Tableau, some chart types were unavailable with my data setup, so I redesigned the comparison as dot plots of medians.
- **Working with AI.** It was most useful when I asked it to explain, and least reliable when it offered an explanation that I then tested.
- **For future work.** Check privacy and portability early, plan the dashboard layout before building the charts, and decide the question before choosing the model.

---

## Credits

Data: Inside Airbnb (see attribution above). Dashboard built with Tableau Public. Analysis in Python (pandas, NumPy, SciPy, scikit-learn, matplotlib, seaborn, Plotly).