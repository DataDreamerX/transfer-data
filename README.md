import requests
from bs4 import BeautifulSoup
from lxml import etree

# Function to extract all XPaths from a URL
def extract_xpaths_from_url(url):
    # Fetch the page content
    response = requests.get(url)
    
    # Parse the page with lxml etree
    parser = etree.HTMLParser()
    tree = etree.fromstring(response.content, parser)

    # Function to build the XPath for each element
    def get_xpath(element):
        path_parts = []
        while element is not None:
            parent = element.getparent()
            if parent is None:
                break
            siblings = parent.xpath('*')
            index = siblings.index(element) + 1
            path_parts.append(f"{element.tag}[{index}]")
            element = parent
        return '/html/' + '/'.join(path_parts[::-1])

    # Extract all elements and their corresponding XPaths
    all_xpaths = []
    for element in tree.xpath('//*'):  # Select all elements
        xpath = get_xpath(element)
        all_xpaths.append(xpath)

    return all_xpaths

# Example usage
url = 'https://example.com'  # Replace with your URL
xpaths = extract_xpaths_from_url(url)

# Print the extracted XPaths
for xpath in xpaths:
    print(xpath)
