# dotwallet-express

[GitHub](https://github.com/dotwallet/dotwallet-express)

An SDK for quickly building an express server that uses dotwallet's APIs

## To use

```bash
npm install dotwallet-express
```

```js
const express = require('express');
const app = express();

const DotWallet = require('dotwallet-express');
// Initialize DotWallet with your app_id and secret
const dotwallet = DotWallet('<YOUR_APP_ID>', '<YOUR_APP_SECRET>');

// Note: The last param of each call 'true' is for logging. It can be omitted if you don't want to see logs.

// Handle the authentication response.
// Optionally redirect the browser to '/restricted-page'.
app.get('/auth', async (req, res, next) => {
  const result = await dotwallet.handleAuthResponse(req, res, next, '/restricted-page', true);
  // Remember to store the refresh and access token in accessData
  refreshToken = result.accessData.refresh_token;
});

// Refresh access token
const newToken = await dotwallet.refreshAccess(refreshToken);

// Handle order
app.post('/create-order', async (req, res) => {
  // Save the merchant_order_sn for records or checking order status
  const merchant_order_sn = req.body.merchant_order_sn;
  const order_sn = await dotwallet.handleOrder(req.body, true);
  // Return the order_sn to the client
  res.json({ order_sn });

  // Or redirect to the order confirmation page
  res.redirect(`https://www.ddpurse.com/desktop/open/order?order_sn=${order_sn}`);
});

// Handle Automatic payment order
app.post('/create-autopayment', async (req, res) => {
  const orderResultData = await dotwallet.autopayment(req.body, true);
  res.json(orderResultData);
});

app.post('/save-data', async (req, res) => {
  const data = req.body;

  // Optionally check if dev account has hosted account active (just do this once during testing)
  const getHostedData = await dotwallet.getHostedAccount('BSV', true);
  console.log('==============getHostedData==============', getHostedData);

  // Optionally check dev hosted account balance
  const getBalanceData = await dotwallet.hostedAccountBalance('BSV', true);
  console.log('==============getBalanceData==============', getBalanceData);
  if (getBalanceData.confirm + getBalanceData.unconfirm < 700) throw 'developer wallet balance too low';

  // Save data on chain
  const saveData = await dotwallet.saveData(data, 0, true);
  res.json(saveData.data);
});
```

## Examples

For more complete examples, see the examples folder in the package in the [`examples/simple-store/src/index.js`](https://github.com/dotwallet/dotwallet-express/blob/master/src/index.ts) file.

## Dev build

```bash
npm i
npm run build
```
