import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans

def group_similar_items(similarity_matrix, max_clusters):
    # Calculate within-cluster sum of squares (WCSS) for different numbers of clusters
    wcss = []
    for num_clusters in range(1, max_clusters + 1):
        kmeans = KMeans(n_clusters=num_clusters)
        kmeans.fit(similarity_matrix)
        wcss.append(kmeans.inertia_)

    # Plot the elbow curve
    plt.plot(range(1, max_clusters + 1), wcss)
    plt.title('Elbow Method')
    plt.xlabel('Number of Clusters')
    plt.ylabel('WCSS')
    plt.show()

    # Determine the optimal number of clusters based on the elbow plot
    optimal_num_clusters = int(input("Enter the optimal number of clusters: "))

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

# Example similarity matrix
similarity_matrix = np.array([[1.0, 0.5, 0.7],
                              [1.0, 0.5, 0.7],
                              [1.0, 0.5, 0.7]])

# Maximum number of clusters to consider
max_clusters = 10

# Group similar items
result = group_similar_items(similarity_matrix, max_clusters)

# Print the groups
for group_id, items in result.items():
    print(f"Group {group_id + 1}: {items}")
