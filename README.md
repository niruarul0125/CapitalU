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

#app.py
from flask import Flask, jsonify, request
from pymongo import MongoClient
from dotenv import load_dotenv
import os
from bson import ObjectId

app = Flask(__name__)

# Load MongoDB connection
load_dotenv()
client = MongoClient(os.getenv("MONGO_URI"))
db = client["finlit_db"]

@app.route('/api/users', methods=['GET', 'POST'])
def users():
    if request.method == 'POST':
        try:
            data = request.json
            result = db.users.insert_one({
                "username": data['username'],
                "email": data['email'],
                "created_at": datetime.utcnow()
            })
            return jsonify({
                "id": str(result.inserted_id),
                "message": "User created successfully!"
            }), 201
        except Exception as e:
            return jsonify({"error": str(e)}), 500

    elif request.method == 'GET':
        users = list(db.users.find())
        for user in users:
            user['_id'] = str(user['_id'])
        return jsonify(users), 200

@app.route('/api/videos', methods=['GET', 'POST'])
def videos():
    if request.method == 'POST':
        try:
            data = request.json
            result = db.videos.insert_one({
                "title": data['title'],
                "url": data['url'],
                "creator": data['creator'],
                "tags": data['tags'],
                "created_at": datetime.utcnow()
            })
            return jsonify({
                "id": str(result.inserted_id),
                "message": "Video added successfully!"
            }), 201
        except Exception as e:
            return jsonify({"error": str(e)}), 500

    elif request.method == 'GET':
        videos = list(db.videos.find())
        for video in videos:
            video['_id'] = str(video['_id'])
        return jsonify(videos), 200

@app.route('/')
def home():
    return jsonify({
        "message": "Financial Literacy API",
        "endpoints": {
            "users": {
                "POST": "/api/users",
                "GET": "/api/users"
            },
            "videos": {
                "POST": "/api/videos",
                "GET": "/api/videos"
            }
        }
    }), 200

if __name__ == '__main__':
    print("\n✅ API READY AT: http://localhost:5000")
    print("Available Endpoints:")
    print("GET  /               - API Documentation")
    print("POST /api/users      - Create user")
    print("GET  /api/users      - List all users")
    print("POST /api/videos     - Add video")
    print("GET  /api/videos     - List all videos\n")
    app.run(debug=True)

#user
from datetime import datetime

class User:
    def __init__(self, username, email, age, financial_goals=None):
        self.username = username
        self.email = email
        self.age = age
        self.financial_goals = financial_goals or []
        self.created_at = datetime.utcnow()
        self.watched_videos = []
        self.bank_info = {}
    
    def to_dict(self):
        return {
            "username": self.username,
            "email": self.email,
            "age": self.age,
            "financial_goals": self.financial_goals,
            "created_at": self.created_at,
            "watched_videos": self.watched_videos,
            "bank_info": self.bank_info
        }

#video
from datetime import datetime

class Video:
    def __init__(self, title, url, creator, tags, difficulty_level):
        self.title = title
        self.url = url
        self.creator = creator
        self.tags = tags  # e.g., ["budgeting", "saving", "investing"]
        self.difficulty_level = difficulty_level  # "beginner", "intermediate", "advanced"
        self.created_at = datetime.utcnow()
        
    def to_dict(self):
        return {
            "title": self.title,
            "url": self.url,
            "creator": self.creator,
            "tags": self.tags,
            "difficulty_level": self.difficulty_level,
            "created_at": self.created_at
        }

#FRONTEND

import { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [data, setData] = useState([]);

  useEffect(() => {
    // Replace "/api/data" with your backend endpoint (e.g., "/users")
    axios.get('http://localhost:5000/api/users')
      .then(response => console.log('niranjana'))
      //setData(response.data))
      .catch(error => console.error('Error fetching data:', error));
  }, []);

  return (
    <div>
      <h1>Data from Backend:</h1>
      <ul>
        {data.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;

#connectingToMongoDB

from flask import Blueprint, request, jsonify, current_app
from bson import ObjectId
#from flask_cors import CORS




user_bp = Blueprint('user', __name__)
#CORS(user_bp, origins=["http://localhost:5174"], methods=["GET", "POST"], allow_headers=["Content-Type"])

@user_bp.route('/users', methods=['POST'])
def create_user():
    try:
        db = current_app.mongo_db
        data = request.json
        
        # Required fields validation
        required = ['username', 'email']
        if not all(field in data for field in required):
            return jsonify({"error": "Missing required fields"}), 400
            
        # Insert user
        result = db.users.insert_one({
            "username": data['username'],
            "email": data['email']
        })
        
        return jsonify({
            "message": "User created",
            "id": str(result.inserted_id)
        }), 201
        
    except Exception as e:
        return jsonify({"error": str(e)}), 500

#PostManAPI Domain
// http://localhost:5000/api/users 
//http://localhost:5000/api/videos





