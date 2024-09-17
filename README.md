from playwright.sync_api import sync_playwright

def get_form_locators(page):
    # Get all input elements
    inputs = page.locator('input')
    # Get all select elements
    selects = page.locator('select')
    # Get all textarea elements
    textareas = page.locator('textarea')
    # Get all button elements
    buttons = page.locator('button')

    # Combine all locators into a list
    form_elements = [inputs, selects, textareas, buttons]

    # Iterate through each locator and print the count of elements found
    for locator in form_elements:
        count = locator.count()
        print(f'Found {count} elements for {locator}')

with sync_playwright() as p:
    browser = p.chromium.launch(headless=False)
    page = browser.new_page()
    page.goto('https://example.com')  # Replace with your target URL
    get_form_locators(page)
    browser.close()
