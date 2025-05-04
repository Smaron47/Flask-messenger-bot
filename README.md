# Flask-messenger-bot
**Flask-Based Facebook Messenger Bot**

---

## Table of Contents

1. Project Overview
2. Creation History
3. Technology Stack & Module Descriptions
4. File Structure
5. Environment Setup & Configuration
6. Facebook App & Messenger Setup
7. Application Architecture & Data Flow
8. Detailed Code Walkthrough

   * 8.1 Global Configuration (Tokens & Bot Initialization)
   * 8.2 Quick Reply & Generic Templates Payloads
   * 8.3 Welcome & Command Handler: `wc()`
   * 8.4 Webhook Verification Endpoint (`verify`)
   * 8.5 Message Webhook Endpoint (`webhook`)
   * 8.6 Application Entry Point
9. Key Functional Components

   * 9.1 Sending Quick Replies
   * 9.2 Sending Generic Template Messages
   * 9.3 Handling Postbacks & Text Messages
10. Error Handling & Logging
11. Deployment Considerations
12. Extensions & Customizations
13. Keywords

---

## 1. Project Overview

This project implements a **Facebook Messenger chatbot** using **Flask** as the web framework and **PyMessenger** for interacting with the Messenger Send API. The bot can respond with quick-reply menus, generic template messages (cards with images & buttons), and simple text replies based on user input.

**Primary Features:**

* Quick reply menu for main options (About Us, Contact Us, Options, etc.)
* Generic template cards for links (projects, webpages, contact cards)
* Simple keyword-based conversational flow via the `wc()` function
* Webhook verification and message processing endpoints

---

## 2. Creation History

* **Year of Creation:** 2020
* **Author:** Smaron Biswas
* **Purpose:** Prototype Messenger bot for a digital agency demo

---

## 3. Technology Stack & Module Descriptions

* **Python 3.x**: Core language
* **Flask**: Lightweight web framework to expose HTTP endpoints for Facebook webhooks
* **pymessenger.Bot**: Wrapper for the Facebook Messenger Send API to send messages back to users
* **requests**: (Imported but not used in final) could be used for external HTTP calls

---

## 4. File Structure

```plaintext
messenger_bot/
├── app.py                   # Main Flask application and bot logic
├── requirements.txt         # Dependencies (Flask, pymessenger, requests)
└── README.md                # Project documentation
```

---

## 5. Environment Setup & Configuration

1. **Install Dependencies**

   ```bash
   pip install flask pymessenger requests
   ```
2. **Facebook App Setup**

   * Create a Facebook App and a Facebook Page.
   * In the App Dashboard, add the Messenger product.
   * Generate a **Page Access Token** and set it as `ACCESS_TOKEN` in `app.py`.
   * Set a **Verify Token** (any secret string) and configure it in the webhook settings.
3. **Webhook Endpoint**

   * Configure `https://<your_domain>/` as your webhook URL for both GET (verification) and POST (messages).

---

## 6. Facebook App & Messenger Setup

1. Under **Messenger → Settings**, set **Webhook URL** and **Verify Token**.
2. Subscribe the webhook to the **messages** and **messaging\_postbacks** events.
3. Assign the Page Access Token to your Flask app configuration.

---

## 7. Application Architecture & Data Flow

1. **Verification**: Facebook sends a GET request with `hub.mode` and `hub.challenge`; the bot verifies the `hub.verify_token` and echoes `hub.challenge`.
2. **Message Handling**: Facebook POSTs incoming messages to the same endpoint. The app parses the JSON, extracts `sender_id` and `message.text`, and invokes `wc(sender_id, query)`.
3. **Response Generation**: Based on the `query`, `wc()` sends quick replies or generic templates via `bot.send_quick_replies_message()` or `bot.send_generic_message()`.

---

## 8. Detailed Code Walkthrough

### 8.1 Global Configuration

```python
ACCESS_TOKEN = "<YOUR_PAGE_ACCESS_TOKEN>"
bot = Bot(ACCESS_TOKEN)
VERIFICATION_TOKEN = "hello"
```

* `ACCESS_TOKEN`: Authorizes Send API calls.
* `bot`: PyMessenger Bot instance.
* `VERIFICATION_TOKEN`: Shared secret for webhook setup.

### 8.2 Quick Reply & Generic Template Payloads

* **`qr`, `Contact`, `gp`, etc.**: Lists of dictionaries defining quick-reply buttons (text, title, payload).
* **`but`, `about`, `webp`, `works`**: Generic template cards (title, subtitle, image\_url, buttons).

### 8.3 Welcome & Command Handler: `wc(sender_id, query)`

```python
def wc(s, query):
    if query in welcome_list:
        bot.send_quick_replies_message(s, "Welcome message", qr)
    elif query == "About Us":
        bot.send_generic_message(s, about)
        bot.send_quick_replies_message(s, "Back to menu?", qr)
    # ... more elif branches for each keyword
```

* Maps user text to appropriate response type.
* Uses `bot.send_text_message`, `bot.send_quick_replies_message`, or `bot.send_generic_message`.

### 8.4 Webhook Verification Endpoint

```python
@app.route('/', methods=['GET'])
def verify():
    if request.args.get("hub.mode") == "subscribe" and request.args.get("hub.challenge"):
        if request.args.get("hub.verify_token") != VERIFICATION_TOKEN:
            return "Verification token mismatch", 403
        return request.args["hub.challenge"], 200
    return "Hello world", 200
```

* Verifies Facebook webhook subscription request.

### 8.5 Message Webhook Endpoint

```python
@app.route('/', methods=['POST'])
def webhook():
    data = request.get_json(force=True)
    if data['object'] == "page":
        for entry in data['entry']:
            for event in entry['messaging']:
                sender_id = event['sender']['id']
                if event.get('message') and event['message'].get('text'):
                    query = event['message']['text']
                    wc(sender_id, query)
    return "ok", 200
```

* Receives and processes incoming messages.
* Delegates to `wc()` for response.

### 8.6 Application Entry Point

```python
if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80, debug=True)
```

* Starts Flask server on port 80.

---

## 9. Key Functional Components

### 9.1 Sending Quick Replies

* Method: `bot.send_quick_replies_message(recipient_id, text, quick_replies_list)`
* `quick_replies_list`: array of `{content_type, title, payload}`

### 9.2 Sending Generic Template Messages

* Method: `bot.send_generic_message(recipient_id, elements_list)`
* `elements_list`: array of card dictionaries `{title, subtitle, image_url, item_url, buttons}`

### 9.3 Handling Postbacks & Text Messages

* Currently handles only text. Postback payloads can be captured via `messaging_event.get('postback')`.

---

## 10. Error Handling & Logging

* Minimal error handling; unrecognized tokens return default text.
* Extend with try/except around API calls.
* Add logging via Python `logging` module for production.

---

## 11. Deployment Considerations

* Host on a public server (Heroku, AWS, DigitalOcean) with HTTPS for webhook.
* Ensure valid SSL certificate and open port 443.

---

## 12. Extensions & Customizations

* Add NLP with wit.ai or Dialogflow for natural language understanding.
* Persist user state in Redis or a database.
* Implement media/message attachments handling.

---

## 13. Keywords

```
Flask messenger bot
pymessenger tutorial
facebook webhook flask
quick replies messenger
generic template python
facebook bot python
flask chatbot
pymessenger example
facebook messenger integration
bot development 2020
digital agency chatbot
```

---

*Documentation authored by Smaron Biswas (2020).*
