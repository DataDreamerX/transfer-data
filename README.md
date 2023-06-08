import re
import math
from collections import Counter

def preprocess_text(text):
    # Remove punctuation and special characters, convert to lowercase
    text = re.sub(r"[^\w\s]", "", text.lower())
    return text

def calculate_cosine_similarity(short_string, long_string):
    # Tokenize and preprocess the strings
    short_tokens = preprocess_text(short_string).split()
    long_tokens = preprocess_text(long_string).split()

    # Create frequency vectors
    short_vector = Counter(short_tokens)
    long_vector = Counter(long_tokens)

    # Compute dot product
    dot_product = sum(short_vector[token] * long_vector[token] for token in short_vector)

    # Compute magnitudes
    short_magnitude = math.sqrt(sum(short_vector[token]**2 for token in short_vector))
    long_magnitude = math.sqrt(sum(long_vector[token]**2 for token in long_vector))

    # Compute cosine similarity
    cosine_similarity = dot_product / (short_magnitude * long_magnitude)

    return cosine_similarity

# Example usage
short_string = "apple"
long_string = "apples are delicious"
similarity = calculate_cosine_similarity(short_string, long_string)
print(f"Cosine similarity: {similarity}")
