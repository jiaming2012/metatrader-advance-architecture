---
title: environment (part 2)
date: 2018-12-23 12:17:26
tags:
---
Git
ELK stack: elasticsearch, logstash, kibana
Filebeat
Ubuntu 14.04
Nodejs

creating-a-rest-api


In order to extend Metatrader functionality, we will create a rest API. This will allow us to make our accounts accessible over the web, as well as, give us the option to stop using MetaQuotes entirely.

## Install your favorite text editor
If you don't have a favorite, you can choose my favorite https://atom.io

## Install Git
- go to https://git-scm.com/downloads
- run the installer and follow the installer's instructions
  - I recommend selecting "Use nano by default" to use the more user-friendly nano text editor when prompted to choose your default editor
  - Choose "Git from the command line and also from 3rd-party software"
  - Choose "Use the OpenSSL library"
  - I recommend choosing "Checkout as-is, commit as-is" when prompted to choose line ending configurations
  - Choose "Use MinTTY"
  - Use the default extra options configuration

## Install nodejs
We are going to be installing the windows version of node on the same server as MT5.

- go to https://nodejs.org/en/
- run the installer and follow instructions to install the current version (currently node-v10.14.2-x64.msi)

## Create a webserver
Open Git Bash and go to your current directory
``` bash
$ cd ~
```

Make a new projects folder
``` bash
$ mkdir projects
```

Start a new node project
``` bash
$ cd projects/
$ npm init
```
Enter a project name when prompted (e.g. mt5webserver) and click <ENTER>, leaving the remaining fields blank

Install a webserver
``` bash
$ npm install express
```
**ADD MORE: express should have been added to the package.json file in your node project directory**

Create a file
``` bash
$ touch index.js
```

Open your favorite text editor and write the following code:
``` node

```

## Metatrader
By default, MT5 blocks all web requests. We need to whitelist our localhost in order to communicate with our nodejs server.
{% asset_img mt5_whitelist_ip.png %}

## Virtualbox Manager
- Right-click the running VM and click 'Settings'
- Click the 'Network' table
- Toggle the 'Advanced' tab
- Click the 'Port Forwarding' button
- Click the 'Add a new port forwarding rule' (plus button)
- Enter 8082 for 'Host Port'
- Enter 8082 for 'Guest Port'
- Click ok to save configure changes

{% asset_img virtualbox_port_forwarding.png %}

``` node
const express = require('express');
const app = express();
const PORT = process.env.PORT || 8082;

app.get('/', function(req, res) {
  console.log('get request');
  res.send('get requests business!!!');
})

app.post('/', function(req, res) {
  console.log('post request');
  res.send('post received');
})

app.listen(PORT, function() {
  console.log('listening on port ' + PORT);
})
```

## Git bash

``` bash
$ npm install websocket --save
$ npm install http --save
```

Special installation instructions (from https://www.npmjs.com/package/websocket)
> Note for Windows Users
>
Because there is a small C++ component used for validating UTF-8 data, you will need to install a few other software packages in addition to Node to be able to build this module:

> - Microsoft Visual C++ (this can take some time)
> -- Download the community version
> -- https://visualstudio.microsoft.com/downloads/
> - Python 2.7 (NOT Python 3.x)
> -- https://www.python.org/downloads/windows/

Open port 8083 (similar to opening port 8082 above) on the VirtualBox manager

## Linux Ubuntu 14.04

name: tradeserver

Using the guide below, for command line newcomers, vi is an old-school text editor. It had two modes: a command mode and an input mode. To complete this guide, you only require two basic commands:
1. i -> activates insert mode (which allows you to input text into the editor)
2. ESC :wq -> saves and quits the edited file

https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-ubuntu-14-04

sudo /opt/logstash/bin/plugin install logstash-filter-mutate (stop service first)

For debugging purposes, the following configuration allows you view the output of the parsed logged data after it passes through logstash in the console:

/etc/logstash/conf.d/30-elasticsearch-output.conf
output {
  stdout { codec => rubydebug }
}

Stop the logstash service and run it from the console (so that you can easily view its output from the rubydebug codec)

If you see error message:
> Unknown setting 'copy' for mutate {:level=>:error}
> Error: Something is wrong with your configuration. {:level=>:error}

Update the logstash-filter-mutate plugin to version 3.3.4 or above:

``` bash
$ sudo /opt/logstash/bin/plugin update logstash-filter-mutate
```

``` bash
$ sudo service logstash stop
$ /opt/logstash/bin/logstash -f /etc/logstash/conf.d/ --verbose -v
```

Once logbeats detects that data was appended to our MT5 log file. The new data is collected, along with other system data and metadata. The data is then sent to logstash, which applies our filters before outputting the data to its destination.


Now lets check the syntax of our configuration file's syntax:
``` bash
$ /opt/logstash/bin/logstash -f /etc/logstash/conf.d/ -t
Configuration OK
```

Sample output explanation:
When logstash receives input from logbeats, it checks that the field.platform attribute reads "mt5".

Since, MT5 puts the date of the message as the filename, we have to get it. Luckily logstash's configuration language provides a set of useful function for extracting data from input. First, add_field creates a new field path and sets it's value to the value of source. Next we use the gsub function of the mutate plugin to strip the null characters and tabs (\u0000) from that is found in the incomming MT5 message. We also replace the windows forward slashes with unix backslashes (the reason I did this is because the split subsequent split function wasn't working with the forward slashes). Next we split the string on the backslash character, which creates an array from the path. We then grab the message data by splitting the '.' on the 9th array-element (indexed from 0).

filter {
  if [fields][platform] == "mt5" {
    mutate {
      add_field => { "path" => "%{source}" }
    }
    mutate {
      gsub => [
        "message", "[\u0000]", "",
        "message", "\t", " ",
        "path", "[\\]", "/"
      ]
      split => { "path" => "/" }
      split => { "path[9]" => "." }
    }
    grok {
      match => { "message" => "%{WORD} %{NONN$
    }
    mutate {
      merge => { "msg_tstamp" => "path[9][0]"$
    }
    mutate {
      join => { "msg_tstamp" => "," }
    }
    date {
      match => [ "msg_tstamp", "HH:mm:ss.SSS,$
      target => "msg_timestamp"
      remove_field => ["msg_tstamp"]
    }
  }
}

We can remove the derived fields "msg_tstamp" and "path" from our data after we've extracted the data, so that these fields won't be saved and take up unnecessary space. Add the following line to the last line INSIDE the date function

```
remove_field => ["msg_tstamp"]
```

Interesting note: add_field and remove_field are **inherited** plugin functions. Hence, they can be called from any logstash plugin.

Sample output:
{
          "message" => "JN 0 02:24:49.812 Network '13499': disconnected from EvolveMarkets-MT5 Demo Server\r",
         "@version" => "1",
       "@timestamp" => "2018-12-29T02:24:57.486Z",
       "prospector" => {
        "type" => "log"
    },
             "host" => {
                "name" => "DESKTOP-676N2NQ",
        "architecture" => "x86_64",
                  "os" => {
            "platform" => "windows",
             "version" => "10.0",
              "family" => "windows",
               "build" => "16299.847"
        },
                  "id" => "946dc936-9c1d-4de5-878e-024ad2a1d8a1"
    },
             "beat" => {
            "name" => "DESKTOP-676N2NQ",
        "hostname" => "DESKTOP-676N2NQ",
         "version" => "6.5.4"
    },
           "source" => "c:\\Users\\Jamal\\AppData\\Roaming\\MetaQuotes\\Terminal\\AB7E0B6EC8E813DD39C073841806A917\\Logs\\20181229.log",
           "offset" => 6499,
            "input" => {
        "type" => "log"
    },
           "fields" => {
        "platform" => "mt5"
    },
             "tags" => [
        [0] "beats_input_codec_plain_applied"
    ],
             "path" => [
        [0] "c:",
        [1] "Users",
        [2] "Jamal",
        [3] "AppData",
        [4] "Roaming",
        [5] "MetaQuotes",
        [6] "Terminal",
        [7] "AB7E0B6EC8E813DD39C073841806A917",
        [8] "Logs",
        [9] [
            [0] "20181229",
            [1] "log"
        ]
    ],
       "msg_source" => "Network",
              "msg" => "'13499': disconnected from EvolveMarkets-MT5 Demo Server",
    "msg_timestamp" => "2018-12-29T07:24:49.812Z"
}

In our example case, the output is our console. Lets change the output back to elasticsearch so that we will be able to view the data in kibana.

Lets add the experts path after the journal path so that we can receive messages related to our EAs.

filebeat.inputs:
- type: log
  enabled: true

  paths:
    # MT5 Journal path
    - c:\Users\Jamal\AppData\Roaming\MetaQuotes\Terminal\AB7E0B6EC8E813DD39C073841806A917\Logs\*.log
    # MT5 Expert path
    - c:\Users\Jamal\AppData\Roaming\MetaQuotes\Terminal\AB7E0B6EC8E813DD39C073841806A917\MQL5\Logs\*.log

MT5 logger

Ideally, if we wanted to log a statement. We'd use the MT5 native Print() function, which would write it to the log file and be picked up by FileBeat. Unfortunately, Metatrader does not flush logs from the console to the log file automatically; it only flushes periodic and when the terminal is closed. As a result, we will create a custom logger that flushes to special directory after each write.

## Windows
Install filebeat
https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation.html

Find path to MT5 terminal logs
1. Open terminal
2. Right-click on 'Journal' tab
3. Select 'Open' from the context menu
(make video)

Step 2
Open favorite text editor as Administrator (show vid)
C:\Program Files\Filebeat\filebeat.yml

* Disabled SSL. Reenable?

Step 3
Disable output to elasticsearch and enable output to logstash

https://www.elastic.co/guide/en/beats/filebeat/current/config-filebeat-logstash.html

Step 4
Windows installation instructions
https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation.html

Start Filebeat
PS C:\Program Files\Filebeat> Start-Service filebeat
