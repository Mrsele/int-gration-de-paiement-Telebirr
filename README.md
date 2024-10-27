# 🚀 Telebirr API Integration Guide

Integrate **Telebirr** mobile payments quickly and securely using Ethio Telecom's API. This guide covers essential steps for authentication, initiating payments, and handling callbacks in your app using Js.

---

## 📝 Prerequisites

- **Developer Account**: [Sign up on Ethio Telecom’s Developer Portal](https://developer.ethiotelecom.et/).
- **API Credentials**: Register your app to get `appId` and `appKey` for API access.

---

## ⚙️ Setup

1. **Set Environment Variables** for secure storage of your credentials:
   ```bash
   export TELEBIRR_APP_ID="your_app_id"
   export TELEBIRR_APP_KEY="your_app_key"
📲 Integration Steps
1️⃣ Authenticate
To begin, generate an accessToken using your credentials. This token is required for all subsequent API requests.

Request:POST /oauth2/token
Content-Type: application/json

{
  "appId": "your_app_id",
  "appKey": "your_app_key"
}
{
  "accessToken": "your_access_token",
  "expiresIn": 3600
}
2️⃣ Initiate Payment
To create a payment, call the /api/payment/initiate endpoint with transaction details.

Request:POST /api/payment/initiate
Authorization: Bearer your_access_token

{
  "appId": "your_app_id",
  "outTradeNo": "unique_transaction_id",
  "subject": "Order Payment #1234",
  "totalAmount": "100.00",
  "notifyUrl": "https://your-server.com/callback",
  "returnUrl": "https://your-app.com/complete"
}
3️⃣ Query Payment Status
To verify a payment’s status, use the outTradeNo from the initiate payment request.

Request:

GET /api/payment/query?outTradeNo=unique_transaction_id
Authorization: Bearer your_access_token
4️⃣ Handle Callbacks
Set up a callback listener at your notifyUrl to process Telebirr’s payment notifications.

Example (Node.js):
app.post('/callback', (req, res) => {
  const paymentData = req.body;
  if (paymentData.status === 'SUCCESS') {
    console.log('Payment successful:', paymentData);
  }
  res.sendStatus(200);
});
🧑‍💻 Example Code (Node.js)
Here’s a code snippet for authenticating and initiating a payment:

const axios = require('axios');

async function authenticate() {
  const { data } = await axios.post('https://api.telebirr.com/oauth2/token', {
    appId: process.env.TELEBIRR_APP_ID,
    appKey: process.env.TELEBIRR_APP_KEY
  });
  return data.accessToken;
}

async function initiatePayment(transactionId, amount) {
  const token = await authenticate();
  const { data } = await axios.post(
    'https://api.telebirr.com/api/payment/initiate',
    {
      appId: process.env.TELEBIRR_APP_ID,
      outTradeNo: transactionId,
      subject: `Order #${transactionId}`,
      totalAmount: amount,
      notifyUrl: 'https://your-server.com/callback',
      returnUrl: 'https://your-app.com/complete'
    },
    { headers: { Authorization: `Bearer ${token}` } }
  );
  return data;
}
