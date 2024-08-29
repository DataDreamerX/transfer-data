# Convert Spark DataFrame to Pandas DataFrame
pandas_df = grouped_df.toPandas()

# Plot a bar chart
plt.figure(figsize=(10, 6))
plt.bar(pandas_df['value'], pandas_df['count'], color='skyblue')
plt.xlabel('Value')
plt.ylabel('Count')
plt.title('Count of Each Value')
plt.show()
