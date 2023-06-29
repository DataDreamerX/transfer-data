import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score

def generate_similarity_matrix(num_items):
    # Generate a random similarity matrix
    similarity_matrix = np.random.rand(num_items, num_items)
    np.fill_diagonal(similarity_matrix, 1.0)  # Set diagonal elements to 1 for self-similarity

    return similarity_matrix

def group_similar_items(similarity_matrix):
    # Perform K-means clustering for different numbers of clusters
    silhouette_scores = []
    clusters_range = range(2, len(similarity_matrix))
    for num_clusters in clusters_range:
        kmeans = KMeans(n_clusters=num_clusters)
        kmeans.fit(similarity_matrix)
        labels = kmeans.labels_
        silhouette_scores.append(silhouette_score(similarity_matrix, labels))

    # Find the optimal number of clusters based on silhouette scores
    optimal_num_clusters = clusters_range[np.argmax(silhouette_scores)]

    # Perform K-means clustering with the optimal number of clusters
    kmeans = KMeans(n_clusters=optimal_num_clusters)
    kmeans.fit(similarity_matrix)

    # Retrieve cluster labels for each item
    labels = kmeans.labels_

    # Group items based on cluster labels
    groups = {}
    for i, label in enumerate(labels):
        if label not in groups:
            groups[label] = []
        groups[label].append(i)

    return groups

# Generate a random similarity matrix with 6 items
num_items = 6
similarity_matrix = generate_similarity_matrix(num_items)

# Group similar items
result = group_similar_items(similarity_matrix)

# Print the groups
for group_id, items in result.items():
    print(f"Group {group_id + 1}: {items}")

# Plot silhouette scores
plt.plot(range(2, num_items), silhouette_scores)
plt.title('Silhouette Analysis')
plt.xlabel('Number of Clusters')
plt.ylabel('Silhouette Score')
plt.show()
