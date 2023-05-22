import itertools

def generate_answer_cases(questions):
    answer_cases = []
    for question in questions:
        options = question['options']
        question_type = question['type']
        
        if question_type == 'yes_no' or question_type == 'single_choice':
            answer_cases.append(options)
        
        if question_type == 'multi_choice':
            for r in range(1, len(options) + 1):
                combinations = itertools.combinations(options, r)
                answer_cases.extend(combinations)
    
    return list(itertools.product(*answer_cases))
