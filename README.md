import json

def add_item_with_highest_accuracy(arr, new_item, n):
    # Convert JSON objects to dictionaries
    arr = [json.loads(item) for item in arr]
    new_item = json.loads(new_item)

    # Sort the array based on "accuracy" in descending order
    arr.sort(key=lambda x: x['accuracy'], reverse=True)

    # Keep only the first "n" items
    arr = arr[:n]

    # Append the new item
    arr.append(new_item)

    return arr

# Example usage
my_array = [
    '{"class": "abc", "accuracy": 0.9}',
    '{"class": "def", "accuracy": 0.8}',
    '{"class": "xyz", "accuracy": 0.95}'
]
new_item = '{"class": "ghi", "accuracy": 0.92}'
n = 2

updated_array = add_item_with_highest_accuracy(my_array, new_item, n)
print(updated_array)
