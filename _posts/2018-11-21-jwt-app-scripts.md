---
layout: post
comments: true
title: JWTs in Google Apps Script
date: 2018-11-21 8:20:30
categories: gas
short_description: How to create a JWT in a Google Apps Script
---

Google Apps Script is a really amazing way to extend Google's suite of productivity tools.
It's even more enjoyable to work with now because of [clasp](https://github.com/google/clasp). clasp makes it easy to check your code into git and gives you a bunch of tooling for deploying and manage your app. It also let's you write your code in [Typescript](https://www.typescriptlang.org/)! The autocompletion you get is a real boon to productivity because you no longer have to
constantly read through the docs.

I've been working on a Google Sheets extension that calls out to the [Stella Connect](https://stellaconnect.io/) API and loads data into your current active sheet. To do this, I needed to create
a [JSON web token](https://jwt.io/) to authenticate. It's pretty easy to do this with the
built in Apps Script [Utilities](https://developers.google.com/apps-script/reference/utilities/utilities) class.

Here's an example that uses the `HMAC256` algorithm to create the signature. You should be
able change this snippet to use whichever algorithm your target API expects. This snippet also only sends the issued at [claim](https://tools.ietf.org/html/rfc7519#page-8), so you'll need to adapt that to your needs as well.

```javascript
var base64Encode = function (str) {
    var encoded = Utilities.base64EncodeWebSafe(str);
    // Remove padding
    return encoded.replace(/=+$/, '');
};

var encodeJWT = function (secret) {
    var header = JSON.stringify({
        typ: 'JWT',
        alg: 'HS256'
    });
    var encodedHeader = base64Encode(header);
    var iat = new Date().getTime() / 1000;
    var payload = JSON.stringify({
        iat: iat
    });
    var encodedPayload = base64Encode(payload);
    var toSign = [encodedHeader, encodedPayload].join('.');
    var signature = Utilities.computeHmacSha256Signature(toSign, secret);
    var encodedSignature = base64Encode(signature);
    console.log("Encoded Signature: " + encodedSignature);
    return [toSign, encodedSignature].join('.');
};
```
