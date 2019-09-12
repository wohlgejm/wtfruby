---
layout: post
comments: true
title: Amazon api requests from google sheets
date: 2017-02-14 20:20:30
categories: javascript
short_description: How to make amazon product api requests from google sheets
---

Here is some javascript code to sign requests for the Amazon Product Advertising API in Google Sheets. This uses
v2 of Amazon's signing process, so you can adopt it to use other AWS services. It uses Google's `Utilities` service
to handle the crypto. The URI encoding also needs to be strict (see this [article](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent)).

```javascript
function fixedEncodeURIComponent(str) {
  return encodeURIComponent(str).replace(/[!'()*]/g, function(c) {
    return '%' + c.charCodeAt(0).toString(16);
  });
}

function AwsSigner() {
  this.accessKey = 'youraccesskey';
  this.secretKey = 'yoursecretkey';
  this.associateId = 'yourassociateid';
  this.host = 'webservices.amazon.com';
  this.method = 'GET';
  this.endpoint = 'http://webservices.amazon.com/onca/xml';
  this.path = '/onca/xml';
  this.service = 'AWSECommerceService';
  this.version = '2013-08-01'
  this.date = new Date().toISOString();
}

AwsSigner.prototype.url = function(params) {
  this.params = params;
  signature = this.signature(this.stringToSign());
  return this.endpoint + '?' + this.queryString() + '&Signature=' + fixedEncodeURIComponent(signature);
}

AwsSigner.prototype.signature = function(value) {
  return Utilities.base64Encode(Utilities.computeHmacSha256Signature(value, this.secretKey, Utilities.Charset.UTF_8));
}

AwsSigner.prototype.stringToSign = function() {
  return this.method + '\n' + this.host + '\n' + '/onca/xml' + '\n' + this.queryString();
}

AwsSigner.prototype.queryString = function () {
  str = 'AWSAccessKeyId=' + this.accessKey;
  this.params['AssociateTag'] = this.associateId;
  this.params['Timestamp'] = this.date;
  this.params['Version'] = this.version;
  var params = this.params;
  Object.keys(params).sort().forEach(function(k) {
    str += '&' + k + '=' + fixedEncodeURIComponent(params[k]);
  });
  return str;
}

var signer = new AwsSigner();
var url = signer.url({
  'ItemId': 'SOMEASIN',
  'Operation': 'ItemLookup',
  'ResponseGroup': 'ItemAttributes'
});
```
