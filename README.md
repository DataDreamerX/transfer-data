import pytest

def test_validation_api():
    # Load the test instructions from the JSON file
    with open('test_cases.json', 'r') as file:
        test_cases = json.load(file)

    api_endpoint = "https://your-api-endpoint/validate"

    def validate_instructions_via_api(instructions):
        response = requests.post(api_endpoint, json={"instructions": instructions})
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"API request failed with status code {response.status_code}")

    for idx, case in enumerate(test_cases):
        instructions = case["instructions"]
        expected_is_valid = case["isValid"]
        expected_reason = case["reason"]
        
        validation_result = validate_instructions_via_api(instructions)
        
        actual_is_valid = validation_result.get("isValid")
        actual_reason = validation_result.get("reason")
        
        assert actual_is_valid == expected_is_valid, f"Test case {idx+1} failed: isValid mismatch.\nExpected: {expected_is_valid}\nActual: {actual_is_valid}"
        assert actual_reason == expected_reason, f"Test case {idx+1} failed: reason mismatch.\nExpected: '{expected_reason}'\nActual: '{actual_reason}'"

# Uncomment the following line to run pytest directly within the script
# pytest.main()
