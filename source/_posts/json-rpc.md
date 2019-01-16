---
title: json-rpc
date: 2018-12-30 16:45:53
tags:
---
# Windows VM
Open the ports
1. Open Windows Defender Firewall
2. Advanced Settings
3. Inbound Rules
4. New Rule
5. Select 'Port'. 'TCP'
6. Specified local ports: 8082-8083
7. Select 'Allow the connection'

Make sure windows is discoverable by outside networks.

Run cmd and find your IPv4 address

{% asset_img "ipconfig-all.png" "ipconfig /all output" %}

Metatrader only allows webrequests to ports 80 and 443, using the http:// and https:// prefixes respectively. Specifying a custom port, e.g. http://localhost:8000, leads to a rather cryptic error code =5203 (ERR_WEBREQUEST_REQUEST_FAILED)

EventChartCustom() is an async MQL5 function: after it is called the user-defined event is placed into a queue, prompting the special OnChartEvent() function to be subsequently called with parameters passed from EventChartCustom().

Benefits of event oriented programming: scalability, clean code, modifying code base

Show video in the beginning of placing a strategy in JavaScript

Show the process of adding a new MT5 function

Show the process of adding a new MT5 event
