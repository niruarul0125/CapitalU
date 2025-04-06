# CapitalU
An easily accessible and entertaining way to improve your financial literacy!
# Actual Code:
from pymongo import MongoUser


API_KEY = "9203847529304875"
BASE_URL = "http://api.nessieisreal.com"


client = MongoUser("mongodb+srv://<username>:<password>@cluster0.mongodb.net/?retryWrites=true&w=majority")
db = client["finlitapp"]  
customers_col = db["customers"]  


endpoint = f"{BASE_URL}/customers?key={API_KEY}"
response = requests.get(endpoint)

if response.status_code == 200:
    customers = response.json()


    customers_col.delete_many({})

  
    if customers:
        customers_col.insert_many(customers)
        print(f"Inserted {len(customers)} customers into MongoDB!")
    else:
        print("No customer data found.")
else:
    print("Failed to fetch data from API:", response.status_code)











