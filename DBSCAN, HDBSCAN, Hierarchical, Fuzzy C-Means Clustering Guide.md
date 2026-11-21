# Clustering Guide: DBSCAN, HDBSCAN, Hierarchical, Fuzzy C-Means

Nov 21, 2026 · @pathan

Clustering groups similar data points without labels, and these four algorithms differ in how they define a "group".

## The big picture

Clustering means grouping similar data points together without labels: no right answer is given, and the algorithm finds the groups itself. The four algorithms below differ in what they treat as a group.

| Algorithm | A group is... |
| --- | --- |
| DBSCAN | a crowded (dense) area |
| HDBSCAN | a dense area, even when densities differ |
| Hierarchical | a branch in a family tree |
| Fuzzy C-Means | a soft group where a point can belong to several |

## 1. DBSCAN (density-based clustering)

Points packed closely together form a cluster, and lonely points are noise.

**Two settings**

- `eps`: how close counts as a neighbor
- `minPts`: how many neighbors make an area crowded

**Three point types**

- Core: has enough neighbors
- Border: near a core point, but not crowded itself
- Noise: alone, belongs to nobody

**Analogy:** a party. People standing in tight crowds are groups, and someone alone in the corner belongs to no group.

**Example:** GPS locations of accidents. Dense spots become accident hotspots, and random single accidents are noise.

- Good: finds any shape (moons, rings), detects outliers, no need to give `k`
- Weak: fails when clusters have different densities, and is sensitive to `eps`

## 2. HDBSCAN (hierarchical DBSCAN)

HDBSCAN is DBSCAN with its biggest problem fixed: instead of choosing one `eps`, it tries all `eps` values and keeps the most stable clusters. The main setting is `min_cluster_size`.

**Analogy:** DBSCAN uses one zoom level on a map. HDBSCAN zooms in and out and picks the groups that look clear at many zoom levels, so a big dense city and a small sparse village both get found.

**Example:** customer locations where Dhaka is very crowded and small towns are sparse. DBSCAN would either merge Dhaka into one blob or call the towns noise, while HDBSCAN handles both.

- Good: handles different densities, finds noise, needs less tuning than DBSCAN
- Weak: slower, and still not great in very high dimensions

## 3. Hierarchical clustering

It builds a tree (a dendrogram): start with every point alone, keep merging the two closest groups until one remains, then cut the tree at any height to get your clusters.

**Analogy:** a family tree. Siblings join into a family, families into a clan, clans into a tribe, and you decide which level you want.

**Example:** grouping animal species by DNA similarity. Cat and lion merge first, then join the big cats, then mammals.

**Linkage** is how distance between groups is measured, and Ward is the most common default.

- Good: no need to choose `k` first, gives the full tree view, great for small data
- Weak: slow and memory-heavy on big data, merges cannot be undone, no noise handling

## 4. Fuzzy C-Means (FCM)

Normal clustering says a point belongs to exactly one group. FCM says a point belongs 30% to A and 70% to B, and a point's memberships always add up to 100%.

**Settings:** `c` (number of clusters) and `m` (fuzziness, usually 2).

**Analogy:** a movie can be 60% Action and 40% Comedy, so it does not have to pick only one genre.

**Example:** a customer who buys both sports and tech items gets membership in both segments. Tissue boundaries in medical images are blurry, so FCM suits them.

- Good: handles overlapping groups, gives probability-like scores
- Weak: you must choose `c`, assumes round clusters, sensitive to outliers, no noise concept

## Master comparison

Only Fuzzy C-Means gives overlapping membership, only DBSCAN and HDBSCAN reliably detect noise, and only HDBSCAN copes well with mixed densities.

| Feature | DBSCAN | HDBSCAN | Hierarchical | Fuzzy C-Means |
| --- | --- | --- | --- | --- |
| Need number of clusters? | No | No | No (cut the tree later) | Yes |
| Detects noise/outliers? | Yes | Yes | No | No |
| Arbitrary shapes? | Yes | Yes | Depends on linkage | No (round) |
| Different densities? | No | Yes | Partly | No |
| Overlapping membership? | No | No | No | Yes |
| Big data? | Good | OK | Poor | OK |
| Gives a tree? | No | Yes (internally) | Yes | No |

## How to pick the right algorithm

Ask these questions in order and stop at the first match.

1. Can one point belong to multiple groups? Use Fuzzy C-Means.
2. Do I want a tree or hierarchy (a taxonomy)? Use Hierarchical.
3. Are there outliers and odd shapes? Use DBSCAN if the density is the same everywhere, and HDBSCAN if densities differ.

**Keyword trick**

| If the problem says... | Pick |
| --- | --- |
| noise, outliers, arbitrary shape, hotspots | DBSCAN |
| varying density, unknown clusters, messy real-world data | HDBSCAN |
| dendrogram, taxonomy, family tree, small dataset | Hierarchical |
| overlap, membership degree, probability, belongs partially | Fuzzy C-Means |

## Practice scenarios

Try to pick the algorithm yourself first, then check the answer and the reason.

| # | Scenario | Answer | Why |
| --- | --- | --- | --- |
| 1 | A police department wants crime hotspots on a map. Some crimes are isolated, and the number of hotspots is unknown. | DBSCAN | Finds density hotspots, treats isolated crimes as noise, needs no `k`. |
| 2 | E-commerce customers in a huge dense city and sparse rural villages; find regions and ignore random outliers. | HDBSCAN | Densities differ a lot, which breaks DBSCAN. |
| 3 | A biologist has 50 species and wants to show how they are related as a tree. | Hierarchical | The tree is the goal, and the data is small. |
| 4 | A music app groups songs, but many are mixed genres (pop-rock, hip-hop-jazz). | Fuzzy C-Means | Songs can belong to several genres at once. |
| 5 | Data looks like two interlocking moon shapes. | DBSCAN (or HDBSCAN) | Round-cluster methods, including FCM, would cut the moons wrongly. |
| 6 | 200 rows, and you do not know whether to make 3 or 5 groups. | Hierarchical | Cut the dendrogram at different heights and compare. |
| 7 | 1 million GPS points with clusters of different sizes and densities. | HDBSCAN | Hierarchical is too slow, and DBSCAN cannot handle the density differences. |
| 8 | MRI brain scan where tissue boundaries are blurry and gradual. | Fuzzy C-Means | Pixels near boundaries belong partly to both tissues. |

## Common confusions cleared

- **DBSCAN vs HDBSCAN:** HDBSCAN is DBSCAN plus "try all `eps` values". Use HDBSCAN when densities differ.
- **Hierarchical vs HDBSCAN:** both involve a hierarchy, but Hierarchical clusters everything with no noise, while HDBSCAN is density-based and labels noise.
- **Fuzzy C-Means vs K-Means:** K-Means is hard (one group per point), FCM is soft (percentages).
- **Does DBSCAN need `k`?** No. It needs `eps` and `minPts`, and the number of clusters comes out as a result.

## One-line memory summary

- DBSCAN = crowds + noise
- HDBSCAN = crowds of different sizes + noise
- Hierarchical = family tree
- Fuzzy C-Means = mixed colors (partial belonging)
