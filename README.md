import requests
from bs4 import BeautifulSoup
from lxml import etree

# Function to extract form-related XPaths, labels, and IDs from a URL
def extract_form_elements_from_url(url):
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

    form_elements_data = []

    for form in forms:
        # Get all form fields (input, select, textarea, button) within the form
        form_elements = form.xpath('.//input | .//button | .//select | .//textarea')

        for element in form_elements:
            element_data = {}

            # Get the XPath
            element_data['xpath'] = get_xpath(element)

            # Get the ID (if any)
            element_id = element.get('id')
            element_data['id'] = element_id if element_id else None

            # Try to find the associated label (either by `for` attribute or wrapping label)
            label_text = None

            # Find label with a `for` attribute matching the element's `id`
            if element_id:
                label = tree.xpath(f'//label[@for="{element_id}"]')
                if label:
                    label_text = label[0].text.strip()

            # If no `for` label, check if the element is wrapped in a label
            if not label_text:
                parent_label = element.getparent()
                if parent_label.tag == 'label' and parent_label.text:
                    label_text = parent_label.text.strip()

            # Store the label text (if any)
            element_data['label'] = label_text if label_text else None

            # Append the element data to the list
            form_elements_data.append(element_data)

    return form_elements_data

# Example usage
url = 'https://example.com'  # Replace with your URL
form_elements = extract_form_elements_from_url(url)

# Print the extracted XPaths, IDs, and labels
for element in form_elements:
    print(f"XPath: {element['xpath']}")
    print(f"ID: {element['id']}")
    print(f"Label: {element['label']}")
    print('-' * 40)
