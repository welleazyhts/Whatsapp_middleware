# 🤖 Welleazy AI WhatsApp Bot

This guide walks you through setting up the **Welleazy WhatsApp Bot**, which integrates **Meta’s WhatsApp Cloud API** with **Flask** and **OpenAI Assistants API** to automate real-time WhatsApp responses for wellness and corporate clients.

---

## ⚙️ Prerequisites

1. **Meta Developer Account** → [Create here](https://developers.facebook.com/).  
2. **Business App (Type: Business)** → [Create here](https://developers.facebook.com/docs/development/create-an-app/).  
3. **Python 3.9+** installed on your system.  
4. **Ngrok account** for secure local tunneling.  
5. **OpenAI API Key** + **Assistant ID**.

---

## 📦 Project Setup

### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/welleazy-whatsapp-bot.git
cd welleazy-whatsapp-bot
2. Create and Activate Virtual Environment
bash
Copy code
python3 -m venv venv
source venv/bin/activate         # Windows: venv\Scripts\activate
3. Install Dependencies
bash
Copy code
pip install -r requirements.txt
4. Create Environment File
Create a .env file in the project root and add:

env
Copy code
ACCESS_TOKEN=<YOUR_META_ACCESS_TOKEN>
PHONE_NUMBER_ID=<YOUR_PHONE_NUMBER_ID>
WHATSAPP_BUSINESS_ACCOUNT_ID=<YOUR_BUSINESS_ACCOUNT_ID>
RECIPIENT_WAID=<YOUR_TEST_WHATSAPP_NUMBER>
VERSION=v20.0

VERIFY_TOKEN=<YOUR_VERIFY_TOKEN>
APP_ID=<YOUR_META_APP_ID>
APP_SECRET=<YOUR_META_APP_SECRET>

OPENAI_API_KEY=<YOUR_OPENAI_API_KEY>
OPENAI_ASSISTANT_ID=<YOUR_ASSISTANT_ID>
☁️ Meta WhatsApp Cloud API Setup
Step 1 — Add the WhatsApp Product
Go to your Meta Developer Dashboard.

Click Add Product → WhatsApp → Set Up.

Link or create a WhatsApp Business Account (WABA).

A test number will be generated for you.

Step 2 — Generate an Access Token
Under API Setup, click Generate Access Token.

Copy the value shown → paste it into your .env as ACCESS_TOKEN.

You’ll also see:

Phone Number ID

WhatsApp Business Account ID
Copy both and update in .env.

If you get “We limit how often you can post…” → wait 10–15 minutes and retry (Meta’s rate limit).

Step 3 — Verify Webhook URL
Run your Flask app:

bash
Copy code
python run.py
Start ngrok:

bash
Copy code
ngrok http 8000 --domain your-welleazy.ngrok-free.app
Copy the public URL (e.g. https://your-welleazy.ngrok-free.app).

In Meta → WhatsApp → Configuration, click Edit Callback URL:

Callback URL: https://your-welleazy.ngrok-free.app/webhook

Verify Token: value from your .env

Click Verify and Save — if Flask logs show WEBHOOK_VERIFIED, it worked.

Step 4 — Subscribe to Messages
In WhatsApp > Configuration, click Manage.

Click Subscribe to messages events.

Send a test message from your WhatsApp — check Flask logs for a webhook event.

Step 5 — Connect OpenAI
Create an Assistant from your OpenAI Dashboard.

Copy its ID → update OPENAI_ASSISTANT_ID in .env.

Upload your Welleazy knowledge base (e.g. FAQs PDF).

The bot will now respond to WhatsApp messages intelligently.

🧠 How It Works
pgsql
Copy code
User → WhatsApp → Meta Webhook → Flask Server
       → OpenAI Assistant (processes query)
       → Flask → Meta API → WhatsApp reply
🚀 Run the Application
Start Flask:

bash
Copy code
python run.py
Expose via Ngrok:

bash
Copy code
ngrok http 8000 --domain your-welleazy.ngrok-free.app
Send a WhatsApp message to your connected test number — you’ll receive an AI-generated reply from Welleazy.

🧩 Folder Structure
bash
Copy code
📦app
 ┣ 📂decorators
 ┃ ┗ 📜security.py          # Verifies Meta webhook signatures
 ┣ 📂services
 ┃ ┗ 📜openai_service.py    # OpenAI Assistant logic
 ┣ 📂utils
 ┃ ┗ 📜whatsapp_utils.py    # WhatsApp send/receive + processing
 ┣ 📜views.py               # Webhook endpoints
 ┣ 📜config.py              # Flask configuration
 ┗ 📜__init__.py            # App initialization
run.py                      # Starts Flask app
.env.example                # Environment template
requirements.txt
🧾 Common Issues
Issue	Cause	Solution
No Phone Number ID	WhatsApp product not linked	Add product under “Add Product → WhatsApp”
Webhook not verifying	Token mismatch	Ensure .env and Meta dashboard match
No AI replies	Missing OPENAI_ASSISTANT_ID or invalid API key	Verify your OpenAI setup
Rate limit error	Testing too often	Wait 10–15 minutes and retry

💡 Notes
Only template messages can be the first message to a user.

For production use, migrate your own number into the business platform.

Use long-lived access tokens for stability (60-day or permanent).

👨‍💻 Author
Developed by: Harsh Palod
Company: Welleazy Technologies Pvt. Ltd.
Purpose: AI-powered WhatsApp assistant for wellness and health support.


