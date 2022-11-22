eGHL Cordova Plugin
=============================

Cordova Library for integrating with the eGHL payment gateway's mobile SDK.


Installation
------------

```
cordova plugin add https://github.com/eGHL/cordova-plugin-eghl.git
```
> cordova plugin add cordova-plugin-eghl is deprecated

Refer [here](https://github.com/eGHL/cordova-plugin-eghl/wiki/iOS-V2-Linking-Libraries) on how to link to eGHL v2 SDK

Usage
-----

### Info

To get the version of this plugin at runtime, the following fields are available:

```javascript
eGHL.version // e.g. '3.2.1'
eGHL.name // 'eGHL Cordova Plugin'
```

<a name="makePayment" />
### Request payment

Note: `makePayment()` and [`mpeRequest()`](#mpeRequest) only
allow one ongoing request at any time. Any subsequent calls made while the
previous request is still ongoing will receive an error.

To request payment via any channel, including Masterpass Express:

```javascript
eGHL.makePayment(
    {
        /*
        * Mandatory payment parameters list:
        * TransactionType, PymtMethod, ServiceID, PaymentID, OrderNumber, PaymentDesc, MerchantReturnURL, Amount, CurrencyCode, CustIp, CustName, CustEmail, CustPhone
        * Check the eGHL documentation for parameter names.
        */
        
        TransactionType : "SALE",
        PymtMethod : "ANY",
        CurrencyCode : "MYR",

        PaymentGateway: "https://...", // Payment gateway URL given by eGHL
        ServiceID : "abc", //Merchant Code or Service ID given by eGHL
        Password: "password", // Merchant password given by eGHL
        MerchantName : "ABC Sdn Bhd",
        
        PaymentID : "P0000001", // Unique string for each payment
        OrderNumber : "P0000001", // Order number to refer current payment, can duplicate.
        PaymentDesc : "eGHL Payment testing",
        Amount : "123.10", // e.g. 1000.00
                       // Invalid format: 1,000.00 or 1000
        CustName : "Beta Tester",
        CustEmail : "Tester@mail.com",
        CustPhone : "60123456789",
        CustMAC : "",
        CustIP : "",
        MerchantReturnURL :  "SDK", // redirect when payment complete.
        MerchantCallBackURL : "https://...", // server to server callback url
        MerchantApprovalURL : "https://...",
        MerchantUnApprovalURL : "https://...",
        LanguageCode : "EN",
        PageTimeout : "600", // eGHL payment page timeout in seconds
        CardHolder : "",
        CardNo : "",
        CardExp : "",
        CardCVV2 : "",
        IssuingBank : "",
        TokenType : "",
        Token : "",
        Param6 : "",
        Param7 : "",
        B4TaxAmt : "",
        TaxAmt : "",
        BillAddr : "",
        BillPostal : "",
        BillCity : "",
        BillRegion : "",
        BillCountry : "",
        ShipAddr : "",
        ShipPostal : "",
        ShipCity : "",
        ShipRegion : "",
        ShipCountry : "",
        SessionID : "",
        EPPMonth : "",
        PromoCode : "",
        
        // SDK exclusive
        PaymentTimeout : -1, // Android: Force close webview after x seconds
        sdkTimeout: -1, // iOS: Force close webview after x seconds
        _finaliseMessage: "Finalising Payment", // iOS only
        _cancelMessage: "Cancelling Payment", // iOS only
        
        // Masterpass exclusive
        ReqToken : "",
        PairingToken : "",
        ReqVerifier : "",
        PairingVerifier : "",
        CheckoutResourceURL : "",
        CardId : "",
        PreCheckoutId : "",
    },
    function (resp) {
        // Success callback
        // TxnStatus:
        // 0 - Transaction Successful
    },
    function (err) {
        // Failure callback
        // TxnStatus:
        // 1 = Transaction Failed
        // 2 = Transaction Pending
        // Other = Error
    }
);
```


<a name="mpeRequest"></a>
### Masterpass Express info

Note: [`makePayment()`](#makePayment) and `mpeRequest()` only
allow one ongoing request at any time. Any subsequent calls made while the
previous request is still ongoing will receive an error.

```javascript
eGHL.mpeRequest(
    {
        "PaymentGateway": "https://..." // payment gateway URL provided by eGHL
        "ServiceID": "abc", // merchant ID provided by eGHL
        "Password": "password" // merchant password provided by eGHL
        "CurrencyCode": "MYR",
        "Amount": "123.10",
        "TokenType": "MPE / MSC", // MPE = Masterpass Express, MSC = Masterpass Standard
            // Checkout.
        "Token": "User identifier, for TokenType==MPE only",
        "PaymentDesc": "Payment description...",
    },
    function (resp) {
        // Success callback
        // resp can take one of two forms, based on whether the user was already
        // paired with Masterpass Express or not.
        // 1. Not paired:
        //      {
        //          "ReqToken": "...",
        //          "PairingToken": "..."
        //      }
        // 2. Already paired:
        //      {
        //          "PreCheckoutId": "...",
        //          "Cards": [
        //              {
        //                  "LastFour": "0014",
        //                  "CardId": "358f9812-a99a-49d9-9385- f0a7b67e377c",
        //                  "BrandId": "master",
        //                  "CardAlias": null,
        //                  "ExpiryMonth": 12,
        //                  "SelectedAsDefault": true,
        //                  "BNBUnverified": null,
        //                  "CardHolderName": "OM Testing",
        //                  "ExtensionPoint": null,
        //                  "BillingAddress": {
        //                      "City": "KL",
        //                      "Country": "US",
        //                      "CountrySubdivision": "WA",
        //                      "Line1": "123, street 123",
        //                      "Line2": null,
        //                      "Line3": null,
        //                      "PostalCode": "11222",
        //                      "ExtensionPoint": null
        //                  },
        //                  "BrandName": "MasterCard",
        //                  "ExpiryYear": 2020
        //              },
        //              ...
        //          ]
        //      }
    },
    function (errMsg) {
        // Error callback
        // errMsg: "error string"
    }
);
```

### Special Android notes

Android may kill your Cordova activity while your user is in the eGHL activity.
When this happens, payment results will NOT be sent to the JavaScript callback
which you passed to the other methods (above). Instead, it will be sent to your
app with the "resume" Cordova event.

```javacript
document.addEventListener("resume", function (event) {
    /* Event:
        {
            action: "resume",
            pendingResult: {
                pluginServiceName: "eGHL",
                pluginStatus: "OK", // or "Error", etc.
                result: ... // same result that would have been passed to
                    // your normal callback function.
            }
        }
    */
}, false)
```

See also:

- https://cordova.apache.org/docs/en/latest/guide/platforms/android/index.html#lifecycle-guide
- https://cordova.apache.org/docs/en/latest/guide/platforms/android/plugin.html#launching-other-activities


### Message Customisation

**iOS:**

On iOS, you can only customise the "Finalising Payment" message. Set your own
message by setting the `_finaliseMessage` property in the first parameter of
your [`.makePayment()` call](#makePayment). If you do not set this, the default message set by
eGHL's SDK will be used.

**Android:**

To customise the messages displayed by the eGHL SDK, set any of these values in
your `strings.xml`:

```
<string name="eghl_progress_message">this is from app strings xml progress </string>
<string name="eghl_verification_message">this is from app strings xml verification</string>
<string name="eghl_cancel_dialog_title">this is from app strings exit title</string>
<string name="eghl_cancel_dialog_message">this is from app strings message</string>
<string name="eghl_prevent_back_toast_message">this is from app strings toast</string>
<string name="eghl_ssl_error_dialog_title">this is from app strings title ssl </string>
<string name="eghl_ssl_error_dialog_message">this is from app strings message ssl </string>
```
