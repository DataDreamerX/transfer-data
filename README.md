import requests
from bs4 import BeautifulSoup
from lxml import etree

# Function to extract form-related XPaths and their corresponding labels
def extract_form_xpaths_and_labels_from_url(url):
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

    # Function to find the label for a form element (if exists)
    def find_label(element):
        # Check if the input element has an id and there's a <label> with a matching 'for' attribute
        element_id = element.get("id")
        if element_id:
            label = tree.xpath(f"//label[@for='{element_id}']")
            if label:
                return label[0].text.strip()

        # Otherwise, check if the <label> directly wraps the input element
        parent = element.getparent()
        if parent.tag == "label":
            return parent.text.strip()

        return None

    # Find all form elements
    forms = tree.xpath('//form')

    form_related_info = []

    for form in forms:
        # Get the form's XPath
        form_xpath = get_xpath(form)
        form_related_info.append({
            'element': 'form',
            'xpath': form_xpath,
            'label': 'Form'  # You can customize this if needed
        })

        # Find all inputs, buttons, and other interactive elements within the form
        form_elements = form.xpath('.//input | .//button | .//select | .//textarea | .//label')

        # Extract XPath and label for each element inside the form
        for element in form_elements:
            element_xpath = get_xpath(element)
            label_text = find_label(element)

            form_related_info.append({
                'element': element.tag,
                'xpath': element_xpath,
                'label': label_text if label_text else 'No Label'
            })

    return form_related_info

# Example usage
url = 'https://example.com'  # Replace with your URL
form_info = extract_form_xpaths_and_labels_from_url(url)

# Print the extracted XPaths and labels
for info in form_info:
    print(f"Element: {info['element']}, XPath: {info['xpath']}, Label: {info['label']}")
