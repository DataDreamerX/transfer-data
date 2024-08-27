import json
import requests

# Load the test instructions from the JSON file
with open('test_cases.json', 'r') as file:
    test_cases = json.load(file)

# Define the API endpoint (replace with your actual endpoint)
api_endpoint = "https://your-api-endpoint/validate"  # Example URL, replace with your API's URL

# Function to send test instructions to the API and receive validation results
def validate_instructions_via_api(instructions):
    response = requests.post(api_endpoint, json={"instructions": instructions})
    
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Failed to validate: {response.status_code}")
        return None

# Iterate through each test case and validate it via the API
for case in test_cases:
    instructions = case["instructions"]
    expected_is_valid = case["isValid"]
    expected_reason = case["reason"]
    
    # Validate the instructions
    validation_result = validate_instructions_via_api(instructions)
    
    if validation_result:
        # Extract the API's response
        is_valid = validation_result.get("isValid")
        reason = validation_result.get("reason")
        
        # Print out the results
        print(f"Instructions: {instructions}")
        print(f"Expected: isValid={expected_is_valid}, reason='{expected_reason}'")
        print(f"Actual: isValid={is_valid}, reason='{reason}'")
        print("-" * 50)

# Optional: Save the validation results to a file
with open('validation_results_from_api.json', 'w') as f:
    json.dump(test_cases, f, indent=4)
