# Holistic-Student-Support-System-HSSS-
Here’s another implementation for the **Holistic Student Support System (HSSS)** chatbot, this time using a different technology stack with **Flask**, **Dialogflow** for Natural Language Understanding (NLU), and **Firebase** for mood tracking. Dialogflow is a powerful tool for building conversational agents and can handle more complex dialogues.

### 1. **Set Up the Environment**

Install the required dependencies:
```bash
pip install flask google-cloud-dialogflow firebase-admin
```

### 2. **Flask Web Application with Dialogflow and Firebase Integration**

#### `app.py` (Main Flask Application)

```python
from flask import Flask, request, jsonify
import dialogflow_v2 as dialogflow
import os
import firebase_admin
from firebase_admin import credentials, firestore

app = Flask(__name__)

# Set up Dialogflow authentication (use your Google service account key)
os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "path-to-your-dialogflow-service-account-key.json"

# Initialize Firebase Firestore for mood tracking
cred = credentials.Certificate('path-to-your-firebase-adminsdk.json')
firebase_admin.initialize_app(cred)
db = firestore.client()

# Define the Dialogflow session client
dialogflow_session_client = dialogflow.SessionsClient()
PROJECT_ID = "your-dialogflow-project-id"

# Dialogflow chatbot route
@app.route('/chat', methods=['POST'])
def chat():
    user_input = request.json['message']
    session_id = request.json.get('session_id', 'default_session')
    session = dialogflow_session_client.session_path(PROJECT_ID, session_id)
    
    # Send user message to Dialogflow
    text_input = dialogflow.types.TextInput(text=user_input, language_code="en")
    query_input = dialogflow.types.QueryInput(text=text_input)
    response = dialogflow_session_client.detect_intent(session=session, query_input=query_input)

    # Process Dialogflow response
    chatbot_response = response.query_result.fulfillment_text

    # Optional: Log user input and chatbot response to Firestore for mood tracking
    mood_data = {
        "user_message": user_input,
        "bot_response": chatbot_response,
        "timestamp": firestore.SERVER_TIMESTAMP
    }
    db.collection('mood_tracking').add(mood_data)

    return jsonify({'response': chatbot_response})

if __name__ == '__main__':
    app.run(debug=True)
```

### 3. **Dialogflow Setup**
You need to set up a **Dialogflow agent** for this to work:
1. Create an agent in [Dialogflow](https://dialogflow.cloud.google.com/).
2. Create **intents** that represent various user inputs (e.g., anxiety, stress, seeking help).
3. Train the agent with sample phrases and responses to handle different emotional queries.
4. Download the **service account key** for your Dialogflow agent and include it in your project for authentication.

### 4. **Firebase Setup**
1. Create a **Firestore database** on [Firebase](https://firebase.google.com/).
2. Download the **Firebase Admin SDK** credentials and place them in your project directory.
3. Initialize Firestore in the code as shown to log user interactions for future mood analysis.

### 5. **Frontend (HTML for User Interface)**

Here’s a basic frontend (HTML + JavaScript) for interacting with the chatbot:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Holistic Student Support System</title>
    <style>
        body { font-family: Arial, sans-serif; }
        .chatbox { width: 50%; margin: 20px auto; }
        .messages { height: 300px; border: 1px solid #ccc; padding: 10px; overflow-y: scroll; }
        .input-box { display: flex; }
        .input-box input { width: 100%; padding: 10px; border: 1px solid #ccc; }
        .input-box button { padding: 10px; background-color: #4CAF50; color: white; border: none; cursor: pointer; }
    </style>
</head>
<body>

<div class="chatbox">
    <div class="messages" id="messages"></div>
    <div class="input-box">
        <input type="text" id="userInput" placeholder="Type your message here..." />
        <button onclick="sendMessage()">Send</button>
    </div>
</div>

<script>
    function sendMessage() {
        const userInput = document.getElementById('userInput').value;
        const messagesDiv = document.getElementById('messages');
        
        // Display user's message
        messagesDiv.innerHTML += `<p><strong>You:</strong> ${userInput}</p>`;
        
        // Clear the input box
        document.getElementById('userInput').value = '';
        
        // Send user's message to Flask backend
        fetch('/chat', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ message: userInput })
        })
        .then(response => response.json())
        .then(data => {
            // Display the chatbot's response
            messagesDiv.innerHTML += `<p><strong>Bot:</strong> ${data.response}</p>`;
            messagesDiv.scrollTop = messagesDiv.scrollHeight;
        });
    }
</script>

</body>
</html>
```

### 6. **How It Works**
- **Dialogflow** handles natural language understanding, providing more robust conversation handling, including intent matching and context awareness.
- **Firebase Firestore** is used for storing user inputs and chatbot responses to track emotional trends over time, allowing for mood tracking.
- The **Flask server** hosts the chatbot and handles communication between the frontend, Dialogflow, and Firebase.

### 7. **Key Features to Add**
- **Sentiment Analysis**: Enhance the system by integrating sentiment analysis based on the logged conversations in Firestore.
- **Resource Suggestions**: Integrate the chatbot’s response to suggest helpful mental health resources or services based on user emotions.
- **Scalability**: Host the solution on a cloud platform like **Google Cloud** for scalability and resilience.

This solution allows for richer interactions by leveraging Dialogflow's advanced capabilities and ensures that user conversations are stored and analyzed over time for better mental health tracking.
