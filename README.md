# Clustering Mini Projects: Country Data

Two clustering mini projects on the same dataset of 167 countries:

1. **Apply & Compare Clustering Algorithms:** K-Means, DBSCAN and Hierarchical Clustering, compared on cluster shape, number of clusters, sensitivity to noise and performance.
2. **Evaluating Clustering Before and After PCA:** K-Means on the original 9 features vs. on 2 PCA components.

The goal is to group countries by socio-economic and health indicators, for example to identify which countries need aid the most.

---

## Dataset

**[Unsupervised Learning on Country Data](https://www.kaggle.com/datasets/rohan0301/unsupervised-learning-on-country-data)** (Kaggle): 167 countries, 9 numerical features, no missing values.

| Feature | Description |
|---|---|
| `child_mort` | Deaths of children under 5 per 1,000 live births |
| `exports` | Exports of goods and services (% of GDP per capita) |
| `health` | Total health spending (% of GDP per capita) |
| `imports` | Imports of goods and services (% of GDP per capita) |
| `income` | Net income per person |
| `inflation` | Annual growth rate of total GDP |
| `life_expec` | Average life expectancy at birth (years) |
| `total_fer` | Number of children born per woman |
| `gdpp` | GDP per capita |

The project briefs suggested the Mall Customers and Wine datasets, but both allow using your own dataset. We used the Country data for both projects so the results can be compared directly.

---

## Workflow

1. **Data preparation:** checked shape, types and missing values (none). Dropped `country` (identifier only) and standardized all 9 features with `StandardScaler`.
2. **K-Means:** chose K with the Elbow Method and the Silhouette Score.
3. **DBSCAN:** chose `eps` with a k-distance plot, then grid-searched `eps` × `min_samples`.
4. **Hierarchical Clustering:** compared 4 linkage methods (Ward, complete, average, single) for k = 2–5, used a dendrogram to choose the cut, and tested stability under noise.
5. **Comparison:** compared the algorithms on cluster sizes, noise points and silhouette, side by side in PCA space.
6. **PCA:** checked the correlations and the explained variance, reduced to 2 components, re-ran K-Means and compared the results before and after PCA.

---

## Results

### Project 1: Comparing the algorithms

| Algorithm | Settings | Clusters | Noise points | Cluster sizes | Silhouette |
|---|---|---|---|---|---|
| **K-Means** | K = 3 | 3 | 0 | 36 / 47 / 84 | **0.2833** |
| **DBSCAN** | eps = 1.3, min_samples = 6 | 2 | 48 (28.7%) | 24 / 95 | 0.4131\* |
| **Hierarchical** | Ward, 3 clusters (cut at 18.5) | 3 | 0 | 34 / 106 / 27 | 0.2456 |

\* DBSCAN's silhouette is computed without the 48 noise points, so it isn't a fair comparison with the methods that cluster all 167 countries.

**K-Means** gave three clear development levels:
- **Developed:** child mortality ~5, income ~45,700, life expectancy ~80
- **Developing:** child mortality ~22, income ~12,300, life expectancy ~73
- **Under-developed:** child mortality ~93, income ~3,900, life expectancy ~59

**DBSCAN:**
- With the eps from the k-distance plot (2.4), it found only **1 cluster + 8 outliers**. The countries change gradually from poor to rich, with no empty gaps between groups.
- With a smaller eps, it merged the rich and middle-income countries into one cluster and labeled 29% of countries as noise.
- Its real value here was **outlier detection**: it flagged Luxembourg, Singapore, Qatar and Nigeria, among others.

**Hierarchical Clustering:**
- **Linkage choice:** average and single linkage had the highest silhouette (0.63), but only by isolating one outlier (166 / 1). Complete linkage scored 0.29, but one of its clusters had only 3 countries. **Ward** was the only method with balanced clusters.
- **Agreement with K-Means:** the two methods agree on **83%** of countries (139 / 167) and never disagree by more than one level.
  - Hierarchical puts oil states (Saudi Arabia, Oman, Libya) in Developed, while K-Means puts advanced non-oil economies there instead (South Korea, Czech Republic, Slovenia).
  - K-Means labels 20 more countries Under-developed.
- **Stability:** with 5% random noise added (20 runs), Ward kept a mean ARI of **0.54**. The overall structure holds, but borderline countries move between clusters.

**Best algorithm: K-Means.** It had the highest silhouette among the methods that cluster all 167 countries, and it gave three clear, interpretable groups.

### Project 2: K-Means before vs. after PCA

2 components keep **63.1%** of the variance. PC1 reflects development level (child mortality, income, life expectancy, fertility), and PC2 reflects trade volume (exports, imports).

| Metric | Before PCA (9 features) | After PCA (2 PCs) |
|---|---|---|
| Variance kept | 100% | 63.1% |
| Cluster sizes | 36 / 47 / 84 | 3 / 72 / 92 |
| Silhouette (original scaled space) | 0.2833 | 0.2951 |
| Silhouette (PCA space) | n/a | 0.4416\* |
| ARI vs. before PCA | n/a | 0.3178 |
| Countries that changed cluster | n/a | 58 / 167 |

\* Computed in a different space, so it isn't comparable with the score before PCA.

- **How did PCA affect the clustering?** It changed it a lot: 58 countries moved, and PC2 isolated 3 countries with extreme trade volume (Luxembourg, Malta, Singapore).
- **Did the visualization become clearer?** Yes. All countries fit in one 2D plot, though that plot shows only 63% of the information.
- **Did we lose important information?** Yes. The 36.9% of variance that was dropped changed the cluster structure.
- **Which approach is better?** The original 9 features give a more faithful and balanced clustering. PCA with 2 components is better for visualization only.

**Bonus (different numbers of components):** with **4 components** (87.2% variance), the clusters almost match the original (ARI 0.96). With **5 or more**, they're identical (ARI 1.0).

---

## Key Insights

- This dataset has **no natural, well-separated groups**. Countries lie on a continuous range from poor to rich, which is why all meaningful silhouette scores are below 0.3.
- K-Means and Hierarchical (Ward) find the **same three levels** (developed / developing / under-developed) and differ only at the borders. Those borderline countries should be reviewed individually in any aid decision.
- **Silhouette alone is misleading:** methods that isolate a few outliers score high, so cluster sizes must always be checked.
- **Oil wealth** strongly affects cluster membership. Gulf states rank as developed, while some oil exporters (Nigeria, Angola, Equatorial Guinea) rank as developing despite very high child mortality.
- **PCA** is useful for plotting, but 2 components lose enough information to change the clusters. At least 4 components are needed to keep the original structure.

---

## How to Run

1. Open `Mini_Project_2_3.ipynb` in [Google Colab](https://colab.research.google.com/) or Jupyter.
2. Install the requirements (already available on Colab):
   ```bash
   pip install pandas numpy matplotlib scikit-learn scipy kagglehub
   ```
3. Run all cells. The dataset downloads automatically with `kagglehub`:
   ```python
   path = kagglehub.dataset_download("rohan0301/unsupervised-learning-on-country-data")
   ```

All models use `random_state=42`, so the results are reproducible.

---

## Repository Structure

```
├── Mini_Project_2_3.ipynb   # Full notebook: code, visualizations, explanations
└── README.md
```

---

## Team

| Member | Part |
|---|---|
| [Raghad Alasmari](https://github.com/Raghad0za) | Hierarchical Clustering |
| [Daniyah Aleid](https://github.com/DAleid) | PCA Clustering |
| Member 3 | |
| Member 4 | |
