---
layout: post
comments: true
title: Flight costs in Google sheets
date: 2017-01-13 20:20:30
categories: javascript
short_description: How to write a flight cost function in Google sheets
image_preview: /images/charliedance.gif
---

The ability write your own javascript functions in Google sheets is incredibly powerful. I'm working in a sheet
and I wanted the ability to include flight costs.

Apparent flight cost data is well guarded and the only solution was to use Google's
[QPX Express API](https://developers.google.com/qpx-express/v1/requests). You are limited to 50 queries
a day, but to get around this, we can use caching/persistence (which is built into Google scripts).

You'll need to set up an API key in the [Google Developer Console](https://console.developers.google.com/apis) and enable QPX Express.
Once that is set up, you can use the following two functions in your sheets that take a depart date, return date,
origin and destination.

```javascript
var apiKey = 'apiKeyGoesHere';
var url = 'https://www.googleapis.com/qpxExpress/v1/trips/search?key=' + apiKey;

function formatDate(d) {
  return d.getYear() + '-' + (d.getMonth() + 1) + '-' + d.getDate();
}

var Flight = (function() {
  function Flight(departDate, returnDate, origin, destination) {
    this.origin = origin;
    this.destination = destination;
    this.departDate = formatDate(departDate);
    this.returnDate = formatDate(returnDate);
    this.props = PropertiesService.getDocumentProperties();
    this.key = origin + destination + this.departDate + this.returnDate + 'v2';
  }

  Flight.prototype.cost = function() {
    return getInfo.call(this).cost;
  }

  Flight.prototype.flightNumber = function() {
    return getInfo.call(this).flightNumber;
  }

  function getInfo() {
    var stored = this.props.getProperty(this.key);
    if (stored != null) {
      return JSON.parse(stored);
    }
    return fetchInfo.call(this);
  }

  function fetchInfo() {
    var response = JSON.parse(UrlFetchApp.fetch(url, options.call(this)));
    var cost = response.trips.tripOption[0].saleTotal;
    var carrier = response.trips.tripOption[0].slice[0].segment[0].flight.carrier;
    var number = response.trips.tripOption[0].slice[0].segment[0].flight.number;

    var data = {
      cost: cost,
      flightNumber: carrier + number
    }
    this.props.setProperty(this.key, JSON.stringify(data));
    return data;
  }

  function options() {
    return {
      method: 'post',
      payload: JSON.stringify(payload.call(this)),
      contentType: 'application/json'
    };
  }

  function payload() {
    return {
      request: {
        slice: [{
          origin: this.origin,
            destination: this.destination,
            date: this.departDate
          },
          {
          origin: this.destination,
          destination: this.origin,
          date: this.returnDate
          }
      ],
      passengers: {
          adultCount: 1,
          infantInLapCount: 0,
          infantInSeatCount: 0,
          childCount: 0,
          seniorCount: 0
        },
        solutions: 1,
        refundable: false
      }
    };
  }

  return Flight;
})();

function FLIGHTCOST(departDate, returnDate, origin, destination) {
  f = new Flight(departDate, returnDate, origin, destination);
  return f.cost();
}

function FLIGHTNUMBER(departDate, returnDate, origin, destination) {
  f = new Flight(departDate, returnDate, origin, destination);
  return f.flightNumber();
}
```
If you don't want to persist the values, you can use Google's `CacheService` instead of `PropertiesService`, to
store results for up to 6 hours. I don't care about this updating in real time, so I opted to just persist it
to avoid rate limits on the API.
