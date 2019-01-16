---
title: environment (part 1)
date: 2018-12-21 12:47:06
tags:
---
# Environment is your key to sanity

I like to classify programs that I write into two broad categories: scripts and applications. Scripts are programs that do one or a few specific things, such as set up my testing environment while I get some coffee. Or pull yesterday's stock prices from Google Finance into a csv file. One nice property of scripts is that they usually run once and are done. They can usually be decomposed into a series of linear steps. If a script fails in the middle step 6, you usually can just display an error stop the script, or skip it and go straight to step 7. Applications on the other hand, usually run continuously. Their programming logic is often much more complex, and hence, logging and error handling become much more important. Adding new features often silently break old features; hence, stringed testing becomes important. Setting up a solid environment pays big dividends down the line: not only will you have more confident that your code works the way you expect it to, you'll spend more time developing new features and testing new ideas, and less time debugging (which can be a significant chunk of your time for poorly managed projects).

Before diving straight into the code, lets take some time to sketch out our desired project. How large will it be? This is a tricky question as oftentimes small code bases slowly grow from few hundred lines of code to a few thousand lines. Event driven programing is a paradigms to help us reduce complexity and write clean code.

Will our algorithm run 24/7 or only a few hours per day? These types of questions will help us set up our environment.

In the trading domain, I've found the following to be essential: robust logging, a trading notification system and IT monitoring system.

## 1. Chose Your Physical Hardware

One of the greatest strengths of the MT5 platform is also its greatest weakness: it's simplicity. Many trading ideas start as simple ideas that in your mind can be fully implemented in a few lines of code. As the market slowly evolves, so does your trading algos. Without the proper tools in place, refactoring your code becomes a burden and you find yourself unable to adjust with the market. Hence, I provide two guides: programming in the language of your choice or creating programs wholly contained inside MT5.

Another MT5 flaw is that it is only natively supported on Windows. You can find unofficial mac and linux support via wine. If you choose to use wine, you can find installation instruction here:
- On Mac, https://www.metatrader5.com/en/terminal/help/start_advanced/install_mac
- On Linux, https://www.metatrader5.com/en/terminal/help/start_advanced/install_linux

Another option is to use the community edition of VirtualBox. It's free. And installation is pretty straightforward. Since, I'm developing this guide on a Mac, this is the option that I will choose. You will also need to find a Windows image if you also choose this path.

### VirtualBox
To set up MT4 on VirtualBox, follow the installation steps below:
1. Download VirtualBox on your computer using the following link <https://www.virtualbox.org/wiki/Downloads>.
2. Run the installer
    1. I selected 2GB RAM (though I recommend 4GB or 8GB if you have enough resources, since MT4 terminals are very memory intensive)
    2. 50 GB of disk space
    3. Dynamically allocated

## 2. Chose a FX Broker
Metatrader (MT) uses a broker licensing and white label model as it's revenue stream. Hence, MT traders do not pay any direct fees to use the platform (they are passed along in the form of price feed markups and commissions).

I'm choosing to use <https://evolve.markets> as my broker. Setting up a brokerage account takes just a few seconds. Furthermore, they accept crypto deposits, so funding a live accounts takes a few minutes as opposed to a few days. However, feel free to choose the broker of your choice as the MT5 platform is mostly the same across different brokers.

{% asset_img MT4_Terminal.png "MT4 terminal running on Virtualbox" %}
