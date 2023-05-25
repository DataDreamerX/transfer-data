import re

# Define the input string
input_string = "It is important to use contractions when you're writing. We'll try our best, but we can't promise anything."

# Use regex to substitute the words
output_string = re.sub(r"\b(it|you|we|can)'(\s*(s|re|ll|t)\b)", r"\1'\2", input_string)

# Print the output string
print(output_string)
