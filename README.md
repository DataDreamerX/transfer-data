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

# Output to CSV file
csv_file = 'fake_records.csv'

# Extract column names
column_names = ['Name', 'Email', 'Question', 'Type', 'Options', 'Answer']

with open(csv_file, 'w', newline='') as file:
    writer = csv.DictWriter(file, fieldnames=column_names)
    writer.writeheader()
    for record in fake_records:
        name = record['name']
        email = record['email']
        answers = record['answers']
        for answer in answers:
            question = answer['question']
            question_type = answer['type']
            options = answer['options']
            answer_value = answer['answer']
            writer.writerow({
                'Name': name,
                'Email': email,
                'Question': question,
                'Type': question_type,
                'Options': options,
                'Answer': answer_value
            })

print(f"Fake records saved to {csv_file} successfully.")
