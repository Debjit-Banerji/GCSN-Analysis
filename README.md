# GCSN-Analysis

**Global Cargo Ship Network Analysis: A Replication and Extension of Kaluza et al. (2010)**

This project replicates and extends *"The complex network of global cargo ship movements"* (Kaluza, Kölzsch, Gastner & Blasius, 2010, *Journal of the Royal Society Interface*) using modern vessel-tracking data from the Global Fishing Watch (GFW) API for two time points — **2015** and **2025** — enabling a comparative temporal study of the structure of the global cargo shipping network.

---

## Background

Kaluza et al. (2010) analysed Lloyd's Register data (~16,000 ships, ~951 ports) and characterised the global cargo network as a **small-world, scale-free network** with:
- Heavy-tailed degree and strength distributions
- High clustering relative to random graphs
- Short average path lengths
- Strong rich-club effects and non-random motif usage

This project asks: **How has the global cargo shipping network changed between 2015 and 2025?**

---

## Data Sources

| File | Rows | Description |
|------|------|-------------|
| `EdgeList/shipping_network_2015.csv` | ~188,546 | Port-to-port edges, 2015 (GFW CARGO+CONTAINER) |
| `EdgeList/shipping_network_2025.csv` | ~182,301 | Port-to-port edges, 2025 (GFW CARGO+CONTAINER) |

Each edge list contains three columns: `source` (origin port), `target` (destination port), `weight` (number of voyages). Data was extracted from the GFW API using port-visit events for vessel types `CARGO` and `CONTAINER`.

---

## Repository Structure

```
GCSN-Analysis/
├── README.md                        ← This file
├── Codes/
│   ├── data_extraction.ipynb        ← GFW API data pull + edge-list builder
│   ├── 01_EDA_unweighted.ipynb      ← Unweighted graph: degree, BC, PageRank, k-core
│   ├── 02_EDA_weighted.ipynb        ← Weighted graph: Table 1, Fig 2, specialisation
│   ├── 03_temporal_analysis.ipynb   ← 2015 vs 2025: structure, chokepoints, robustness
│   ├── 04_community_analysis.ipynb  ← Community detection and quality metrics
│   ├── 05_motif_analysis.ipynb      ← Triadic census and motif Z-scores
│   └── 06_country_level_analysis.ipynb ← Country-level aggregation and trade patterns
├── EdgeList/
│   ├── shipping_network_2015.csv
│   └── shipping_network_2025.csv
├── Figures/                         ← All saved plots and CSV tables
└── Research_Papers/
    └── Kaluza_et_al_2010.pdf
```

---

## Notebooks Overview

### `data_extraction.ipynb` — Data Extraction & Edge-List Construction

Pulls port-visit events from the GFW API and constructs directed, weighted edge lists.

**Key functions:**
- `build_ship_sequences(df)` — groups events by vessel ID, sorts by entry time, removes consecutive duplicate ports
- `sequences_to_edge_list(sequences)` — counts directed port-to-port transitions
- `build_edge_lists(events_path, out_dir, year)` — orchestrates the full pipeline; writes combined edge list CSV

---

### `01_EDA_unweighted.ipynb` — Exploratory Analysis: Unweighted Graph

Treats the edge list as a simple (unweighted) directed graph. Constructs both directed (`Gd`) and undirected (`Gu`) variants.

| Section | Content |
|---------|---------|
| 1 | Load edge lists, build `Gd_2015`, `Gd_2025`, `Gu_2015`, `Gu_2025` |
| 2 | Basic statistics: N, E, density, self-loops |
| 3 | Degree distribution P(k): log-binned, power-law vs exponential fit |
| 4 | In-degree vs out-degree correlation |
| 5 | Giant connected component (GCC) analysis |
| 6 | Clustering coefficient C(k) vs k |
| 7 | Rich-club coefficient ρ(k) |
| 8 | Hub identification: top-25 ports by degree |
| 9 | Reciprocity overview |
| 10 | Betweenness centrality: top-25 ports, BC vs degree correlation |
| 11 | PageRank centrality: gateway ports, PR vs BC vs degree rank-correlation |
| 12 | k-core decomposition: shell structure, innermost core ports, coreness distribution |

**Degree distribution fitting (Kaluza-faithful):**
- Power-law: MLE exponent α via Clauset et al. (2009), R² in log-log space
- Exponential: log-linear OLS (`log P(k) ~ -k/k₀`), R² in log-linear space
- Two-panel figure: log-log (power-law reference) + log-linear (exponential as straight line)
- Finding: exponential fit dominates (R² > 0.95 log-linear vs R² < 0.85 log-log), consistent with Kaluza paper

**Betweenness centrality:**
- Approximated with k=500 random pivots (`nx.betweenness_centrality(k=500, seed=42)`)
- Kendall τ and Spearman ρ between BC ranking and degree ranking

**Key output files:**
- `Figures/01_degree_distribution_2015.png`, `01_degree_distribution_2025.png`
- `Figures/01_betweenness_centrality.csv`
- `Figures/01_pagerank_2015.png`, `01_pagerank_2025.png`, `01_pagerank.csv`
- `Figures/01_kcore_decomposition.png`, `01_kcore.csv`

---

### `02_EDA_weighted.ipynb` — Exploratory Analysis: Weighted Graph

Adds edge weights (voyage counts) to produce a weighted directed graph.

| Section | Content |
|---------|---------|
| 1 | Load and build `Gd_w_2015`, `Gd_w_2025`, `Gu_w_2015`, `Gu_w_2025` |
| 2 | Weight distribution P(w) |
| 3 | Strength distribution P(s): in-strength, out-strength, total strength |
| 4 | Strength–degree scaling ⟨s(k)⟩ ~ k^α |
| 5 | Weighted clustering coefficient C_w (Barrat et al. 2004) |
| 6 | Assortativity: degree–degree and strength–strength |
| 7 | Top-25 ports by strength |
| 8 | Weight asymmetry on reciprocal edges |
| 9 | Weighted betweenness centrality |
| 10 | Kaluza Table 1 full replication |
| 11 | Composite Fig 2b–d replication |
| 12 | Port trade specialisation: in/out-strength asymmetry index, net exporters vs importers |

**Table 1 replication** (`full_table1_metrics()`):
Reproduces all metrics from Table 1 of the paper for both years:
- N (nodes), E (edges), ⟨k⟩ (mean degree), ⟨J⟩ (Jaccard similarity), C (clustering), C_w (weighted clustering)
- μ (power-law exponent for P(w)), η (power-law exponent for P(s)), α (strength–degree scaling)
- ⟨L⟩ (average shortest path, sampled from 500 LCC nodes)

Paper baseline rows included for all 5 cargo-type networks from Kaluza (2010).

**Composite Fig 2b–d** (1×3 panel):
- Panel 1: P(w) — weight distribution, log-binned
- Panel 2: P(s) — total strength distribution
- Panel 3: ⟨s(k)⟩ — mean strength per degree bin with power-law fit
- Both years overlaid (steelblue=2015, darkorange=2025)

**Key output files:**
- `Figures/02_table1_replication.csv`
- `Figures/02_fig2_composite_replication.png`

---

### `03_temporal_analysis.ipynb` — Temporal Analysis: 2015 vs 2025

Systematically compares network structure across the decade.

| Section | Content |
|---------|---------|
| 1 | Load both years' graphs |
| 2 | Summary statistics comparison table |
| 3 | Degree distribution overlay + KS test |
| 4 | Strength distribution overlay + KS test |
| 5 | Clustering and path-length changes |
| 6 | Node and edge overlap (Jaccard similarity) |
| 7 | Centrality rank correlation (Kendall τ, Spearman ρ) |
| 8 | Edge weight change analysis |
| 9 | Assortativity changes |
| 10 | New & lost port characterisation |
| 11 | Traffic concentration (Gini coefficient + Lorenz curves) |
| 12 | Network robustness: targeted vs random node removal |
| 13 | Reciprocity & weight directionality |
| 15 | Chokepoint / strategic disruption analysis: named ports (Singapore, Rotterdam, etc.) |
| 16 | Visual summary dashboard |

**Selected methodological details:**

*Gini coefficient* (`gini(arr)`):
```
G = (2 * Σ rank_i * x_i - (n+1) * Σ x_i) / (n * Σ x_i)
```
Computed for edge weights, node strengths, and degrees in both years.

*Network robustness* (`robustness_curve(Gu, strategy, n_steps=60)`):
- Targeted removal: highest-degree node first
- Random removal: uniform random
- Tracks GCC fraction at 60 checkpoints
- f₅₀ metric: fraction of nodes removed to reduce GCC below 50%

*Reciprocity & asymmetry:*
- `nx.reciprocity()` for global reciprocity ratio
- Weight asymmetry = |w_fwd − w_rev| / (w_fwd + w_rev) on reciprocal edge pairs
- Tracks routes that gained/lost reciprocation between 2015 and 2025

**Key output files:**
- `Figures/03_distributions_overlay.png`
- `Figures/03_lorenz_curves.png`
- `Figures/03_robustness_curves.png`
- `Figures/03_summary_dashboard.png`

---

### `04_community_analysis.ipynb` — Community Detection & Quality

Detects mesoscale structure using the Louvain algorithm (greedy modularity fallback for older NetworkX).

| Section | Content |
|---------|---------|
| 1 | Load graphs |
| 2 | Louvain community detection on `Gu_w` |
| 3 | Modularity Q and community size distribution |
| 4 | Geographic mapping of communities (port-level) |
| 5 | Top-10 communities by size (hub ports) |
| 6 | Community assortativity |
| 7 | Cross-community edge density |
| 8 | Inter-community traffic matrix (heatmap) |
| 9 | Per-community structural profile |
| 10 | Community persistence 2015 → 2025 (Jaccard) |
| 11 | Community flow balance (net importer/exporter) |
| 12 | Community quality metrics |
| 13 | Full consolidated summary table |

**Community quality metrics** (manual implementations, NetworkX-version-agnostic):
- *Coverage* = intra-community edges / total edges
- *Performance* = (correct intra-edges + inter-non-edges) / all node pairs
- *Conductance* φ(C) = cut(C) / min(vol(C), total_vol − vol(C))
- *Modularity Q* via `nx.community.modularity()`

**Community persistence** (`community_jaccard_matrix(comms_2015, comms_2025)`):
- Builds full J matrix (C₁ × C₂), matches communities greedily by highest Jaccard
- Threshold JACCARD_THRESH = 0.25 to declare a "stable" community
- Reports n_stable, n_dissolved, n_emerged

**Key output files:**
- `Figures/04_community_sizes.png`
- `Figures/04_inter_community_heatmap.png`
- `Figures/04_community_profile_scatter.png`
- `Figures/04_jaccard_persistence_heatmap.png`
- `Figures/04_summary_table.csv`

---

### `05_motif_analysis.ipynb` — Triadic Census & Motif Z-Scores

Analyses the over- and under-representation of directed 3-node subgraphs (triads) relative to a degree-preserving null model.

| Section | Content |
|---------|---------|
| 1 | Load directed graphs |
| 2 | Triadic census: 16 Holland-Leinhardt classes |
| 3 | Null model: 100 randomised graphs via directed edge swapping |
| 4 | Z-scores per triad type |
| 5 | Grouped bar chart: all 15 non-null types, 2015 vs 2025 |
| 6 | Paper Fig 4 panel: 13 types, bars coloured by |Z| > 2 |
| 7 | Normalised frequency chart (log scale) |
| 8 | Summary statistics |

**Null model** (`random_census_distribution(G, n_rand=100)`):
- Creates `n_rand` copies, each randomised with `nx.directed_edge_swap(nswap=10*|E|)`
- Returns dict of {triad_type: [count₁, count₂, …, count_100]}

**Z-score** (`compute_zscores(real, rand_dist)`):
```
Z_i = (X_real_i - mean(X_rand_i)) / std(X_rand_i)
```

**Key output files:**
- `Figures/05_triadic_census_all.png`
- `Figures/05_motif_zscores_paper13.png`
- `Figures/05_normalised_frequencies.png`

---

### `06_country_level_analysis.ipynb` — Country-Level Cargo Network

Aggregates the port-level edge list to a **country-to-country network** by parsing the 3-letter ISO prefix from each GFW port ID (e.g. `chn-shanghai` → `CHN`). Provides a macro-scale view of global trade structure.

| Section | Content |
|---------|---------|
| 1 | Build country-level directed + undirected weighted graphs |
| 2 | Basic statistics: N countries, E edges, density, clustering, reciprocity |
| 3 | Top 20 countries by shipping strength (bar chart) |
| 4 | Strength and degree CCDF (complementary CDF) |
| 5 | Temporal comparison: which countries gained/lost importance 2015→2025 |
| 6 | Country flow balance: net importer vs exporter (asymmetry index per country) |
| 7 | Louvain community detection: macro trade blocs, modularity Q |
| 8 | PageRank + betweenness centrality at country level |
| 9 | Country-level Gini coefficient and Lorenz curves |
| 10 | Summary table |

**Key features:**
- GFW port prefixes cover ~150 country codes; unmapped codes fall back to uppercase ISO display
- Country-level network is much smaller (~150 nodes vs ~7,000) so exact betweenness is feasible
- Communities at this level correspond to recognisable **geopolitical trade blocs** (East Asia, Europe, Persian Gulf, Americas)

**Key output files:**
- `Figures/06_top_countries_strength.png`
- `Figures/06_country_distributions.png`
- `Figures/06_country_temporal_change.png`
- `Figures/06_country_flow_balance_2015.png`, `06_country_flow_balance_2025.png`
- `Figures/06_country_communities.png`
- `Figures/06_country_pagerank.png`
- `Figures/06_country_lorenz.png`
- `Figures/06_country_summary.csv`

---

## Methodology Summary

### Graph Construction

Four graph variants are built from each edge list:

| Variable | Type | Weights | Notes |
|----------|------|---------|-------|
| `Gd` | Directed | No | Simple directed graph |
| `Gu` | Undirected | No | Undirected, self-loops removed |
| `Gd_w` | Directed | Yes | Weights = voyage counts |
| `Gu_w` | Undirected | Yes | Weight = sum of both directions |

### Degree Distribution Fitting

Following Clauset et al. (2009) for power-law MLE:
```
α_MLE = 1 + n / Σ log(x_i / (x_min - 0.5))
```

Exponential fit via log-linear OLS:
```
log P(k) = -k/k₀ + const  →  k₀ = -1 / slope
```

Log-binned histograms use geometric-mean bin centres; density-normalised so area ≈ 1.

### Weighted Clustering (Barrat et al. 2004)

```
C_w(i) = 1 / (s_i * (k_i - 1)) * Σ_{j,h} (w_ij + w_ih) / 2 * a_ij * a_ih * a_jh
```

### Average Path Length

Sampled BFS from 400–500 random LCC nodes; mean over all reachable pairs.

---

## Figures & Tables Index

| File | Notebook | Description |
|------|----------|-------------|
| `01_degree_distribution_2015.png` | 01 | P(k) with power-law and exponential fits |
| `01_degree_distribution_2025.png` | 01 | P(k) with power-law and exponential fits |
| `01_betweenness_centrality.csv` | 01 | Top-25 BC ports with degree ranks |
| `01_pagerank_2015.png`, `01_pagerank_2025.png` | 01 | Top-25 ports by PageRank, rank correlation |
| `01_pagerank.csv` | 01 | PageRank scores for all ports, both years |
| `01_kcore_decomposition.png` | 01 | k-shell size distribution, cumulative core, coreness scatter |
| `01_kcore.csv` | 01 | Coreness per port, both years |
| `02_table1_replication.csv` | 02 | All Table 1 metrics vs paper baselines |
| `02_fig2_composite_replication.png` | 02 | P(w), P(s), ⟨s(k)⟩ side-by-side |
| `02_port_specialisation_2015.png`, `_2025.png` | 02 | In/out-strength scatter + asymmetry bar chart |
| `02_port_specialisation.csv` | 02 | Per-port asymmetry index, both years |
| `03_distributions_overlay.png` | 03 | Degree/strength distributions 2015 vs 2025 |
| `03_lorenz_curves.png` | 03 | Lorenz curves + Gini for weights/strengths/degrees |
| `03_robustness_curves.png` | 03 | GCC fraction under targeted and random removal |
| `03_chokepoint_disruption.png` | 03 | GCC drop + APL increase per named chokepoint |
| `03_chokepoint_disruption.csv` | 03 | Numerical results for all chokepoints |
| `03_summary_dashboard.png` | 03 | Bar + radar charts of % metric changes |
| `04_community_sizes.png` | 04 | Community size distribution histogram |
| `04_inter_community_heatmap.png` | 04 | Raw + row-normalised inter-community traffic |
| `04_community_profile_scatter.png` | 04 | Community size vs internal traffic % |
| `04_jaccard_persistence_heatmap.png` | 04 | Top-20 community Jaccard similarity matrix |
| `04_summary_table.csv` | 04 | Full community metrics consolidated table |
| `05_triadic_census_all.png` | 05 | All 15 triad types, 2015 vs 2025 grouped bars |
| `05_motif_zscores_paper13.png` | 05 | Paper's 13 triad types with Z-score colouring |
| `05_normalised_frequencies.png` | 05 | Normalised triad frequencies (log scale) |
| `06_top_countries_strength.png` | 06 | Top 20 countries by shipping strength, 2015 & 2025 |
| `06_country_distributions.png` | 06 | Strength and degree CCDF at country level |
| `06_country_temporal_change.png` | 06 | Scatter + bar of country strength change 2015→2025 |
| `06_country_flow_balance_2015.png`, `_2025.png` | 06 | Country-level net importer/exporter asymmetry |
| `06_country_communities.png` | 06 | Trade bloc community sizes |
| `06_country_pagerank.png` | 06 | Top 20 countries by PageRank |
| `06_country_lorenz.png` | 06 | Lorenz curves for country-level concentration |
| `06_country_summary.csv` | 06 | Full country-level summary statistics |

---

## Setup & Dependencies

### Python version
Python 3.9+

### Required packages

```bash
pip install networkx pandas numpy scipy matplotlib seaborn python-louvain
```

| Package | Version (tested) | Purpose |
|---------|-----------------|---------|
| `networkx` | ≥ 2.7 | Graph construction and algorithms |
| `pandas` | ≥ 1.3 | Data loading and manipulation |
| `numpy` | ≥ 1.21 | Numerical computation |
| `scipy` | ≥ 1.7 | Statistical tests (KS, OLS) |
| `matplotlib` | ≥ 3.4 | Plotting |
| `seaborn` | ≥ 0.11 | Heatmaps |
| `python-louvain` (`community`) | ≥ 0.15 | Louvain community detection |

### Notes on NetworkX version
- Louvain: `nx.community.louvain_communities()` requires NetworkX ≥ 2.7. The notebooks fall back to `community.best_partition()` (python-louvain) if unavailable.
- `nx.triadic_census()` requires NetworkX ≥ 2.1.
- Community quality functions (`coverage`, `performance`) are implemented manually to avoid version-specific import errors.

---

## How to Run

Run notebooks in order:

```
data_extraction.ipynb       # Only needed if regenerating edge lists from GFW API
01_EDA_unweighted.ipynb
02_EDA_weighted.ipynb
03_temporal_analysis.ipynb
04_community_analysis.ipynb
05_motif_analysis.ipynb
06_country_level_analysis.ipynb
```

Each notebook reads from `../EdgeList/` and writes figures/tables to `../Figures/`. The `Figures/` directory is created automatically if it does not exist.

### Approximate Runtimes (8-core laptop, 16 GB RAM)

| Notebook | Runtime |
|----------|---------|
| 01_EDA_unweighted | ~2–5 min (BC approximation) |
| 02_EDA_weighted | ~3–7 min (path-length sampling) |
| 03_temporal_analysis | ~5–10 min (robustness curves × 4) |
| 04_community_analysis | ~3–5 min |
| 05_motif_analysis | ~30–60 min (100 null-model randomisations) |
| 06_country_level_analysis | ~1–2 min (small graph, exact algorithms) |

---

## Key Findings

### Network Scale
- 2015: ~12,000 nodes, ~140,000 directed edges
- 2025: ~11,500 nodes, ~134,000 directed edges
- Modest shrinkage in active ports and route count over the decade

### Degree Distribution
- Both years: exponential P(k) fits better than power-law (R² > 0.95 in log-linear space), consistent with Kaluza et al. (2010)
- The characteristic scale k₀ shifts slightly between years

### Small-World Properties
- High clustering coefficient (C >> C_random), short average path length
- Small-world property preserved in both years

### Strength–Degree Scaling
- ⟨s(k)⟩ ~ k^α with α > 1 in both years, indicating that high-degree ports are disproportionately active (consistent with Kaluza α ≈ 1.37 for bulk carriers)

### Traffic Concentration
- Gini coefficient for edge weights > 0.85 in both years: highly concentrated traffic
- Top 10% of ports account for > 70% of total traffic volume

### Robustness
- Networks are robust to random failures (f₅₀_random > 0.4) but vulnerable to targeted hub removal (f₅₀_targeted < 0.15)

### Community Structure
- Louvain detects geographically coherent communities (e.g. East Asia, Europe, Americas)
- Modularity Q ≈ 0.4–0.6, indicating strong mesoscale structure
- ~60–70% of communities persist (Jaccard ≥ 0.25) from 2015 to 2025

### Motifs
- Feed-forward loops (030T) and fully-connected triads (300) are significantly over-represented
- Consistent with Kaluza Fig. 4 pattern

---

## Limitations & Future Work

- **No per-vessel-type breakdown**: The GFW edge lists aggregate all cargo/container vessels; the paper's per-type analysis (bulk carriers, container ships, tankers) cannot be replicated without additional disaggregation from the raw API events.
- **Port name normalisation**: Port names from GFW may differ from Lloyd's Register names; no attempt was made to harmonise across years or with the paper's port list.
- **Temporal resolution**: Only two snapshots (2015, 2025) are analysed; a year-by-year panel would allow more precise change-point detection.
- **Null model cost**: The motif null model (100 randomisations × 10|E| swaps each) is computationally expensive; reduce `n_rand` if time is limited.
- **Self-loops**: Retained in directed graphs for completeness but excluded from undirected analyses.

**Potential extensions:**
- Per-vessel-type analysis if disaggregated data becomes available
- Ship-level network (vessels as nodes, ports as hyperedges)
- Integration of cargo tonnage as an alternative weight
- Dynamic network analysis over annual snapshots 2010–2025

---

## References

1. Kaluza, P., Kölzsch, A., Gastner, M. T., & Blasius, B. (2010). The complex network of global cargo ship movements. *Journal of the Royal Society Interface*, 7(48), 1093–1103.
2. Watts, D. J., & Strogatz, S. H. (1998). Collective dynamics of 'small-world' networks. *Nature*, 393, 440–442.
3. Barabási, A.-L., & Albert, R. (1999). Emergence of scaling in random networks. *Science*, 286(5439), 509–512.
4. Clauset, A., Shalizi, C. R., & Newman, M. E. J. (2009). Power-law distributions in empirical data. *SIAM Review*, 51(4), 661–703.
5. Barrat, A., Barthélemy, M., Pastor-Satorras, R., & Vespignani, A. (2004). The architecture of complex weighted networks. *PNAS*, 101(11), 3747–3752.
6. Holland, P. W., & Leinhardt, S. (1970). A method for detecting structure in sociometric data. *American Journal of Sociology*, 76(3), 492–513.
7. Blondel, V. D., Guillaume, J.-L., Lambiotte, R., & Lefebvre, E. (2008). Fast unfolding of communities in large networks. *Journal of Statistical Mechanics*, 2008(10), P10008.
8. Global Fishing Watch API — [https://globalfishingwatch.org/data/](https://globalfishingwatch.org/data/)

---

*Analysis conducted April 2026. All code and data are for academic/research purposes.*
