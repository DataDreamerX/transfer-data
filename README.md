from playwright.sync_api import sync_playwright

def get_element_xpath(element):
    """
    Function to get the XPath of a given element by executing a JS function on the page.
    """
    return element.evaluate('''
        (element) => {
            const getXPath = (element) => {
                if (element.id !== '') {
                    return '//*[@id="' + element.id + '"]';
                }
                if (element === document.body) {
                    return '/html/body';
                }

                let ix = 0;
                const siblings = element.parentNode ? element.parentNode.childNodes : [];
                for (let i = 0; i < siblings.length; i++) {
                    const sibling = siblings[i];
                    if (sibling === element) {
                        return getXPath(element.parentNode) + '/' + element.tagName.toLowerCase() + '[' + (ix + 1) + ']';
                    }
                    if (sibling.nodeType === 1 && sibling.tagName === element.tagName) {
                        ix++;
                    }
                }
            };
            return getXPath(element);
        }
    ''')

with sync_playwright() as p:
    browser = p.chromium.launch(headless=False)
    page = browser.new_page()
    page.goto('https://example.com/login')  # Replace with your login page URL
    
    # Select the element you want to get the XPath for
    element = page.query_selector("input[type='text']")  # Adjust this selector as needed

    if element:
        xpath = get_element_xpath(element)
        print(f"XPath: {xpath}")

    browser.close()
