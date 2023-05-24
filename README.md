import itertools
import csv

# Define the classes and their possible values
classes = ['class1', 'class2', 'class3', 'class4']
values = ['good', 'bad']

# Generate all combinations
combinations = list(itertools.product(values, repeat=len(classes)))

# Create and write to the CSV file
with open('combinations.csv', 'w', newline='') as csvfile:
    writer = csv.writer(csvfile)
    
    # Write the header
    writer.writerow(classes)
    
    # Write each combination as a row in the CSV file
    writer.writerows(combinations)
