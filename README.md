def jaccard_similarity(str1, str2):
    set1 = set(str1.lower())
    set2 = set(str2.lower())
    
    intersection = set1.intersection(set2)
    union = set1.union(set2)
    
    similarity = len(intersection) / len(union)
    return similarity

# Example strings
string1 = "Hello"
string2 = "Hola"

# Calculate Jaccard similarity
similarity_score = jaccard_similarity(string1, string2)

print(f"Jaccard similarity: {similarity_score}")
