K-Means Clustering is the process of grouping similar data points into 
clusters without needing labeled data. It is used to uncover hidden 
patterns when the goal is to organised data based on similarity.

- Helps identify natural groupings in unlabeled datasets
- Works by grouping points based on distance to cluster centers
- Commonly used in customer segmentation, image compression and pattern discovery
- Useful when you need structure from raw, unorganized data

## How K-Means Works

Suppose we have a dataset of items with certain features represented as 
vectors. The task is to categorize those items into `k` groups or clusters 
of similarity.

### The Algorithm

1. **Initialization**: Randomly select `k` cluster centroids from the dataset
2. **Assignment Step**: Each data point is assigned to its nearest centroid
3. **Update Step**: Recalculate each centroid as the mean of all points assigned to it
4. **Repeat**: Continue until centroids stop moving or `max_iter` is reached

### Why Random Initialization is a Problem

Because we pick starting centroids randomly, the final result can vary 
between runs. To solve this, we run the algorithm multiple times and keep 
the best result. "Best" means lowest **inertia** — the total distance from 
each point to its assigned centroid:

```
inertia = sum of distances from each point to its centroid
```

The lower the inertia, the tighter and better defined the clusters are.

## Distance Metrics

Distance measures how similar two points are. Two common options:

**L1 (Manhattan)** — sum of absolute differences, simplest and fastest:

```
distance = |h1-h2| + |w1-w2| + |b1-b2|
```

**L2 (Euclidean)** — straight line distance through space:

```
distance = sqrt((h1-h2)² + (w1-w2)² + (b1-b2)²)
```

For K-Means, L1 and L2 usually give similar results. L1 is preferred 
when you want simplicity and speed.

## Our Dataset

The `solar_system_census.csv` dataset has three features per person:

| Feature | Description |
|---|---|
| height | Height in cm |
| weight | Weight in kg |
| bone_density | Bone density value |

Each person is a point in 3D space. We want to find 4 clusters 
corresponding to the 4 regions of the solar system.

## Implementation

### Step 1 — Initialize centroids

Randomly pick `ncentroid` rows from the dataset as starting centroids:

```python
indices = np.random.choice(len(X), self.ncentroid, replace=False)
self.centroids = X[indices]
```

`np.random.choice` picks random indices without replacement, and numpy 
fancy indexing `X[indices]` grabs those rows.

### Step 2 — Assign each point to nearest centroid

For each point, calculate L1 distance to every centroid and find the closest:

```python
for point in X:
    distances = [np.sum(np.abs(point - c)) for c in self.centroids]
    assignments.append(np.argmin(distances))
```

`np.argmin` returns the **index** of the smallest distance, not the value 
itself — so it tells us which centroid the point belongs to.

### Step 3 — Update centroids

After assigning all points, recalculate each centroid as the mean of its 
assigned points — the "gravity center" of the cluster:

```python
for i in range(self.ncentroid):
    points_in_cluster = X[assignments == i]
    if len(points_in_cluster) > 0:
        self.centroids[i] = points_in_cluster.mean(axis=0)
```

`X[assignments == i]` uses fancy indexing to grab all points assigned to 
centroid `i`. `.mean(axis=0)` averages each feature column independently.

### Step 4 — Run multiple times, keep best result

```python
best_inertia = float('inf')
for _ in range(50):
    km = KmeansClustering(max_iter=max_iter, ncentroid=ncentroid)
    km.fit(X)
    predictions = km.predict(X)
    inertia = sum(np.sum(np.abs(X[i] - km.centroids[predictions[i]]))
                  for i in range(len(X)))
    if inertia < best_inertia:
        best_inertia = inertia
        best_centroids = km.centroids.copy()
        best_predictions = predictions.copy()
```

## Interpreting the Output

After fitting with `ncentroid=4` we get 4 centroids. We match them to 
planets using known characteristics:

| Planet | Height | Weight | Bone Density | Notes |
|---|---|---|---|---|
| Venus | ~172cm | medium | highest | Shortest, slender |
| Earth | ~192cm | ~87kg | ~0.80 | Average everything |
| Belt | ~191cm | ~70kg | low | Tall, lightest (low gravity) |
| Mars | ~194cm | ~105kg | lowest | Tallest, heaviest |

### Our Results

Running with `ncentroid=4` and `max_iter=30` across multiple runs gives 
consistent clusters:

```
~172cm, weight=79,  bone=0.94  → 40 people  = Venus
~191cm, weight=70,  bone=0.82  → 25 people  = Belt
~192cm, weight=87,  bone=0.80  → 40 people  = Earth
~194cm, weight=105, bone=0.77  → 14 people  = Mars
```

The Belt citizens are tall with the lowest weight — consistent with low 
gravity environments reducing muscle and fat mass. Venus citizens are the 
shortest with the highest bone density.
