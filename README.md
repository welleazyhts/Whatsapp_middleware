# WhatsApp AI Bot with OpenAI & Flask 🤖💬

This project is a simple yet powerful WhatsApp bot that listens to incoming messages in real-time, processes them using the OpenAI (ChatGPT) API for intelligent responses, and replies instantly. The bot is built with Flask and can be run locally and exposed to the internet using ngrok for testing.

## ⚙️ How It Works

The message flow is straightforward:

1.  You send a message from your phone to the bot's WhatsApp number.
2.  Meta receives the message and forwards it to your specified Webhook URL.
3.  **ngrok** tunnels the request from the public internet to your local Flask application.
4.  The **Flask app** processes the incoming message payload.
5.  The message content is sent to the **OpenAI API** to generate a smart reply.
6.  The Flask app sends the AI-generated response back to your WhatsApp number via the Meta Graph API.



## ✨ Features

* **Real-time Messaging**: Responds to WhatsApp messages instantly.
* **AI-Powered Responses**: Leverages OpenAI's `gpt-3.5-turbo` for intelligent, human-like replies.
* **Local Development**: Easy to set up and run on your local machine.
* **Simple to Deploy**: Can be easily deployed to services like Render, Railway, or AWS.

---

## 📋 Prerequisites

Before you begin, ensure you have the following set up:

* **Python 3.8+**
* **A Meta Developer Account**: Create one at [developers.facebook.com](https://developers.facebook.com/).
* **An OpenAI Account**: Get your API key from [platform.openai.com/api-keys](https://platform.openai.com/api-keys).
* **ngrok**: A tool to expose your local server to the internet. [Install ngrok here](https://ngrok.com/download).

---

## 🚀 Setup and Installation

### 1. Set Up Your Meta App

1.  Go to the [Meta Developer Portal](https://developers.facebook.com/).
2.  Click **Create App** → Select **Other** → **Business**.
3.  Name your app and create it.
4.  From the app dashboard, find the **WhatsApp** product and click **Set up**.
5.  In the **API Setup** section, you will find your temporary **Test Phone Number**, **Phone Number ID**, and a temporary **Access Token**.
6.  Add your personal phone number as a recipient to test the integration.

### 2. Clone and Set Up the Project

First, clone the repository or create the project structure manually.

```bash
# Create the project directory
mkdir whatsapp-ai-bot
cd whatsapp-ai-bot

# Create a Python virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows, use `venv\Scripts\activate`

# Install the required packages
pip install flask requests python-dotenv openai

```
## 3.Project Structure
### Your project directory should look like this:

```bash 

whatsapp-ai-bot/
│
├── .env
├── run.py
├── whatsapp_utils.py
└── openai_service.py

```
## 4. Configure Environment Variables
### Create a .env file in the root of your project and add the following credentials.

❗️ Important: Never commit your .env file to version control.

Code snippet

```bash  
# A secret token of your choice to verify webhook requests from Meta
VERIFY_TOKEN="your_super_secret_verify_token"

# Your access token from the Meta App -> WhatsApp -> API Setup page
ACCESS_TOKEN="your_long_lived_meta_access_token"

# The Phone Number ID from the same page
PHONE_NUMBER_ID="your_whatsapp_phone_number_id"

# Your secret API key from OpenAI
OPENAI_API_KEY="your_openai_api_key"
```


💻 The Code
## Here is the code for each file.


### openai_service.py

Python
```bash 
import openai
import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")

def generate_response(message):
    """
    Generates a response from OpenAI's ChatCompletion API.
    """
    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[{"role": "user", "content": message}]
        )
        return response['choices'][0]['message']['content']
    except Exception as e:
        print(f"Error generating response from OpenAI: {e}")
        return "Sorry, I couldn't process your request right now."

</details>
```

### 💬 whatsapp_utils.py

Python
```bash 
import requests
import os
from dotenv import load_dotenv
from openai_service import generate_response
```
```bash 
# Load environment variables
load_dotenv()

ACCESS_TOKEN = os.getenv("ACCESS_TOKEN")
PHONE_NUMBER_ID = os.getenv("PHONE_NUMBER_ID")
WHATSAPP_API_URL = f"[https://graph.facebook.com/v18.0/](https://graph.facebook.com/v18.0/){PHONE_NUMBER_ID}/messages"
```
```bash 
def send_whatsapp_message(recipient, message):
    """
    Sends a message to a WhatsApp recipient using the Meta Graph API.
    """
    headers = {
        "Authorization": f"Bearer {ACCESS_TOKEN}",
        "Content-Type": "application/json"
    }
    data = {
        "messaging_product": "whatsapp",
        "to": recipient,
        "type": "text",
        "text": {"body": message}
    }
    try:
        response = requests.post(WHATSAPP_API_URL, headers=headers, json=data)
        response.raise_for_status()  # Raise an exception for bad status codes
        print(f"Message sent to {recipient}, status: {response.status_code}")
    except requests.exceptions.RequestException as e:
        print(f"Error sending WhatsApp message: {e}")
```
```bash 
def process_whatsapp_message(body):
    """
    Processes the incoming webhook payload from WhatsApp.
    """
    try:
        entry = body.get("entry", [])[0]
        changes = entry.get("changes", [])[0]
        value = changes.get("value", {})
        messages = value.get("messages", [])

        if messages:
            msg = messages[0]
            from_number = msg["from"]
            user_text = msg["text"]["body"]
            
            print(f"Message from {from_number}: {user_text}")
            
            # Get AI response and send it back
            ai_reply = generate_response(user_text)
            send_whatsapp_message(from_number, ai_reply)
            
    except (IndexError, KeyError) as e:
        print(f"Error processing webhook payload: {e}")
        print(f"Payload: {body}")

</details>
```
###  🚀 run.py (The Flask App)
```bash 
Python

from flask import Flask, request
from whatsapp_utils import process_whatsapp_message
import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

app = Flask(__name__)

# Get the verify token from environment variables
VERIFY_TOKEN = os.getenv("VERIFY_TOKEN")

@app.route('/webhook', methods=['GET'])
def verify_webhook():
    """
    Verifies the webhook subscription with Meta.
    """
    if request.args.get("hub.verify_token") == VERIFY_TOKEN:
        print("Webhook verified.")
        return request.args.get("hub.challenge")
    print("Invalid verification token.")
    return "Invalid verification token", 403

@app.route('/webhook', methods=['POST'])
def receive_message():
    """
    Receives messages from the WhatsApp webhook.
    """
    data = request.get_json()
    print("Received data:", data)  # Log incoming data
    
    if data and data.get("object") == "whatsapp_business_account":
        process_whatsapp_message(data)
        
    return "OK", 200

if __name__ == '__main__':
    app.run(port=8000, debug=True)

</details>
```

 ## Running the Bot
### Step 1: Start the Flask App
### Open your terminal, navigate to the project directory, and run the Flask application.

``` Bash

python3 run.py
Your app should now be running on http://127.0.0.1:8000.
```

## Step 2: Expose Your App with ngrok
### Open a new terminal window and run ngrok to expose your local port 8000 to the public internet.

``` Bash

# If you haven't already, add your authtoken
ngrok config add-authtoken <YOUR_NGROK_AUTHTOKEN>
```
# Expose port 8000
```Bash
ngrok http 8000
ngrok will provide you with a public HTTPS forwarding URL. 
Copy it. It will look something like https://your-unique-id.ngrok-free.app. 
```

## Step 3: Configure the Webhook in Meta
### Go back to your Meta App Dashboard → WhatsApp → Configuration.

### Click Edit in the Webhook section.

### Callback URL: Paste your ngrok HTTPS URL and add /webhook to the end (e.g., https://your-unique-id.ngrok-free.app/webhook).

* Verify Token: Enter the same secret token you defined in your .env file.
Click Verify and Save.

* After verifying, click Manage and subscribe to the messages webhook field.

* You should see a WEBHOOK_VERIFIED message in your Flask terminal.

## Step 4: Test Your Bot! 📲
### You're all set! Send a message from your personal WhatsApp account to the test number provided by Meta. You should receive a reply generated by OpenAI. 🎉




### 🧠 Going Further
This is just the beginning! Here are some ideas to enhance your bot:

Deploy to Production: Deploy your Flask app to a cloud service like Render, Railway, or AWS so it's always online.

Add Context Memory: Keep track of conversation history (e.g., in a simple SQLite database or a dictionary) to have more contextual conversations.

Support More Features: Extend the bot to handle interactive buttons, images, or even voice messages.

Define a Personality: Customize the AI's personality by providing a system prompt in openai_service.py (e.g., "You are a friendly and helpful assistant named Botly.").

Go Live: When ready, follow Meta's official documentation to connect a permanent business phone number using the Meta Business API