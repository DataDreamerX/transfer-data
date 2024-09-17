import requests
from bs4 import BeautifulSoup
from lxml import etree

# Function to extract form-related XPaths from a URL
def extract_form_xpaths_from_url(url):
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

    # Find all form elements
    forms = tree.xpath('//form')

    form_related_xpaths = []

    for form in forms:
        # Get the form's XPath
        form_xpath = get_xpath(form)
        form_related_xpaths.append(form_xpath)

        # Find all inputs, buttons, and other interactive elements within the form
        form_elements = form.xpath('.//input | .//button | .//select | .//textarea | .//label')
        
        # Extract XPath for each element inside the form
        for element in form_elements:
            element_xpath = get_xpath(element)
            form_related_xpaths.append(element_xpath)

    return form_related_xpaths

# Example usage
url = 'https://example.com'  # Replace with your URL
form_xpaths = extract_form_xpaths_from_url(url)

# Print the extracted XPaths related to forms
for xpath in form_xpaths:
    print(xpath)
