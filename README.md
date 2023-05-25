instring = "She said, 'I'm happy' but I don't believe her."
words = instring.split()

for i, word in enumerate(words):
    if "'" in word and not word.endswith("'"):
        words[i] = '"' + word.replace("'", "") + '"'

outstring = ' '.join(words)
print(outstring)
