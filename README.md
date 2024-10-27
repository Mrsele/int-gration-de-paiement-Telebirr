Telebirr API Integration
This guide provides the steps to integrate Telebirr's payment API into your application. The Telebirr API, offered by Ethio Telecom, enables businesses to initiate and track mobile payments directly within their applications.

Table of Contents
Prerequisites
Getting Started
Integration Steps
1. Authenticate
2. Initiate Payment
3. Query Payment Status
4. Handle Callbacks
Example Code
Useful Links
Prerequisites
Ethio Telecom Developer Account: Sign up on the Ethio Telecom Developer Portal.
API Credentials: Register your application on the portal to obtain your appId and appKey.
Server Setup: Have a backend server capable of making HTTP requests and handling webhook callbacks.
Getting Started
Register on the developer portal and obtain your credentials:

appId: Unique identifier for your app.
appKey: Secret key for authenticating API requests.
Set up environment variables for your API credentials:

bash
Copy code
export TELEBIRR_APP_ID="your_app_id"
export TELEBIRR_APP_KEY="your_app_key"
Integration Steps
1. Authenticate
The Telebirr API requires an accessToken for each session. Use the /oauth2/token endpoint to generate this token. Tokens may expire periodically, so ensure you handle token renewal.

Request:

http
Copy code
POST /oauth2/token
Content-Type: application/json
{
  "appId": "your_app_id",
  "appKey": "your_app_key"
}
Response:

json
Copy code
{
  "accessToken": "your_access_token",
  "expiresIn": 3600
}
2. Initiate Payment
To start a payment, call the /api/payment/initiate endpoint with details of the transaction.

Request:

http
Copy code
POST /api/payment/initiate
Content-Type: application/json
Authorization: Bearer your_access_token
{
  "appId": "your_app_id",
  "outTradeNo": "unique_transaction_id",
  "subject": "Payment for Order #1234",
  "totalAmount": "100.00",
  "notifyUrl": "https://your-server.com/callback",
  "returnUrl": "https://your-app.com/payment-complete"
}
Parameters:

outTradeNo: Unique ID for this transaction.
totalAmount: Amount to be paid.
notifyUrl: Callback URL for Telebirr to notify upon payment completion.
returnUrl: URL where the user is redirected post-payment.
3. Query Payment Status
Check the status of a payment using the outTradeNo from the initiate payment request.

Request:

http
Copy code
GET /api/payment/query?outTradeNo=unique_transaction_id
Authorization: Bearer your_access_token
Response: The response contains the payment status and details, useful for confirming successful transactions.

4. Handle Callbacks
Implement a listener on your notifyUrl endpoint to handle payment notifications from Telebirr. Verify the data to confirm if the payment was successful.

Example Code
Here’s an example of initiating a payment using Node.js and Express.

javascript
Copy code
const axios = require('axios');

const TELEBIRR_APP_ID = process.env.TELEBIRR_APP_ID;
const TELEBIRR_APP_KEY = process.env.TELEBIRR_APP_KEY;

async function authenticate() {
  const response = await axios.post('https://api.telebirr.com/oauth2/token', {
    appId: TELEBIRR_APP_ID,
    appKey: TELEBIRR_APP_KEY
  });
  return response.data.accessToken;
}

async function initiatePayment(transactionId, amount) {
  const accessToken = await authenticate();
  const response = await axios.post(
    'https://api.telebirr.com/api/payment/initiate',
    {
      appId: TELEBIRR_APP_ID,
      outTradeNo: transactionId,
      subject: `Order Payment #${transactionId}`,
      totalAmount: amount,
      notifyUrl: 'https://your-server.com/callback',
      returnUrl: 'https://your-app.com/payment-complete'
    },
    { headers: { Authorization: `Bearer ${accessToken}` } }
  );
  return response.data;
}
Callback Example
Set up a route on your server to listen for payment notifications:

javascript
Copy code
app.post('/callback', (req, res) => {
  const paymentData = req.body;
  // Process and verify the payment
  if (paymentData.status === 'SUCCESS') {
    console.log('Payment successful:', paymentData);
  }
  res.status(200).send('Callback received');
});
Useful Links
Ethio Telecom Developer Portal
Telebirr API Documentation
