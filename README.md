import random
from faker import Faker

faker = Faker()

def generate_fake_records(num_customers):
    records = []

    for _ in range(num_customers):
        customer = {
            'name': faker.name(),
            'email': faker.email(),
            'answers': []
        }
        for _ in range(10):
            question_type = random.choice(['single_choice', 'multi_choice'])
            question = {
                'type': question_type,
                'question': faker.sentence(),
                'answer': []
            }
            if question_type == 'single_choice':
                options = [faker.word() for _ in range(random.randint(2, 4))]
                answer = random.choice(options)
                question['options'] = options
                question['answer'] = answer
            elif question_type == 'multi_choice':
                options = [faker.word() for _ in range(random.randint(3, 6))]
                num_choices = random.randint(1, min(len(options), 3))
                answer = random.sample(options, num_choices)
                question['options'] = options
                question['answer'] = answer
            customer['answers'].append(question)
        
        records.append(customer)
    
    return records

fake_records = generate_fake_records(10000)
