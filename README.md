# MongoDB Assignment


### 1. Batch Create with minimum 100 records in MongoDb (create batch).

#### Code (Q1):

```python

from pymongo import MongoClient
import random
from datetime import datetime
import faker
from pymongo import IndexModel, ASCENDING
from pymongo.errors import PyMongoError
from dotenv import load_dotenv
import os

load_dotenv()

class EmployeeDataGenerator:
    def __init__(self):
        mongo_uri = os.getenv("MONGO_URI")
        self.client = MongoClient(mongo_uri)
        self.db = self.client['employeeDB']
        self.collection = self.db['employees']

        self.fake = faker.Faker('en_IN')

        self.departments = [
            'Sales', 'Marketing', 'Engineering', 
            'Human Resources', 'Finance', 'Customer Support', 
            'Product Management', 'Design', 'Operations'
        ]

        self.skills = [
            'communication', 'leadership', 'problem-solving', 
            'teamwork', 'data analysis', 'project management', 
            'coding', 'design', 'marketing', 'sales strategy'
        ]

    def generate_salary(self, level):
        """Generate more realistic salary based on levels"""
        salary_ranges = {
            'Entry Level': (50000, 100000),
            'Mid Level': (100000, 250000),
            'Senior Level': (250000, 500000),
            'Executive Level': (500000, 1000000)
        }

        min_salary, max_salary = salary_ranges[level]
        return random.randint(min_salary, max_salary)

    def generate_employee(self, index, null_status=False):
        levels = ['Entry Level', 'Mid Level', 'Senior Level', 'Executive Level']
        level = levels[index % len(levels)]
        
        salary = self.generate_salary(level)

        employee_doc = {
            'employeeId': f'EMP-{str(index + 1).zfill(3)}',
            'name': self.fake.name(),
            'department': self.departments[index % len(self.departments)], 
            'salary': salary,
            'skills': random.sample(self.skills, 3),
            'contact': {
                'email': f'employee{(index % 10) + 1}@company.com', 
                'phone': self.fake.phone_number()
            },
            'performance': {
                'rating': round(random.uniform(1, 5), 2)
            },
            'metadata': {
                'createdAt': datetime.now(),
                'level': level
            }
        }

        if not null_status:
            employee_doc['status'] = ['active', 'inactive'][index % 2]

        return employee_doc

    def insert_employees_batch(self, count=100, null_status_count=10, batch_size=50):
        # self.collection.delete_many({})  

        regular_employees = [
            self.generate_employee(index) 
            for index in range(count - null_status_count)
        ]

        null_status_employees = [
            self.generate_employee(index + count - null_status_count, null_status=True)
            for index in range(null_status_count)
        ]

        all_employees = regular_employees + null_status_employees
        random.shuffle(all_employees)

        for i in range(0, len(all_employees), batch_size):
            batch = all_employees[i:i + batch_size]
            
            try:
                result = self.collection.insert_many(batch)
                print(f"Inserted batch starting from index {i}")
            except Exception as e:
                print(f"Error in batch starting from index {i}: {e}")

def main():
    generator = EmployeeDataGenerator()
    generator.insert_employees_batch(count=100, null_status_count=10)

if __name__ == "__main__":
    main()


'''
index : db.employees.createIndex({"department":1,"salary":-1,"performance.rating":1})
'''
```

![Description of Image](Output/Q1_MongoDB_1.png)
### 2. Batch Update with minimum 100 records  in MongoDB (update batch).

#### Code (Q2):


```javascript
db.employees.updateMany({"status": null}, {$set: {"status": "active"}})
```
![Description of Image](Output/Q2_MongoDB_1.png)

### 3. Perform indexing on particular 3 fields in MongoDB.
#### Code (Q3):

Create an index on the `department`, `salary`, and `performance.rating` fields.

```javascript
db.employees.createIndex({"department": 1, "salary": -1, "performance.rating": 1})
```

![Description of Image](Output/Q3_MongoDB.png)
### 4. Find duplicates using aggregation in MongoDB

#### Code (Q4):

```javascript
db.employees.aggregate([
    { 
        "$group": { 
            "_id": {"department": "$department"},
            "count": { "$sum": 1 }
        } 
    },
    { 
        "$match": { 
            "count": { "$gt": 1 } 
        } 
    }
])
```
![Description of Image](Output/Q4_MongoDB.png)

![Description of Image](Output/Q4_MongoDB_1.png)
---
