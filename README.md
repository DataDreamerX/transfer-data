import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from pyspark.sql import SparkSession

# Create a Spark session
spark = SparkSession.builder.appName("Latency and Result Count Visualization").getOrCreate()

# Sample DataFrame with category, latency, result_count, and timestamp
data = [("Category A", 1.2, 50, "2024-09-01 10:01:00"),
        ("Category B", 3.5, 150, "2024-09-01 10:03:00"),
        ("Category A", 2.0, 80, "2024-09-01 10:06:00"),
        ("Category B", 4.1, 200, "2024-09-01 10:11:00"),
        ("Category A", 3.0, 90, "2024-09-01 10:14:00"),
        ("Category C", 1.5, 30, "2024-09-01 10:16:00")]

# Create a DataFrame with category, latency, result count, and timestamp
df = spark.createDataFrame(data, ["category", "query_latency", "result_count", "timestamp"])

# Convert 'timestamp' to actual timestamp type
df = df.withColumn("timestamp", df["timestamp"].cast("timestamp"))

# Convert PySpark DataFrame to Pandas DataFrame for resampling
pdf = df.toPandas()

# Ensure the 'timestamp' is in datetime format in Pandas
pdf['timestamp'] = pd.to_datetime(pdf['timestamp'])

# Resample by 5-minute intervals and aggregate query latency and result count
pdf.set_index('timestamp', inplace=True)
resampled_pdf = pdf.groupby('category').resample('5T').agg({
    'query_latency': 'mean',
    'result_count': 'sum'  # Can also use 'mean' or 'median'
}).reset_index()

# Set up the plot style with a green color palette
sns.set(style="whitegrid")
green_palette = sns.color_palette("Greens_d", len(resampled_pdf['category'].unique()))

# Create a dual-axis plot
fig, ax1 = plt.subplots(figsize=(12, 6))

# Plot latency on the primary y-axis
sns.lineplot(x="timestamp", y="query_latency", hue="category", data=resampled_pdf, marker="o", palette=green_palette, ax=ax1)
ax1.set_ylabel('Query Latency (seconds)', fontsize=12)
ax1.set_xlabel('Time', fontsize=12)
ax1.tick_params(axis='x', rotation=45)

# Create a secondary y-axis to plot result count
ax2 = ax1.twinx()
sns.lineplot(x="timestamp", y="result_count", hue="category", data=resampled_pdf, marker="x", palette=green_palette, ax=ax2, linestyle="--")
ax2.set_ylabel('Result Count', fontsize=12)

# Add legend to the first axis only to avoid duplication
handles, labels = ax1.get_legend_handles_labels()
ax1.legend(handles, labels, title="Categories")

# Add plot title and adjust layout
plt.title("Query Latency and Result Count Over 5-Minute Intervals by Category", fontsize=14)
plt.tight_layout()

# Show the plot
plt.show()
