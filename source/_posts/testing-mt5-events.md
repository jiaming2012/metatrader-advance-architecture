---
title: testing-mt5-events
date: 2019-01-20 10:45:03
tags:
---
1. Create a new Unittest folder inside Scripts/

Scripts
 | Examples
 | Unittest
   | events
     | bands.mq5

2. Lets get a failing test!
```
//+------------------------------------------------------------------+
//|                                                        bands.mq5 |
//|                        Copyright 2018, MetaQuotes Software Corp. |
//|                                             https://www.mql5.com |
//+------------------------------------------------------------------+
#property version   "1.00"

#include <Testing/Unittest.mqh>

class Tests : Unittests
{
 public:
  bool price_crosses_resistance_band_from_below();

  Tests() : Unittests(__FILE__) {};
};

//+------------------------------------------------------------------+
//| Script program start function                                    |
//+------------------------------------------------------------------+
void OnStart()
  {
//---
   Tests test;

   RUN(test.price_crosses_resistance_band_from_below());
  }
//+------------------------------------------------------------------+
bool Tests::price_crosses_resistance_band_from_below(void)
{
 BEGIN_TEST

 FAIL("write the test");

 END_TEST
}
```
In the terminal, drag the script onto a chart and open the output in your text editor.
MQL5\Files\Unittest\Logs\bands.txt
```
----------------------------------------------------
Starting bands.mq5 at local time 2019.01.20 16:09:11
----------------------------------------------------
Begin: price_crosses_resistance_band_from_below
Failed: Unittests::fail::Tests::price_crosses_resistance_band_from_below, line=35
msg=write the test
----------------------------------------------------
assertions passed: 0
assertions failed: 1
----------------------------------------------------
TEST FAILED !!
time took: 0.623712 second(s)
Finished test at local time 2019.01.20 16:09:12
----------------------------------------------------
```

node-feed.mq5
```
//+------------------------------------------------------------------+
//|                                                  nodejs-feed.mq5 |
//|                        Copyright 2018, MetaQuotes Software Corp. |
//|                                             https://www.mql5.com |
//+------------------------------------------------------------------+
#property copyright "Copyright 2018, MetaQuotes Software Corp."
#property link      "https://www.mql5.com"
#property version   "1.00"

#include <Files/FilePipe.mqh>

CFilePipe client;

string SUBKEY;
string SYMBOL;

input bool TICKDATA=true;
input bool EVENTS=true;

enum ENUM_CUSTOM_CHART_EVENT {
  NULL_EVENT,
  CUSTOM_CHARTEVENT_SUPPORT_LINE_DRAG=1,
  CUSTOM_CHARTEVENT_RESISTANCE_LINE_DRAG,
  CUSTOM_CHARTEVENT_SUPPORT_LINE_CROSSED,
  CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED
};

//Variables declared outside of a function are declared globally
//Global variables are stored in permanent memory. Hence, there values
//persist between function calls. Furthermore, global variables can be access
//in all functions. It is good coding practice to limit the use of global variables
//or not use them at all
bool glbSupportLineCrossed=false;
bool glbResistanceLineCrossed=false;

//+------------------------------------------------------------------+
//| Expert initialization function                                   |
//+------------------------------------------------------------------+
int OnInit()
  {
//---
    MathSrand(GetTickCount());

    //SUBKEY is used by nodejs named-pipes module for keeping track
    //of multiple client connections
    SUBKEY = (string)((MathRand()*1000) + ChartID());

    //Store chart symbol in variable to prevent repeated function calls   
    SYMBOL = Symbol();

    //Keep checking for nodejs server connection  
    while(!IsStopped())
     {
      if(client.Open("\\\\.\\pipe\\MQL5.Pipe.Server", FILE_READ|FILE_WRITE|FILE_BIN) != INVALID_HANDLE)
       break;

      Sleep(250);
     }

    Print("[DEBUG] Data feed connected to pipe");

    //Initial one-way handshake to server sets up client connection
    string payload = JSONPayload("npmsg:connect-to-subkey", SUBKEY, SUBKEY);
    int size=StringLen(payload);
    if(!client.WriteString(payload, size))
     {
      Print("[HIGH] initial handshake failed");
     }
//---
   return(INIT_SUCCEEDED);
  }
//+------------------------------------------------------------------+
//| Expert deinitialization function                                 |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
  {
//---

  }
//+------------------------------------------------------------------+
//| Expert tick function                                             |
//+------------------------------------------------------------------+
void OnTick()
  {
//---
   MqlTick tick;

   //Store tick data in MqlTick struct
   SymbolInfoTick(SYMBOL, tick);

   if(TICKDATA)   //Allows disabling sending of tick data to nodejs
    {
     //Send tick data to nodejs
     string payload = SYMBOL + "," + (string)tick.bid + "," + (string)tick.ask;
     string json = JSONPayload("tick", payload, SUBKEY);
     int size=StringLen(json);
     if(!client.WriteString(json, size))
      {
       Print("[HIGH] sending tick failed");
      }
    }

   if(EVENTS)  //Allows disabling of computing ChartEvents
    {
     double price;
     color clr;

     //--- Check if price crosses MySupportLine
      price=ObjectGetDouble(0, "MySupportLine", OBJPROP_PRICE);
      clr=(color)ObjectGetInteger(0, "MySupportLine", OBJPROP_COLOR);

     //Price will be zero if support line is not drawn on chart
     // and the color will be set to white if the line is triggered
     if(price > 0 && clr != clrWhite)   
      {
       if(tick.bid < price)
        {
         //When support line is crossed, change the support line to white,
         //make it unselectable, send a custom event to nodejs,
         //and log a INFO message
         ObjectSetInteger(0, "MySupportLine", OBJPROP_COLOR, clrWhite);
         ObjectSetInteger(0, "MySupportLine", OBJPROP_SELECTABLE, false);
         EventChartCustom(0, CUSTOM_CHARTEVENT_SUPPORT_LINE_CROSSED, 0, price, "");     
         Print("[INFO] support line crossed @ " + (string)price);
        }
       }

    // --- Check if price crosses MyResistanceLine
     price=ObjectGetDouble(0, "MyResistanceLine", OBJPROP_PRICE);
     clr=(color)ObjectGetInteger(0, "MyResistanceLine", OBJPROP_COLOR);

    //Price will be zero if resistance line is not drawn on chart
    //and the color will be set to white if the line is triggered
    if(price > 0 && clr != clrWhite)
     {
      if(tick.bid > price)
       {
        //When resistance line is crossed, change the resistance line to white,
        //make it unselectable, send a custom event to nodejs,
        //and log a INFO message
        glbResistanceLineCrossed=true;
        ObjectSetInteger(0, "MyResistanceLine", OBJPROP_COLOR, clrWhite);
        ObjectSetInteger(0, "MyResistanceLine", OBJPROP_SELECTABLE, false);
        EventChartCustom(0, CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED, 0, price, "");

        Print("[INFO] resistance line crossed @ " + (string)price);
       }
     }
   }
  }
//+------------------------------------------------------------------+
//| Trade function                                                   |
//+------------------------------------------------------------------+
void OnTrade()
  {
//---

  }
//+------------------------------------------------------------------+
//| TradeTransaction function                                        |
//+------------------------------------------------------------------+
void OnTradeTransaction(const MqlTradeTransaction& trans,
                        const MqlTradeRequest& request,
                        const MqlTradeResult& result)
  {
//---

  }
//+------------------------------------------------------------------+
//| ChartEvent function                                              |
//+------------------------------------------------------------------+
void OnChartEvent(const int id,
                  const long &lparam,
                  const double &dparam,
                  const string &sparam)
  {
//---
   if(EVENTS)  //Allows disabling of sending ChartEvents to nodejs
   {
    //Set the name of the event: use MT5 name if the event was a native MT5 event;
    //otherwise, use the enum name set by the user
    string EVENT_NAME;
    if (id < CHARTEVENT_CUSTOM) {
     EVENT_NAME = EnumToString((ENUM_CHART_EVENT)id);
    } else {
     int custom_id = id - CHARTEVENT_CUSTOM;
     EVENT_NAME = EnumToString((ENUM_CUSTOM_CHART_EVENT)custom_id);
    }

    //Check for custom events
    if(id==CHARTEVENT_OBJECT_DRAG) {
     string obj_name=sparam;
     double price=ObjectGetDouble(0, obj_name, OBJPROP_PRICE);
     ENUM_CUSTOM_CHART_EVENT e=NULL_EVENT;

     //MySupportLine was dragged
     if (obj_name=="MySupportLine") {
      e=CUSTOM_CHARTEVENT_SUPPORT_LINE_DRAG;
     }

     //MyResistanceLine was dragged
     if (obj_name=="MyResistanceLine") {
      e=CUSTOM_CHARTEVENT_RESISTANCE_LINE_DRAG;
     }

     //Send custom event
     if (e!=NULL_EVENT)
      EventChartCustom(0, (ushort)e, 0, price, "");
    }

    //Send the chart event to nodejs    
    string payload = EVENT_NAME + "," + SYMBOL + "," + (string)lparam + "," + (string)dparam + "," + sparam;
    string json = JSONPayload("chart", payload, SUBKEY);
    int size=StringLen(json);
    if(!client.WriteString(json, size))
     {
      Print("[HIGH] sending tick failed");
     }
   }
  }
//+------------------------------------------------------------------+
string JSONPayload(string event, string args, string subkey)
{
 return "{\"event\":\"" + event + "\",\"arguments\":[\"" + args + "\"],\"subKey\":\"" + subkey + "\"}";
}
```
Now its time to refactor. We need to isolate our custom events in functions so that they can be both tests and easily inserted into our main application. Make a new include file
Include\Events\bands.mqh

nodejs-feed.mqh
```
if(EVENTS)  //Allows disabling of computing ChartEvents
    {
     double price;
     color clr;

     //--- Check if price crosses MySupportLine
      price=ObjectGetDouble(0, "MySupportLine", OBJPROP_PRICE);
      clr=(color)ObjectGetInteger(0, "MySupportLine", OBJPROP_COLOR);

     //Price will be zero if support line is not drawn on chart
     // and the color will be set to white if the line is triggered
     if(price > 0 && clr != clrWhite)   
      {
       if(tick.bid < price)
        {
         //When support line is crossed, change the support line to white,
         //make it unselectable, send a custom event to nodejs,
         //and log a INFO message
         ObjectSetInteger(0, "MySupportLine", OBJPROP_COLOR, clrWhite);
         ObjectSetInteger(0, "MySupportLine", OBJPROP_SELECTABLE, false);
         EventChartCustom(0, CUSTOM_CHARTEVENT_SUPPORT_LINE_CROSSED, 0, price, "");     
         Print("[INFO] support line crossed @ " + (string)price);
        }
       }

    // --- Check if price crosses MyResistanceLine
     price=ObjectGetDouble(0, "MyResistanceLine", OBJPROP_PRICE);
     clr=(color)ObjectGetInteger(0, "MyResistanceLine", OBJPROP_COLOR);

    //Price will be zero if resistance line is not drawn on chart
    //and the color will be set to white if the line is triggered
    if(price > 0 && clr != clrWhite)
     {
      if(tick.bid > price)
       {
        //When resistance line is crossed, change the resistance line to white,
        //make it unselectable, send a custom event to nodejs,
        //and log a INFO message
        glbResistanceLineCrossed=true;
        ObjectSetInteger(0, "MyResistanceLine", OBJPROP_COLOR, clrWhite);
        ObjectSetInteger(0, "MyResistanceLine", OBJPROP_SELECTABLE, false);
        EventChartCustom(0, CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED, 0, price, "");

        Print("[INFO] resistance line crossed @ " + (string)price);
       }
     }
   }
```
In OnTick(), cut the code between if(EVENTS) { } and paste it into bands.mqh

Create a file, enums.mqh, inside of Events/


Cut ENUM_CUSTOM_CHART_EVENT from nodejs-feed.mq5 and paste it into enums.mqh. Then import enums into nodejs-feed.mq5 and bands.mqh
```
#include <Events/enums.mqh>
```
In bands.mqh set up your event return function:
```
#include <Events/enums.mqh>

int priceCrossesResistanceLineFromBelow(const int resistance_line,
                                        const double price)
{

 return NULL_EVENT;
}
```

Include band.mqh into Unittest\events\bands.mq5

Edit enums.mqh
```
enum CUSTOM_CHART_EVENT {
  NULL_EVENT,
  CUSTOM_CHARTEVENT_SUPPORT_LINE_DRAG=1,
  CUSTOM_CHARTEVENT_RESISTANCE_LINE_DRAG,
  CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED_FROM_BELOW,
  CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED_FROM_ABOVE,
  CUSTOM_CHARTEVENT_SUPPORT_LINE_CROSSED_FROM_BELOW,
  CUSTOM_CHARTEVENT_SUPPORT_LINE_CROSSED_FROM_ABOVE
};
```
Now that we have refactored our event into an event function. Lets write a simple test to test it.
```
bool Tests::price_crosses_resistance_band_from_below(void)
{
 BEGIN_TEST

 double line = 1.5;
 double price_feed[] = {1.3, 1.7};
 int result;

 // Initial tick
 result = priceCrossesResistanceLineFromBelow(line, price_feed[0]);
 ASSERT_EQUALS((CUSTOM_CHART_EVENT)result, NULL_EVENT);

 // Next tick
 result = priceCrossesResistanceLineFromBelow(line, price_feed[1]);
 ASSERT_EQUALS((CUSTOM_CHART_EVENT)result, CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED_FROM_BELOW);

 END_TEST
}
```
Our second assertion fails.
Output from our test:
```
----------------------------------------------------
Starting bands.mq5 at local time 2019.01.20 19:03:49
----------------------------------------------------
Begin: price_crosses_resistance_band_from_below
Failed: Unittests::assertEquals<CUSTOM_CHART_EVENT>::Tests::price_crosses_resistance_band_from_below, line=44
x = NULL_EVENT
y = CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED_FROM_BELOW
----------------------------------------------------
assertions passed: 1
assertions failed: 1
----------------------------------------------------
TEST FAILED !!
time took: 0.117035 second(s)
Finished test at local time 2019.01.20 19:03:50
----------------------------------------------------
```
Now lets make the test pass.
bands.mqh
```
int priceCrossesResistanceLineFromBelow(const double resistance_line,
                                        const double price)
{
 static double last_price;

 if(price > resistance_line)
 {
  if(last_price > 0 && last_price < resistance_line) {
   return CUSTOM_CHARTEVENT_RESISTANCE_LINE_CROSSED_FROM_BELOW;
  }
 }

 last_price = price;
 return NULL_EVENT;
}
```
Variables declared with the *static* modifier live throughout the entire life of the program. If we omitted this variable. last_price would be set to zero on each function call.

Our tests should now pass.
```
----------------------------------------------------
Starting bands.mq5 at local time 2019.01.20 19:18:56
----------------------------------------------------
Begin: price_crosses_resistance_band_from_below
[OK] Passed price_crosses_resistance_band_from_below in 0.012167 second(s)
----------------------------------------------------
Passed 1 test(s)
----------------------------------------------------
assertions passed: 4
assertions failed: 0
----------------------------------------------------
ALL TESTS PASSED !!!
time took: 0.126409 second(s)
Finished test at local time 2019.01.20 19:18:56
----------------------------------------------------
```
