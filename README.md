# mobilpay-card-node

[![dependencies Status](https://david-dm.org/robi-richter/mobilpay-node/status.svg)](https://david-dm.org/robi-richter/mobilpay-node)
[![devDependencies Status](https://david-dm.org/robi-richter/mobilpay-node/dev-status.svg)](https://david-dm.org/robi-richter/mobilpay-node?type=dev)

## About
This is a lightweight NodeJs library, updated and already builded (developed by robi-richter - *https://github.com/robi-richter/mobilpay-node*), to integrate MobilPay payment gateway in your projects.
It's still **under development** and **does't support all the payment options**.


## Usage

```javascript
var mobilpay = require('mobilpay-card-node');
var constants = mobilpay.constants;
```

Bare in mind that you need to extract the public key from the public certificate provided by Mobilpay

```bash
openssl x509 -pubkey -noout -in PUBLIC_CERTIFICATE_FILE.cer > PUBLIC_KEY.pem
```

Create Mobilpay instance
```javascript
// create Mobilpay instance
var MobilPay = new mobilpay.Mobilpay({
  signature: '',
  sandbox: true,
  publicKeyFile: '',
  privateKeyFile: '',
});
```
Create a new payment request
```javascript
var paymentRequest = MobilPay.createRequest({
  amount: parseFloat(Math.round(Math.random() * 10000)/100).toFixed(2),
  customerId: '12345',
  details: 'Payment details',
  billingAddress: {
    type: constants.ADDRESS_TYPE_PERSON,
    firstName: 'Damian',
    lastName: 'Gardner',
    email: 'damian.gardner@inbound.plus',
    address: '4793 College Street, Cluj-Napoca, Cluj',
    mobilePhone: '0722222222'
  },
  confirmUrl: 'http://mysite.local/confirm',
  returnUrl: 'http://mysite.local/return',
  params: {
    test1: 'test param 1',
    test2: 'test param 2',
  }
});
```

Prepare redirect data
```javascript
MobilPay.prepareRedirectData(paymentRequest)
  .then(function(result) {

    console.log(
      'Redirect URL: ' + result.url,
      'env_key: ' + result.envKey,
      'data: ' + result.data
    );

  })
  .catch(function(err) {
    console.log(err);
  });
```

With the prepared data render a form and display it to the client

```html
<form name="frmPaymentRedirect" method="post" action="{{ redirectUrl }}">
    <input type="hidden" name="env_key" value="{{ envKey }}"/>
    <input type="hidden" name="data" value="{{ data }}"/>
    <p>
        You will be redirected to a secure page on mobilpay.ro
    </p>
    <p>
        Click to continue <input type="submit" value="Submit">
    </p>
</form>
```

Mobilpay will POST notifications to your confirm URL which can be something
like this *http://mysite.local/confirm*. to handle these notifications you'll need
to add this pice of code your route handler
```javascript
mobilPay.handleGatewayResponse({
  envKey: 'content from env_key POST body',
  data: 'content from data POST body'
  })
  .then(function(data) {
    // check the response and send back an acknowledge message
    var response = new mobilpay.MerchantResponse({
      message: data.response.crc
    });

    // send response.toXml() as response
  })
  .catch(function(err) {
    // handle error case
  });
```
