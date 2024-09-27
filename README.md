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

# Add plot title and adjust layout
plt.title("Query Latency and Result Count Over 5-Minute Intervals by Category", fontsize=14)
plt.tight_layout()

# Show the plot
plt.show()
