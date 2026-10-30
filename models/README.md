---
license: mit
library_name: sklearn
tags:
  - sklearn
  - clustering
  - unsupervised-learning
  - kmeans
  - facades
  - building-inspection
  - built-environment
  - iaac-maai
  - ai-methods-1
datasets:
  - iaac-maai/day3-team1-nyc-facade-typologies
---

# NYC facade typologies: grouping buildings by what their facade is made of

This model assigns a **New York City building** to one of **six facade typologies** (solid load-bearing brick, brick cavity wall, terra cotta & brick, concrete frame, glass & metal curtain wall, unspecified "other" facade) based on the **materials and wall types** listed in its facade inspection reports. It is a scikit-learn pipeline (`KMeans`, k = 6) fitted on 15,747 buildings of NYC's Facade Inspection & Safety Program (FISP, Local Law 11). It was built as a course assignment for AI Methods I (IAAC MaAI 2026/27).

- **GitHub repository (full analysis):** https://github.com/iaac-maai/group1_day3_assignment
- **Dataset with the typology of every building:** https://huggingface.co/datasets/iaac-maai/day3-team1-nyc-facade-typologies
- **Team:** Amelia, Jerome, Kazuko, Sahil

## Intended use

- **Intended users and use:** the NYC Department of Buildings (DOB) or facade consultants planning an inspection cycle. Knowing a building's typology helps to **send the right specialist** (terra cotta, curtain wall, masonry), plan **typical repairs and budgets** per typology, and see which typologies need attention first. It is also a teaching model.
- **Out-of-scope uses:**
  - Deciding whether a facade is **safe**: the typology is not a safety rating, and only an inspection can say that.
  - Buildings **outside New York City**, or of **6 stories or fewer**: they were not in the data.
  - Facade systems missing from the inspection forms (for example timber, or new panel systems that inspectors record as "OTHER"): the model will still put them into one of the six groups.
  - Any decision about a specific owner.

## Model details

| | |
|---|---|
| **Task** | Clustering (unsupervised) |
| **Algorithm** | `KMeans` |
| **Key hyperparameters** | `n_clusters=6`, `n_init=10`, `random_state=42` |
| **Preprocessing** | None inside the pipeline. The 13 inputs are all **shares between 0 and 1** (the same unit), so no `StandardScaler`: scaling would give rare families (glass, curtain wall) much more weight than brick |
| **Assigner** | Not needed: K-Means assigns new buildings with `.predict()` |
| **File** | `model.joblib`: the whole fitted scikit-learn `Pipeline` (65 KB) |
| **Library versions** | scikit-learn 1.9.1, pandas 3.0.6, numpy 2.5.3, joblib 1.6.0, Python 3.12.14 |
| **Authors** | Amelia, Jerome, Kazuko, Sahil |

## Input features

The model expects a `pandas.DataFrame` with exactly these **13 columns**, in this order. Each value is the **share of the building's inspection reports** that list this family (0 to 1). For a single report, use 1 (listed) or 0 (not listed).

| Column | Type | Unit | Description (grouped from the original report values) | Preprocessing |
|---|---|---|---|---|
| `MAT_BRICK` | numeric | share 0–1 | material MASONRY | none |
| `MAT_TERRA_COTTA` | numeric | share 0–1 | material TERRA COTTA | none |
| `MAT_STONE` | numeric | share 0–1 | NATURAL STONE, CAST STONE | none |
| `MAT_CONCRETE` | numeric | share 0–1 | CONCRETE, GFRC | none |
| `MAT_STUCCO_EIFS` | numeric | share 0–1 | STUCCO/EIFS | none |
| `MAT_GLASS` | numeric | share 0–1 | GLASS | none |
| `MAT_METAL` | numeric | share 0–1 | METAL, SHEET METAL, CAST IRON | none |
| `WALL_MASONRY_LOAD_BEARING` | numeric | share 0–1 | BRICK MASONRY, MASONRY LOAD-BEARING WALL, BROWNSTONE, STONE MARBLE, STONE LIMESTONE | none |
| `WALL_MASONRY_CAVITY` | numeric | share 0–1 | MASONRY CAVITY WALL, CONCRETE BLOCK MASONRY | none |
| `WALL_STEEL_FRAME` | numeric | share 0–1 | STEEL FRAME | none |
| `WALL_CONCRETE_FRAME` | numeric | share 0–1 | CONCRETE FRAME, CONCRETE | none |
| `WALL_CURTAIN_WALL` | numeric | share 0–1 | CURTAIN WALL, METAL & GLASS, GLASS, METAL SIDING/PANELS | none |
| `WALL_TERRA_COTTA_CLAD` | numeric | share 0–1 | wall type TERRA COTTA | none |

"OTHER" is not an input. A building described only as "OTHER" has all 13 values at 0, and the model puts it in the "unspecified facade" group.

## Output and clusters

`model.predict(X)` returns the typology number of each building:

| Cluster | Name | Share of the buildings | Defining features | Not used by the model |
|---|---|---|---|---|
| 0 | **glass & metal curtain wall** | 6.0% | glass 0.88, metal 0.71, curtain wall 0.75, little brick | 75% Manhattan; **10% UNSAFE reports** (lowest) |
| 1 | **brick cavity wall** | 9.7% | brick 0.98 on a cavity wall 0.81, the most stucco/EIFS | the most spread out (34% Manhattan, 33% Brooklyn); 17% UNSAFE |
| 2 | **concrete frame** | 4.9% | concrete 0.82, concrete frame 0.74 | 61% Manhattan; 21% UNSAFE |
| 3 | **terra cotta & brick (pre-war ornament)** | 12.0% | terra cotta 0.89, brick 0.95, load-bearing 0.86, stone 0.26 | 88% Manhattan; 25% UNSAFE, 57% ever UNSAFE |
| 4 | **solid load-bearing brick** | 48.7% | brick 0.99, load-bearing 0.89, almost nothing else | all boroughs; 21% UNSAFE |
| 5 | **unspecified facade ("other")** | 18.7% | low on every family; 70% of reports say "OTHER" | **33% UNSAFE reports** (highest), 64% ever UNSAFE |

(Values are the average shares in each cluster. "UNSAFE" = share of the buildings' initial reports with an UNSAFE result; average over all buildings: 23%.)

The names are our interpretation, based on the profiles and on variables the model never saw (borough, inspection results). "Pre-war" comes from domain knowledge (glazed terra cotta was used mostly before WWII); the data has no construction year.

![Facade families and UNSAFE rate per typology](https://huggingface.co/datasets/iaac-maai/day3-team1-nyc-facade-typologies/resolve/main/dataset_cluster_profiles.png)

## Training data

| | |
|---|---|
| **Dataset** | DOB NOW: Safety – Facades Compliance Filings (NYC Department of Buildings), aggregated to one row per building |
| **Source** | [original (NYC Open Data)](https://data.cityofnewyork.us/d/xubg-57si) · [our clustered version](https://huggingface.co/datasets/iaac-maai/day3-team1-nyc-facade-typologies) |
| **Licence** | NYC Open Data [terms of use](https://www.nyc.gov/home/terms-of-use.page) (free public use) |
| **One row is** | One building (`BIN`), summarising all its initial inspection reports |
| **Size used** | 15,747 buildings (from 52,245 initial reports) |
| **Coverage** | New York City, buildings taller than 6 stories, FISP cycles 6–10 (about 2006–2026), downloaded 30 October 2026 |

**Cleaning and feature engineering:** duplicates and 19,799 `NO REPORT FILED` placeholders removed; owner and inspector contact details dropped; only INITIAL reports with a valid `BIN` and a facade description kept. The 18 wall types and 12 materials were grouped into 13 families, and each building gets the share of its reports that list each family.

## Method and evaluation

- **Methods compared:** K-Means and Ward hierarchical clustering (fitted on a sample of 6,000 buildings because of memory, then extended to all buildings with 5 nearest neighbours). They agree only partly (**ARI = 0.59**): both find the curtain-wall group and the terra cotta group, but they cut the large mass of brick buildings differently.
- **Choice of k:** the elbow curve has no sharp bend, but the silhouette has a **clear peak at k = 6 (0.38)**, against 0.37 at k = 5 and 0.35 at k = 7. Six groups can also be named.
- **Stability** (ARI against the final clusters):

| Check | Mean ARI | Min ARI |
|---|---|---|
| 10 different random seeds | 0.99 | 0.97 |
| 10 random subsamples of 80% of the buildings | 0.99 | 0.98 |
| Ward hierarchical clustering, k = 6 | 0.59 | – |

- **Why this model:** K-Means has a better silhouette than Ward (0.38 vs 0.24), it is very stable, its centroids read as "typical facades", and it can assign **new** buildings with `.predict()`.

## Limitations and biases

- **Based on inspectors' descriptions.** The inputs are what inspectors ticked in their reports, not a survey of the building. The same facade can be described differently from one cycle to the next.
- **"OTHER" is common.** 19% of the buildings fall into the "unspecified facade" group. This group is defined by **missing information**, not by a construction type. Its high UNSAFE rate may come from the facades themselves or from poorer reporting, and we cannot tell which.
- **Always one of six.** K-Means has no "I don't know": an unusual facade is still assigned to the nearest typology.
- **Moderate separation.** With a silhouette of 0.38 and only partial agreement with Ward, the borders between the brick groups (load-bearing, cavity wall, concrete frame) are a modelling choice.
- **Missing information:** no age, height or size, which also shape a facade.
- **Coverage:** NYC, over 6 stories, FISP cycles 6–10 only.

## How to use

```python
import joblib
import pandas as pd
from huggingface_hub import hf_hub_download

path = hf_hub_download(repo_id="iaac-maai/day3-team1-facade-typologies", filename="model.joblib")
model = joblib.load(path)

CLUSTER_NAMES = {
    0: "glass & metal curtain wall",
    1: "brick cavity wall",
    2: "concrete frame",
    3: "terra cotta & brick (pre-war ornament)",
    4: "solid load-bearing brick",
    5: 'unspecified facade ("other")',
}
FEATURES = [
    "MAT_BRICK", "MAT_TERRA_COTTA", "MAT_STONE", "MAT_CONCRETE", "MAT_STUCCO_EIFS", "MAT_GLASS", "MAT_METAL",
    "WALL_MASONRY_LOAD_BEARING", "WALL_MASONRY_CAVITY", "WALL_STEEL_FRAME", "WALL_CONCRETE_FRAME",
    "WALL_CURTAIN_WALL", "WALL_TERRA_COTTA_CLAD",
]

# One building: a glass and metal office tower on a steel frame (1 = listed in the report, 0 = not listed)
example = pd.DataFrame([dict.fromkeys(FEATURES, 0.0)])
example[["MAT_GLASS", "MAT_METAL", "WALL_CURTAIN_WALL", "WALL_STEEL_FRAME"]] = 1.0

cluster = model.predict(example)[0]
print(cluster, CLUSTER_NAMES[cluster])   # 0 glass & metal curtain wall
```

⚠️ `.joblib` files are Python pickles and can execute code when loaded: only load models from sources you trust. Use the library versions listed above; the model may not load with other versions.

## Reproducibility

The full analysis (download, cleaning, feature engineering, choice of k, stability, interpretation, export) is in the GitHub repository: https://github.com/iaac-maai/group1_day3_assignment. Run `notebook/unsupervised_learning.ipynb` with the packages in `requirements.txt`.

## Authors and licence

Amelia, Jerome, Kazuko, Sahil. Course assignment for **AI Methods I: Machine Learning for the Built Environment**, IAAC MaAI 2026/27.

Model released under the MIT License. It was fitted on data derived from *DOB NOW: Safety – Facades Compliance Filings* by the NYC Department of Buildings, published on NYC Open Data under its [terms of use](https://www.nyc.gov/home/terms-of-use.page).
