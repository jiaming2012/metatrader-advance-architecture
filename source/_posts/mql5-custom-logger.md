---
title: mql5-custom-logger
date: 2018-12-29 11:56:31
tags:
---

Open bash terminal:

``` bash
$ cd path/to/MQL5
$ git clone https://github.com/jiaming2012/mql5-helper.git
$ cp mql5-helper/Include/*.mqh Include/
```

Create a new script:
TestLogger.mq5

{% asset_img "test-logger-output-journal.png" "TestLogger.ex5 output" %}

Create next script:
TestLogger2.mq5

without the double-underscore prefix: notice line number for OnStart() and FuncA() are set to the line number of the respective BEGIN macro
{% asset_img "test-logger-output-journal-2.png" "TestLogger2.ex5 INCORRECT output" %}

with the double-underscore prefix: notice the line number for OnStart() and FuncA() match where the function is actually called
{% asset_img "test-logger-output-journal-2b.png" "TestLogger2.ex5 CORRECT output" %}
