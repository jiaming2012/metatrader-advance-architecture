---
title: functional-test
date: 2019-01-14 23:26:17
tags:
---
In contrast to unit tests, whose sole purpose is to test individual components of your program. Functional tests are macro tests, whose job is to test that our program do what we intend for them to do. I like to think of functional tests as telling a story.

Oscillating Market Simulation:
{% asset_img functional_test_sketch.jpg "Oscillating market simulation sketch"%}

Our story goes like this:
1. Price moves from between the bands and breaks the resistance line -> A buy trade is opened.
2. Price retraces back in between the bands -> The buy trade is closed.
3. Price reverses lower and breaks support line -> A sell trade is opened.
4. Price retraces back in between the bands -> The sell trade is closed.

** Note that we remove the beforeEach hook in our functional tests: we want the state of our program to persist as the story plays out.

First, we test the first bullet point in our story:

``` javascript
const chai = require('chai');
const assert = chai.assert;
const MockTradingInterface = require('../../mock/index.js');
const Strategy = require('../index.js');

describe('oscillating market simulation', function() {
  before(function() {
    this.interface = new MockTradingInterface();
    this.strategy = new Strategy(this.interface, 'EURUSD', 1.0);
  })

  it('Price moves from between the bands and breaks the resistance line', function() {
    this.strategy.onChartEvent([
      'CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED'
    ]);
    const pos = this.interface.getPosition();
    assert.equal(pos['EURUSD'], 1.0);   //A buy trade is opened
  })
})
```

``` bash
$ npm test test/functional.js
```
```
> bands@1.0.0 test /Users/admin/projects/trading/mt5/bands
> mocha "test/functional.js"



  oscillating market simulation
    ✓ Price moves from between the bands and breaks the resistance line


  1 passing (13ms)
```
Now, let add to the second bullet point below the first
``` javascript
it('Price moves from between the bands and breaks the resistance line', function() {
    this.strategy.onChartEvent([
      'CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED'
    ]);
    const pos = this.interface.getPosition();
    assert.equal(pos['EURUSD'], 1.0);   //A buy trade is opened
  })

  it('Price retraces back in between the bands', function() {
    this.strategy.onChartEvent([
      'CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED'
    ]);

    const pos = this.interface.getPosition();
    const eurusd_pos = pos['EURUSD'] == undefined ? 0 : pos['EURUSD'];
    assert.equal(eurusd_pos, 0);    //The buy trade is closed
  })
```
``` bash
$ npm test test/functional.js
```
```
oscillating market simulation
    ✓ Price moves from between the bands and breaks the resistance line
    1) Price retraces back in between the bands


  1 passing (16ms)
  1 failing

  1) oscillating market simulation
       Price retraces back in between the bands:

      AssertionError: expected 2 to equal 0
      + expected - actual

      -2
      +0

      at Context.<anonymous> (test/functional.js:27:12)
```
The second test fails! Our algo is not closing the original trade when the price recrosses the resistance line! This brings up two important points:
1. We need more precise events:
  1. 'CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED_FROM_BELOW'
  2. 'CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED_FROM_ABOVE'
  3. 'CUSTOM_CHARTEVENT_SUPPORT_LINE_CROSSED_FROM_BELOW'
  4. 'CUSTOM_CHARTEVENT_SUPPORT_LINE_CROSSED_FROM_ABOVE'
  
2. Naively, because of the underlying assumption that a price crosses above means price is actually above the line is false. Have we tested this assumption in MT5, it could have been detected early. We cannot blindly trust our MT5 events.
