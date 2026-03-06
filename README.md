
# 📧 Email Importance Classifier with n8n + Groq

This project uses **n8n** to automatically classify incoming emails as **important** or **not important** using Groq’s Llama 3 models. If an email is marked important, you’ll receive an instant notification via SMTP.

---

## 🚀 Workflow Overview

1. **IMAP Email Trigger**  
   - Connects to Gmail (or any IMAP server).  
   - Fetches new emails from your inbox.  

2. **HTTP Request (Groq API)**  
   - Sends the email subject and body to Groq’s Chat Completions API.  
   - Model classifies the email as `"important"` or `"not important"`.  

3. **IF Node**  
   - Checks if the Groq response equals `"important"`.  

4. **SMTP Node**  
   - Sends a notification email when an important email is detected.  

---

## 🔑 Prerequisites

- [n8n](https://n8n.io) (self-hosted or cloud)  
- Gmail account with **IMAP enabled**  
- Gmail **App Password** (required if 2FA is enabled)  
- Groq API key from [Groq Console](https://console.groq.com)  

---

## ⚙️ Node Configuration

### 1. IMAP Email Trigger
- **Host**: `imap.gmail.com`  
- **Port**: `993`  
- **User**: your Gmail address  
- **Password**: Gmail App Password  
- **SSL/TLS**: enabled  
- **Mailbox**: `INBOX`  

---

### 2. HTTP Request (Groq)
- **Method**: `POST`  
- **URL**: `https://api.groq.com/openai/v1/chat/completions`  

**Headers**
```json
{
  "Authorization": "Bearer YOUR_API_KEY",
  "Content-Type": "application/json",
  "Accept": "application/json"
}
```

**Body**
```json
{
  "model": "llama3-8b-instant",
  "messages": [
    {
      "role": "system",
      "content": "Classify emails strictly as 'important' or 'not important'. Reply only with one word."
    },
    {
      "role": "user",
      "content": "Email subject: {{$json.subject}}. Email body: {{$json.text}}"
    }
  ],
  "temperature": 0,
  "max_tokens": 50
}
```

---

### 3. IF Node
- **Value 1**:  
  ```
  {{$json["choices"][0]["message"]["content"]}}
  ```
- **Operation**: Equals  
- **Value 2**:  
  ```
  important
  ```

---

### 4. SMTP Node
- **Host**: `smtp.gmail.com`  
- **Port**: `465` (SSL) or `587` (TLS)  
- **User**: your Gmail address  
- **Password**: Gmail App Password  
- **SSL/TLS**: enabled  

**Message Example**
- **To**: your notification email  
- **Subject**:  
  ```
  Important Email Alert: {{$json.subject}}
  ```
- **Text**:  
  ```
  You received an important email.
  Subject: {{$json.subject}}
  Body: {{$json.text}}
  ```

---

## ✅ Final Workflow
```
IMAP Email Trigger → HTTP Request (Groq) → IF Node → [True → SMTP Notification] / [False → Stop]
```

---

## 📌 Notes
- Use `llama3-8b-8192` for faster classification, or `llama3-70b-8192` for more accurate results.  
- Make sure IMAP is enabled in Gmail settings.  
- Always use a valid Groq API key with the format:  
  ```
  Authorization: Bearer sk_xxxxxxxxx
  ```

---

## 📜 License
MIT
```

