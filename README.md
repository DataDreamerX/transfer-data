import numpy as np
from sklearn.cluster import KMeans

def generate_similarity_matrix(num_items):
    # Generate a random similarity matrix
    similarity_matrix = np.random.rand(num_items, num_items)
    np.fill_diagonal(similarity_matrix, 1.0)  # Set diagonal elements to 1 for self-similarity

    return similarity_matrix

def group_similar_items(similarity_matrix, num_clusters):
    # Perform K-means clustering
    kmeans = KMeans(n_clusters=num_clusters)
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

# Number of desired groups
num_clusters = 3

# Group similar items
result = group_similar_items(similarity_matrix, num_clusters)

# Print the groups
for group_id, items in result.items():
    print(f"Group {group_id + 1}: {items}")
