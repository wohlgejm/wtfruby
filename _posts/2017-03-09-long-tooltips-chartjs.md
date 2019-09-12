---
layout: post
comments: true
title: Hacking multi-line tooltips in Chart.js
date: 2017-03-09 20:20:30
categories: javascript chartjs
short_description: Creating multi-line tooltips in Chart.js
---

[Chart.js](http://www.chartjs.org/) is an awesome javascript library with an API that can handle almost any
requirements. However, because Chart.js uses canvas to draw the charts, the tooltips can be frustrating.
The tooltips cannot render outside of the canvas and if you have very long data, this can cause the tooltip
on the chart to become useless.

Here is a very hacky workaround for putting in line breaks in your tooltips. Chart.js does not supporting
rendering html inside the tooltip, so it's not as easy as throwing in a `<br>`. This code will place the first line as
the `label` and subsequent lines as the `afterLabel` option. `afterLabel` accepts an array of strings, which
it will break onto new lines.

```javascript
const maxTooltipLength = 50;

let wordsToArray = function(words) {
   let lines = [];
   let str = '';
   words.forEach(function (word) {
     if ((str.length + word.length + 1) <= maxTooltipLength) {
       str += word + ' ';
     } else {
       lines.push(str);
       str = word + ' ';
     }
   });
   lines.push(str);
   return lines;
}

let breakLabels = fucntion(tooltipItems, data) {
   let label = data.labels[tooltipItems.index];
   if (label.length <= maxTooltipLength) { return [ label ] }
   let words = label.split(' ');
   return wordsToArray(words);
}

let firstLabel = function (tooltipItems, data) {
  return breakLabels(tooltipItems, data)[0];
}

let otherLabels = function (tooltipItems, data) {
  return breakLabels(tooltipItems, data).slice(1);
}

let options = {
  tooltips: {
    callbacks: {
      label: firstLabel.bind(this),
      afterLabel: otherLabels.bind(this)
    }
  }
}
```

Binding the callback is necessary otherwise, `this` will the `Chart` instance.
