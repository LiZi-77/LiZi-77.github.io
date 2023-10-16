# DBSCAN algorithm

Density-Based Spatial Clustering of Applications with Noise, which also called `DBSCAN`, is a data clustering algorithm proposed in 1996, cite(wiki https://en.wikipedia.org/wiki/DBSCAN and also the related paper).

Data clustering is the task of grouping objects in such a way that objects in the same group (called a **cluster**) are more similar (in some sense) to each other than to those in other groups (clusters). cite (https://en.wikipedia.org/wiki/Cluster_analysis). A lot of algorithms has been proposed over the past decades, among them, they have different difinition of `cluster models`, such as `connectivity model`, `centroid model`, `density models`.

DBSCAN algrotithm has the following features:

1. Density model identifies clusters as regions of high data point density separated by regions of lower density. It is particularly effective at identifying clusters of arbitrary shape in spatial data, without requiring the number of clusters to be specified in advance.
2. DBSCAN categorizes data points into three categories. Core points are densely connected to a sufficient number of other points within a specified radius. Border points are sparsely connected but lie within the radius of a core point. Noise points have no sufficient connections to form clusters.
3. DBSCAN uses two parameters to define the clustering behavior. Epsilon (ε) determines the radius within which points are considered neighbors. MinPoints specifies the minimum number of points required to form a dense region (core point).
4. DBSCAN iteratively expands clusters by connecting core points and border points within the specified radius. It identifies connected components of core and border points and assigns them to clusters.
5. DBSCAN handles noise points by labeling them as outliers or noise, denoting data points that do not belong to any specific cluster.

Add the pseudocode for DBSCAN:
`to be added May 23th`

The DBSCAN algorithm was utilized in the event extraction model described in the "first step" paper. Subsequently, in our validation chapter, we employed this algorithm as an example to demonstrate the versatility of our tool when applied to real-world data and tasks. It is important to note that the selection of various algorithms and their corresponding parameters was not the primary focus of our project. Given that the DBSCAN algorithm was mentioned in the related paper, we chose to employ it as a representative example, showcasing the broad applicability of our tool in real-world scenarios.
