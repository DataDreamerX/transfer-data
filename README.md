import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Sample PySpark DataFrame
data = [("Category A", 1.2), ("Category B", 3.5), ("Category A", 2.0), 
        ("Category B", 4.1), ("Category A", 3.0), ("Category C", 1.5)]
df = spark.createDataFrame(data, ["category", "query_latency"])

# Convert PySpark DataFrame to Pandas
pdf = df.toPandas()

# Function to show highest or lowest latency
def show_latency(pdf, latency_type="highest"):
    if latency_type == "highest":
        # Get category with the highest average latency
        max_latency_category = pdf.groupby('category')['query_latency'].mean().idxmax()
        max_latency = pdf.groupby('category')['query_latency'].mean().max()
        print(f"Category with the highest latency: {max_latency_category} (Latency: {max_latency:.2f} seconds)")
    elif latency_type == "lowest":
        # Get category with the lowest average latency
        min_latency_category = pdf.groupby('category')['query_latency'].mean().idxmin()
        min_latency = pdf.groupby('category')['query_latency'].mean().min()
        print(f"Category with the lowest latency: {min_latency_category} (Latency: {min_latency:.2f} seconds)")

    # Plot boxplot
    plt.figure(figsize=(10, 6))
    sns.boxplot(x="category", y="query_latency", data=pdf)
    
    # Highlight the max/min latency category in the plot
    if latency_type == "highest":
        plt.title(f"Category with Highest Latency: {max_latency_category}")
        sns.boxplot(x="category", y="query_latency", data=pdf[pdf["category"] == max_latency_category], color='red')
    elif latency_type == "lowest":
        plt.title(f"Category with Lowest Latency: {min_latency_category}")
        sns.boxplot(x="category", y="query_latency", data=pdf[pdf["category"] == min_latency_category], color='green')

    plt.xlabel("Category")
    plt.ylabel("Query Latency (seconds)")
    plt.show()

# Example usage: Get user input for highest or lowest latency
latency_type = input("Enter 'highest' or 'lowest' to see the corresponding category latency: ").strip().lower()

# Call function with user input
show_latency(pdf, latency_type)
