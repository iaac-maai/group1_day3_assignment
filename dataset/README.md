---
license: other
license_name: nyc-open-data-terms-of-use
license_link: https://www.nyc.gov/home/terms-of-use.page
pretty_name: NYC facade typologies (FISP buildings, cycles 6–10), with facade-type clusters
size_categories:
  - 10K<n<100K
tags:
  - built-environment
  - facades
  - building-inspection
  - new-york-city
  - clustering
  - iaac-maai
  - ai-methods-1
configs:
  - config_name: default
    data_files:
      - split: train
        path: nyc_facade_typologies.csv
---

# NYC facade typologies (FISP buildings, cycles 6–10), with facade-type clusters

**15,747 buildings** in New York City taller than 6 stories, inspected under the city's Facade Inspection & Safety Program (FISP, Local Law 11) between about 2006 and 2026. Each row is **one building**: its location, what its facade is made of (summarised over all its inspection reports), its inspection history (how often it was found UNSAFE), and the **facade typology** it belongs to ("solid load-bearing brick", "terra cotta & brick (pre-war ornament)", "glass & metal curtain wall"…), found by clustering.

It turns a table of 87,500 filings (one row per report) into a **ready-to-use building table** for studying facade types and their inspection results. Made for the Day 3 assignment of **AI Methods I: Machine Learning for the Built Environment** (IAAC MaAI 2026/27).

- **GitHub repository (how it was made):** https://github.com/iaac-maai/group1_day3_assignment (notebook `notebook/unsupervised_learning.ipynb`)
- **Model fitted on it:** https://huggingface.co/iaac-maai/day3-team1-facade-typologies
- **Team:** Amelia, Jerome, Kazuko, Sahil

## Source and attribution

| | |
|---|---|
| **Original dataset** | [DOB NOW: Safety – Facades Compliance Filings](https://data.cityofnewyork.us/d/xubg-57si), NYC Department of Buildings, published on NYC Open Data |
| **Original licence** | NYC Open Data [terms of use](https://www.nyc.gov/home/terms-of-use.page): free public use, no restrictions on reuse |
| **Original content** | 87,556 facade filings (on the download date) (one per inspection report), 40 columns: building ID and address, cycle, filing type and status (SAFE / SWARMP / UNSAFE), dates, penalties, wall types and materials, inspector and owner contact details |
| **Downloaded** | 30 October 2026 |

This dataset is a derived work. Please credit the NYC Department of Buildings and NYC Open Data when you reuse it.

## What we changed

- **Cleaning** (as in our Day 1/2 notebook): exact duplicates and repeated filing numbers removed; 19,799 `NO REPORT FILED` placeholders removed (67,732 real reports left); text in capitals; dates parsed; placeholder building IDs (`1000000`…) and IDs in the wrong borough set to missing.
- **Privacy:** all **owner and inspector names, companies, addresses and licence numbers were dropped**. The only address left is the building's own (public) street address.
- **Selection:** only **INITIAL** filings (the first report of a building in a cycle) with a valid building ID (`BIN`) and a facade description: 52,245 reports.
- **Aggregation:** reports → **one row per building** (15,747 buildings).
- **Feature engineering:**
  - the 18 wall types and 12 materials (free multi-value text such as `"MASONRY; TERRA COTTA"`) were grouped into **7 material families and 6 wall families**; for each building, the **share of its reports** that mention each family (0–1);
  - `MAT_OTHER`, `WALL_OTHER` (share of reports that say "OTHER"), `N_MATERIALS` (materials per report);
  - inspection history: `N_REPORTS`, `N_UNSAFE`, `N_SWARMP`, `SHARE_UNSAFE`, `EVER_UNSAFE`, `LATEST_STATUS`, `TOTAL_PENALTY_AMT`, first and last cycle and filing year.
- **Clustering:** K-Means (k = 6) on the 13 facade-family shares only (`cluster`, `cluster_name`); Ward hierarchical clustering for comparison (`cluster_ward`); the building's position on PCA, t-SNE and UMAP maps of the same 13 features.

## The clusters

![The facade families and UNSAFE rate of each typology](https://huggingface.co/datasets/iaac-maai/day3-team1-nyc-facade-typologies/resolve/main/dataset_cluster_profiles.png)

| Cluster | Name | Share of buildings | What makes it different | UNSAFE reports (not clustered on) |
|---|---|---|---|---|
| 0 | glass & metal curtain wall | 6.0% | glass, metal and curtain walls; little brick; 75% in Manhattan | **10%** (lowest) |
| 1 | brick cavity wall | 9.7% | brick veneer on a cavity wall, the most stucco/EIFS; the most spread over Brooklyn and the Bronx | 17% |
| 2 | concrete frame | 4.9% | concrete material and concrete frame | 21% |
| 3 | terra cotta & brick (pre-war ornament) | 12.0% | terra cotta with brick and stone on load-bearing walls; 88% in Manhattan | 25% |
| 4 | solid load-bearing brick | 48.7% | plain brick, load-bearing, almost nothing else | 21% |
| 5 | unspecified facade ("other") | 18.7% | inspectors mostly wrote "OTHER" for the facade (70% of reports) | **33%** (highest) |

**Reading.** The typologies are fairly well separated (silhouette 0.38, with a clear peak at k = 6) and very stable (ARI 0.99 over 10 seeds and 10 subsamples of 80%). Curtain-wall and terra cotta buildings are distinct groups for both K-Means and Ward; the large brick mass is cut differently by the two algorithms (ARI between them: 0.59). Although the UNSAFE history was **never given to the algorithm**, the typologies differ in risk: from 10% UNSAFE reports for curtain walls to 33% for the facades described only as "other". "Pre-war" is our interpretation (terra cotta was mostly used before WWII); the data has no construction year.

![The buildings on the PCA, t-SNE and UMAP maps, coloured by cluster_name](https://huggingface.co/datasets/iaac-maai/day3-team1-nyc-facade-typologies/resolve/main/dataset_clusters_maps.png)

On the **PCA** map (axes have a meaning) the buildings form a V: traditional brick on the right (PC1 = masonry), terra cotta going up (PC2 = terra cotta), glass and metal on the left. On **t-SNE** and **UMAP** only the neighbourhoods matter: dense knots are buildings with identical descriptions (the large purple blob in UMAP is thousands of "brick, load-bearing" buildings); group sizes and the distances between groups mean nothing.

## Columns

| Column | Type | Unit | Description |
|---|---|---|---|
| `BIN` | integer | – | Building Identification Number (NYC DOB), unique per row |
| `BOROUGH` | text | – | MANHATTAN, BRONX, BROOKLYN, QUEENS, STATEN ISLAND |
| `BLOCK`, `LOT` | integer | – | Tax block and lot of the building |
| `HOUSE_NO`, `STREET_NAME` | text | – | Street address (from the latest report) |
| `FIRST_CYCLE`, `LAST_CYCLE` | integer | – | First and last FISP cycle with a report (6–10; one cycle = 5 years) |
| `N_REPORTS` | integer | – | Number of initial reports with a facade description |
| `FIRST_FILING_YEAR`, `LAST_FILING_YEAR` | integer | year | Year of the first and last report |
| `EXTERIOR_WALL_TYPES_LATEST`, `EXTERIOR_WALL_MATERIALS_LATEST` | text | – | Wall types and materials as written in the latest report |
| `MAT_BRICK`, `MAT_TERRA_COTTA`, `MAT_STONE`, `MAT_CONCRETE`, `MAT_STUCCO_EIFS`, `MAT_GLASS`, `MAT_METAL` | float | share 0–1 | Share of the building's reports that list this material family (stone = natural + cast stone; concrete = concrete + GFRC; metal = metal + sheet metal + cast iron). **Clustered on** |
| `WALL_MASONRY_LOAD_BEARING`, `WALL_MASONRY_CAVITY`, `WALL_STEEL_FRAME`, `WALL_CONCRETE_FRAME`, `WALL_CURTAIN_WALL`, `WALL_TERRA_COTTA_CLAD` | float | share 0–1 | Share of reports that list this wall family (see the notebook for the exact grouping). **Clustered on** |
| `MAT_OTHER`, `WALL_OTHER` | float | share 0–1 | Share of reports that say "OTHER" for materials / wall type |
| `N_MATERIALS` | float | – | Average number of named materials per report |
| `LATEST_STATUS` | text | – | Result of the latest report: SAFE, SWARMP (safe with repair and maintenance program), UNSAFE |
| `N_UNSAFE`, `N_SWARMP` | integer | – | Number of reports with that result |
| `SHARE_UNSAFE` | float | share 0–1 | `N_UNSAFE / N_REPORTS` |
| `EVER_UNSAFE` | integer | 0/1 | 1 if at least one report was UNSAFE |
| `TOTAL_PENALTY_AMT` | float | USD | Sum of late-filing, failure-to-file and failure-to-correct penalties on the initial reports (missing for 44 buildings) |
| `cluster` | integer | – | Facade typology assigned by the K-Means model (0–5) |
| `cluster_name` | text | – | Name of the typology (our interpretation), see the table above |
| `cluster_ward` | integer | – | Cluster from Ward hierarchical clustering (k = 6, fitted on 6,000 buildings, extended with 5 nearest neighbours). Numbers do **not** match `cluster`; for comparison only |
| `pca_x`, `pca_y` | float | – | Position on the first two principal components of the 13 facade features (31% and 15% of the variance) |
| `tsne_x`, `tsne_y` | float | – | Position on a 2D t-SNE map (`perplexity=30`, `init="pca"`). Only for plotting |
| `umap_x`, `umap_y` | float | – | Position on a 2D UMAP map (`n_neighbors=30`, `min_dist=0.1`, features jittered with sd 0.02 to separate identical buildings). Only for plotting |

## Limitations and biases

- **Descriptions, not surveys.** The facade families come from what the inspector ticked in each report. The same building can be described differently in different cycles; the shares soften this but do not remove it.
- **"OTHER" is common.** About a third of reports use "OTHER", and 19% of buildings end up in the "unspecified" typology. This group is defined by missing information, not by a construction type.
- **Missing information:** no construction year, height, floor area or renovation history. "Pre-war" in a cluster name is our interpretation.
- **Coverage:** only NYC buildings taller than 6 stories that filed at least one real initial report with a facade description; 8 reports with an invalid `BIN` (placeholder or wrong borough) are left out.
- **The clusters are one reading of the data.** Ward clustering cuts the brick buildings differently (ARI 0.59); another k would give other borders.
- **Not a safety rating.** A building's typology or past results do not say whether its facade is safe today.
- The t-SNE and UMAP coordinates are not exactly reproducible on another computer or library version.

## How to load it

```python
import pandas as pd

df = pd.read_csv("hf://datasets/iaac-maai/day3-team1-nyc-facade-typologies/nyc_facade_typologies.csv")
df.groupby("cluster_name")["SHARE_UNSAFE"].mean()

# Draw the typologies on any of the three maps: pca_x/pca_y, tsne_x/tsne_y or umap_x/umap_y
import matplotlib.pyplot as plt

for name, rows in df.groupby("cluster_name"):
    plt.scatter(rows["umap_x"], rows["umap_y"], s=3, label=name)
plt.legend(markerscale=4)
plt.show()
```

Reading `hf://` paths requires the `huggingface_hub` package.

## Authors and licence

Amelia, Jerome, Kazuko, Sahil. Course assignment for **AI Methods I: Machine Learning for the Built Environment**, IAAC MaAI 2026/27.

Derived from *DOB NOW: Safety – Facades Compliance Filings* by the NYC Department of Buildings (NYC Open Data), reused under the [NYC Open Data terms of use](https://www.nyc.gov/home/terms-of-use.page). Our additions (aggregation, features, clusters, coordinates) are released under CC BY 4.0.
