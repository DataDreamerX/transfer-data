# Python3 program to implement
# the above approach
# Import the following modules

# pip install matplotlib
import matplotlib.pyplot as plt 

# pip install wordcloud
from wordcloud import WordCloud, STOPWORDS 
import numpy as np
from PIL import Image

# Function for changing the color of the text
def one_color_func(word = None, font_size = None, 
				position = None, orientation = None, 
				font_path = None, random_state = None):

# This HSL is for the green color
	h = 99
	s = 62
	l = 45
	return "hsl({}, {}%, {}%)".format(h, s, l)

# Give the whole path of the text file, 
# open it, read it, and encode it.
text = open(r'C:\Users\Dell\Desktop\Text.txt',
			mode = 'r', encoding = 'utf-8').read() 

# For changing the fonts of wordcloud fonts
path = r'C:\Users\Dell\Downloads\Garbage\Candy Beans.otf'

# The Image shape in which you wanna convert it to.
mask = np.array(Image.open(
				r'C:\Users\Dell\Downloads\Garbage\GFG!.png'))

# Now inside the WordCloud, provide some functions:
# stopwords - For stopping the unuseful words 
# like [,?/\"]
# font_path - provide the font path to which
# you wanna convert it to.
# max_words - Maximum number of words in 
# the output image.
# Also provide height and width of the mask
wc = WordCloud(stopwords = STOPWORDS, 
			font_path = path,
			mask = mask, 
			background_color = "white",
			max_words = 2000, 
			max_font_size = 500,
			random_state = 42, 
			width = mask.shape[1],
			height = mask.shape[0], 
			color_func = one_color_func)

# Finally generate the wordcloud of 
# the given text
wc.generate(text) 
plt.imshow(wc, interpolation = "None")

# Off the x and y axis
plt.axis('off')

# Now show the output cloud
plt.show()
