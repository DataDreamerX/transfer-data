[
    {
        "instructions": "1. Navigate to the website. 2. Enter 'testuser' into the input field labeled 'Username'. 3. Click the button with text 'Login'.",
        "isValid": false,
        "reason": "Missing URL for the navigation step."
    },
    {
        "instructions": "1. Navigate to 'https://example.com'. 2. Click the 'Submit' button. 3. Verify that the success message appears.",
        "isValid": false,
        "reason": "Ambiguous element locator for the 'Submit' button; multiple elements may match."
    },
    {
        "instructions": "1. Navigate to 'https://example.com'. 2. Enter 'password123' into the input field labeled 'Password'. 3. The button with text 'Login'.",
        "isValid": false,
        "reason": "Missing action for the 'Login' button in step 3."
    },
    {
        "instructions": "1. Go to 'https://example.com'. 2. Fill the username. 3. Click submit.",
        "isValid": false,
        "reason": "Unclear instruction in step 2; no information provided on how to locate the 'username' field."
    },
    {
        "instructions": "1. Visit 'https://example.com'. 2. Hover over the 'Features' menu. 3. Select 'Pricing' from the dropdown.",
        "isValid": false,
        "reason": "Unsupported action 'Hover' in step 2."
    },
    {
        "instructions": "1. Navigate to 'https://example.com'. 2. Click the 'Login' button.",
        "isValid": false,
        "reason": "Instructions are incomplete; additional steps like entering credentials are likely needed."
    },
    {
        "instructions": "1. Navigate to 'https://example.com'. 2. Enter 'testuser' into the input field labeled 'Username'. 3. Enter 'password123' into the input field labeled 'Password'. 4. Click the 'Login' button.",
        "isValid": true,
        "reason": ""
    },
    {
        "instructions": "1. Open 'https://example.com'. 2. Click the button labeled 'More Info'. 3. Click the button labeled 'More Info' again.",
        "isValid": false,
        "reason": "Ambiguous actions; multiple clicks on the same element without specifying different conditions or waits."
    },
    {
        "instructions": "1. Visit 'https://example.com'. 2. Enter 'testuser' into the input field. 3. Click 'Login'.",
        "isValid": false,
        "reason": "Missing element locator for the 'Login' button in step 3."
    },
    {
        "instructions": "1. Go to 'example_com'. 2. Enter 'testuser' into the input field labeled 'Username'. 3. Click the 'Submit' button.",
        "isValid": false,
        "reason": "Invalid URL format in step 1."
    }
]
