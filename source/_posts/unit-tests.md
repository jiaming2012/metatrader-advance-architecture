---
title: unit-tests
date: 2018-12-30 13:31:31
tags:
---
A huge limitation of the Metatrader platform is the absence of native unit test suite. Proper unit testing is paramount for creating robust software, especially when a bug could cost you financial losses. The ability to follow test-driven development paradigms and confidence in code refactoring are two reason I strongly embrace unit testing. I cannot stress latter enough: without proper unit tests in place, changes to a complex code base will result in undetected bugs, ultimately deteriorating your confidence in your ability to make changes to your strategy.

# The Code
I created an elementary MQL5 unit testing framework in [my github](https://github.com/jiaming2012/mql5-helper/tree/master/Include/Testing); despite only implement some basic features, it does get the job done. Feel free to contribute any new features.

TestUnit.mq4

When your run the script, you will see a message appear in the Expert console.

{% asset_img unit-test-console-output.png %}

The output from the unit test is saved in a text file, whose name is derived from the name of the Script from which the unit test was run. Most text editors refresh automatically, making it convenient to view the unit test output after the script is run from the terminal. Let's open it up to see more details as to why the unit test failed.

{% asset_img unit-test-text-editor-output.png %}

Test running a video

{% html5video 'video/mp4' %} {% asset_path mt5-unit-clip-2.mp4 %} {% endhtml5video %}
