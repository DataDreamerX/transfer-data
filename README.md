from playwright.sync_api import sync_playwright

def get_login_form_elements(page):
    # Assuming that forms related to login contain specific keywords in form elements like 'login' or 'password'
    elements = page.query_selector_all("input[type='text'], input[type='password'], input[type='email'], input[type='submit'], input[type='button'], button, select")

    login_elements = []
    
    for element in elements:
        try:
            type_attr = element.get_attribute('type')
            class_attr = element.get_attribute('class')
            id_attr = element.get_attribute('id')
            
            # Filtering elements that likely belong to login forms by checking for keywords
            if any(keyword in (id_attr or '').lower() for keyword in ['login', 'username', 'email', 'password']):
                locator = element
                login_elements.append({
                    "locator": locator,
                    "type": type_attr,
                    "class": class_attr,
                    "id": id_attr
                })
        except Exception as e:
            print(f"Error processing element: {e}")
    
    return login_elements

with sync_playwright() as p:
    browser = p.chromium.launch(headless=False)
    page = browser.new_page()
    page.goto('https://example.com/login')  # Replace with your login page URL
    login_form_elements = get_login_form_elements(page)
    
    for elem in login_form_elements:
        print(elem)
    
    browser.close()
