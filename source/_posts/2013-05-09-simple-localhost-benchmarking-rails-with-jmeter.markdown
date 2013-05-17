---
layout: post
title: "Simple localhost benchmarking Rails with JMeter on OSX"
date: 2013-05-09 10:15
comments: true
categories: 
---

A little while ago I gave a talk at LRUG on Padrino. Someone asked about 
how I made the benchmarking graphs for the comparison between Padrino and Rails 4 so I thought I'd do a write up.

## Warning

Real benchmarking is hard. There's a big difference between running apache bench on your Macbook and how your site performs in production. The method I describe here will give you slightly better benchmarking than `ab`, and you'll get some pretty graphs but it's really like licking your finger and sticking it up in the air to see which way the breeze is blowing. With that caveat, let's crack on.

## Setting up JMeter

This one is fairly easy. Make sure you have Java and Homebrew installed, then run;

```
brew install jmeter
```

Easy!

## Making JMeter useful

Out of the box JMeter is a very powerful tool, but it's geared up to be infinitely configurable which is maybe more than we need for a simple benchmark. Also, I don't want to have to faff around producing graphs myself for something as simple as requests per second, so there are a couple of extras that help out here.

## JMeter-plugins

First up we need to add `jmeter-plugins` to give us some realtime graphs. 
Get the file from here: `https://jmeter-plugins.googlecode.com/files/JMeterPlugins-1.0.0.zip`
Then unzip the files to the following path:

```
/usr/local/Cellar/jmeter/2.9/libexec/lib/ext
```

*NB* your version of jmeter might not be 2.9 so double check.

## JMeter ServerAgent

The JMeter-plugins project also provides a handy agent to monitor the CPU and memory usage for the machine you are benchmarking. We can use this to get an idea of how resources are consumed for a given process.

Get the ServerAgent here: 

```
https://jmeter-plugins.googlecode.com/files/ServerAgent-2.2.1.zip
```

and unzip to the following path:

```
/usr/local/Cellar/jmeter/2.9/ServerAgent-2.2.1
```

In reality you can put this wherever you like but I like to keep all the jmeter stuff in one place.
You can run the agent with the following command:

```
/usr/local/Cellar/jmeter/2.9/ServerAgent-2.2.1/startAgent.sh
```

This will keep track of the memory and CPU for later.

## Using JMeter

Let's fire up the GUI by typing `jmeter` into a terminal and hitting enter.

You should be presented with a GUI window like this:

(insert screengrab of jmeter UI)

Welcome to Javaland. Farewell simple DSLs. I still don't *really* understand how to configure JMeter very well, but there's a Ruby based shortcut in the shape of [GridInit](https://github.com/altentee/gridinit-jmeter). It's a gem to produce appropriate config files for JMeter to use, intended for use with their [JMeter-as-a-service](http://www.gridinit.com/) offering.

If you follow the instructions there you should end up with a config file that looks a bit like this:

```ruby testplan.rb
require 'gridinit-jmeter'
test do
  threads 10 do
    visit 'My App Home', 'http://localhost:3000'
  end
end.run
```


