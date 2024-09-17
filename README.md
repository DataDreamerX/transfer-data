from playwright.sync_api import sync_playwright

def get_form_elements(page):
    """
    Extract form elements grouped by their containing form.
    """
    # Select all forms
    forms = page.query_selector_all('form')
    grouped_elements = []

    for form in forms:
        form_id = form.get_attribute('id') or 'no-id'
        form_name = form.get_attribute('name') or 'no-name'

        # Get all relevant elements within the form
        elements = form.query_selector_all("input[type='text'], input[type='password'], input[type='email'], input[type='submit'], input[type='button'], button, select")

        form_elements = []
        
        for element in elements:
            try:
                type_attr = element.get_attribute('type')
                class_attr = element.get_attribute('class')
                id_attr = element.get_attribute('id')
                
                # Extract button text if it's a button element
                text = ""
                if element.tag_name == 'button' or (type_attr in ['submit', 'button']):
                    text = element.inner_text()

                # Collect element information
                form_elements.append({
                    "locator": element,
                    "type": type_attr,
                    "class": class_attr,
                    "id": id_attr,
                    "text": text
                })
            except Exception as e:
                print(f"Error processing element: {e}")

        if form_elements:
            grouped_elements.append({
                "form_id": form_id,
                "form_name": form_name,
                "elements": form_elements
            })

    return grouped_elements

with sync_playwright() as p:
    browser = p.chromium.launch(headless=False)
    page = browser.new_page()
    page.goto('https://example.com/login')  # Replace with your login page URL

    # Wait for the page to fully load
    page.wait_for_load_state('load')

    grouped_form_elements = get_form_elements(page)
    
    for form in grouped_form_elements:
        print(f"Form ID: {form['form_id']}, Form Name: {form['form_name']}")
        for elem in form['elements']:
            print(elem)
    
    browser.close()
