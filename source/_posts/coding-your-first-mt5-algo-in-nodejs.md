---
title: coding-your-first-mt5-algo-in-nodejs
date: 2019-01-09 12:04:56
tags:
---

Now lets get to the fun stuff: creating a live trading bot! For this tutorial, I will create a basic bot that buys when the price touches a MT5 support band and sells short when the price touches a MT5 resistance band.

This algo would be classified as a semi-automatic trading algo since we are going to require manual placement of the support and resistance bands in MT5. When I developing new algorithms, I am a big fan of keeping manual components for the sake of rapid prototyping in the initial development phases; and then, iteratively automating away the manual processes as patterns become more obvious. We will also follow the test driven development (TDD) paradigm.

Lets open our terminals

``` bash
$ mkdir -p trading/mt5/bands
$ cd trading/mt5/bands
$ npm init
```

Tap enter for all npm prompts.

``` bash
$ npm install --save-dev mocha chai
$ mkdir tests/
$ touch tests/unit.js
```

Replace the scripts section in **package.json**
``` javascript
"scripts": {
    "test": "mocha"
  },
```

Open tests/unit.js in your favorite editor:

``` javascript
const chai = require('chai');
const assert = chai.assert;
const MockTradingInterface = require('../../mock/index.js');
const Strategy = require('../index.js');

beforeEach(function() {
  // Create an instance of our strategy and pass it a mock interface
  // for simulating the placing of trades
  this.interface = new MockTradingInterface();
  this.strategy = new Strategy(this.interface);
})

describe('open trade', function() {
  it('place buy when receiving cross resistance band event', function() {
    this.strategy.onChartEvent([
      'CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED'
    ]);

    const pos = this.interface.getPosition();
    assert.equal(pos['EURUSD'], 1.0);
  })
})
```

To run this test on the command line, go to the project directory (which is **bands/** in our case) and run:

``` bash
$ npm run test
```

You should see the following output:

```
> bands@1.0.0 test /Users/admin/projects/trading/mt5/bands
> mocha



  open trade
    1) place buy when receiving cross resistance band event


  0 passing (13ms)
  1 failing

  1) open trade
       place buy when receiving cross resistance band event:
     AssertionError: expected undefined to equal 1
      at Context.<anonymous> (test/unit.js:20:12)



npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! bands@1.0.0 test: `mocha`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the bands@1.0.0 test script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/admin/.npm/_logs/2019-01-14T11_55_40_238Z-debug.log
```

Congratulations! You just passed first step in a TDD paradigm: create a failing test. Carefully reading the output from our test: our error is caused by line 20 in our unit test.

```
AssertionError: expected undefined to equal 1
 at Context.<anonymous> (test/unit.js:20:12)
```

When we search 'EURUSD' in our positions, it is returning undefined as getPosition() returns an empty object {}, since no trade was placed by our algo. Now lets move on to step \#2: write the minimum code to make the test pass.

``` javascript

class Strategy {
  constructor(tradeInterface, symbol, volume) {
    this.interface = tradeInterface;
    this.symbol = symbol;
    this.volume = volume;
  }

  onChartEvent(events) {
    for (let i in events) {
      if (events[i] == 'CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED') {
        this.interface.send(this.symbol, this.volume, 'buy');
      }
    }
  }
}

module.exports = Strategy
```
Lets run our test again:
```
> mocha



  open trade
    ✓ place buy when receiving cross resistance band event


  1 passing (10ms)
```
Great!!! We have one passing test! Now, lets go through a similar process for selling once we break support.

``` javascript
it('place sell when receiving cross below resistance support band', function() {
    this.strategy.onChartEvent([
      'CUSTOM_CHARTEVENT_SUPPORT_LINE_CROSSED'
    ]);

    const pos = this.interface.getPosition();
    assert.equal(pos['EURUSD'], -1.0);
  })
```

``` bash
$ npm run test
```
output:
```
> mocha



  open trade
    ✓ place buy when receiving cross above resistance band
    1) place sell when receiving cross below resistance support band


  1 passing (13ms)
  1 failing

  1) open trade
       place sell when receiving cross below resistance support band:
     AssertionError: expected undefined to equal -1
      at Context.<anonymous> (test/unit.js:29:12)
```
Change the inside of our for loop:
``` javascript
if (events[i] == 'CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED') {
  this.interface.send(this.symbol, this.volume, 'buy');
}

if (events[i] == 'CUSTOM_CHARTEVENT_SUPPORT_LINE_CROSSED') {
  this.interface.send(this.symbol, this.volume, 'sell');
}
```
Run the test:
``` bash
$ npm run test
```
output:
```
> bands@1.0.0 test /Users/admin/projects/trading/mt5/bands
> mocha



  open trade
    ✓ place buy when receiving cross above resistance band
    ✓ place sell when receiving cross below resistance support band


  2 passing (10ms)
```

``` bash
jiamingeMacBook:bands admin$ npm run test

> bands@1.0.0 test /Users/admin/projects/trading/mt5/bands
> mocha

module.js:549
    throw err;
    ^

Error: Cannot find module '../index.js'
```

Node is telling us that we don't have a strategy file. Lets create one!

``` bash
$ touch ../bands/index.js
```

Let's run our tests again:
``` bash
jiamingeMacBook:bands admin$ npm run test

> bands@1.0.0 test /Users/admin/projects/trading/mt5/bands
> mocha



  initialize
WebSocket client connected to remote node host
Pinging MT5 server to check connection ...
    ✓ no trades opened at first iteration


  1 passing (22ms)

WebSocket client disconnected from remote node host
```

Our tests are passing again. Now its time to write more tests.

``` javascript
describe('initialize', function() {
  it('no trades opened at first iteration', function() {
    result = strategy.execute({
      events: [],
      tick: null
    })

    assert.equal(result, null)
  });
})
```

First we will test that the strategy initializes as we expect:

``` javascript
it('successfully initializes when first tick is inside bands', function() {
    strategy.set({
      lowerBand: 1.0,
      upperBand: 1.5
    })

    var result = strategy.initialize({
      events: [],
      tick: 1.3
    })

    assert.equal(result.length, 1);
    assert.equal(result[0], 'INIT_SUCCESS');
  })
```

We will write the minimum to make this test pass.

mt5/bands/index.js

``` javascript
function execute({events, ticks}) {
  return null
}

module.exports = {
  execute
}
```
