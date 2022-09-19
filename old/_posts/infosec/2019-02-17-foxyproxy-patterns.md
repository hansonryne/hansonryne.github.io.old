---
layout: page
sidebar: right
subheadline: 'Pentesting Tools'
title:  "Selective Proxy Routing with FoxyProxy Patterns"
teaser: "Don't send all your personal traffic to your intercepting proxy when you are testing a web app.  Use patterns to choose what gets logged."
breadcrumb: true
categories:
    - infosec
tags:
    - blog
    - pentesting
    - 'web applications'
    - firefox
    - proxy
    - tools
image:
    thumb: foxyproxypatterns/foxyproxy.svg
    title: foxyproxypatterns/title.PNG
---
FoxyProxy is one of the first things I add to a fresh install of Firefox when I am setting up my workstation.  As a web app pentester, there is nothing more important to me than the ability to push my traffic through different tools to inspect and modify data in transit, and a proxy is the easiest way to do that.  With FoxyProxy, it's as easy as making a profile with the IP address and port of where you want to route your traffic and then clicking that profile to turn it on.  In a few seconds, everything you do on the internet can be inspected by Burp, Zap, or any other tools you want to use.

However, there are times where I don't want all of my traffic to go through my proxy.  Maybe I want to keep my logs clean of personal information so I can share those logs with a client later, or I want to be sure that I don't accidentally modify data going to sites I'm not actively testing.  Whatever the reason, it's nice to know that FoxyProxy comes with profile settings that allow you to set up a pattern that will only reroute matching domains to your proxy.  This is a quick walkthough of that process.

## Step 1: Set Up a New Profile
The first thing you will want to do is setup a new profile in FoxyProxy.  Use a name that makes sense for the traffic you want to selectively route.  I'll use example.com

To setup the profile, click the FoxyProxy icon in the toolbar at the top of Firefox and select Options from the dropdown menu.  

<figure style="text-align:center; margin:1em"> <img src="/images/foxyproxypatterns/options.PNG" alt="options"><figcaption style="font-size:small"></figcaption></figure>

In the options tab that opens, click 'Add' and you will get to the page where you create a new proxy profile.  I will be setting this profile up for a Burp proxy, so I will choose HTTP as the type, example.com for the title, 127.0.0.1 as the IP, and 8080 as the port.  Your settings may vary depending on your proxy address and port.  You can change the color if you want as well, but uncheck the setting that adds a whitelist pattern to match all URLs.  Make sure when you are done filling in the information you click 'Save & Edit Patterns' to get to the next step.

<figure style="text-align:center; margin:1em"> <img src="/images/foxyproxypatterns/add.PNG" alt="add"><figcaption style="font-size:small"></figcaption></figure>

### Step 2: Create Your Patterns

After you save the information in step one, you will find yourself at the Add/Edit Patterns page.  To selectively route traffic to the proxy, you will want to add 'White Patterns' to the profile.  The 'Black Patterns' at the bottom of the screen are there to ensure local traffic is not sent to the proxy.  Usually this is ok to leave alone if you are testing external sites.

To create a new pattern, click on the 'New White' button at the bottom of the screen and a new line will be added under the 'White Patterns' section.  Click into each of the areas and fill in the settings to your liking.  For this example, I will name the pattern "example.com" and set the pattern to "\*example.com" with a type of "wildcard" and http(s) set to "all" or "both."

<figure style="text-align:center; margin:1em"> <img src="/images/foxyproxypatterns/patterns.PNG" alt="set up patterns"><figcaption style="font-size:small"></figcaption></figure>

There is no restriction against using subdomains in the pattern setting.  You can do things like \*www.example.com or \*mail.example.com, if you want to.  However, you can only match domains.  You man not match directories under a domain such as \*example.com/testing*.

When you have the rule filled in the way you want, make sure you click 'Save' to apply the settings.

## Step 3: Turn On the Patterns
After the patterns are saved into the profile, you will see a new option when you click the FoxyProxy icon in the Firefox toolbar.  Whatever you named your profile will be in the dropdown menu, but you won't want to click that because it will use that proxy for ALL TRAFFIC and ignore the patterns you just created.  Instead, click "Use Enabled Proxies By Pattern and Priority" to turn on selective proxy routing based on the patterns you set up.  

<figure style="text-align:center; margin:1em"> <img src="/images/foxyproxypatterns/use.PNG" alt="turn on patterns"><figcaption style="font-size:small"></figcaption></figure>

Now if your proxy is running, you should notice that only traffic that matches the patterns you entered in the profile settings is being sent to your proxy.  Congratulations!  No more worrying about sending your personal traffic to a proxy log or having to filter through hundreds of different domains trying to find the one request you need.

## Other Notes
* When the patterns are set up and enabled, the icon in the browser toolbar will show you what proxy profile you are currently sending traffic through based on the page you are looking at.  Take a look at this while you are testing to make sure it says the profile you expect.

* You can enable or disable patterns without deleteing them or a profile by going into FoxyProxy options and toggling the switch for a profile on and off.

* This is only a guide for FoxyProxy, but there are many browser proxy add-ons that have similar features.  SwitchyOmega for Chrome is an example.  In the SwitchyOmega settings, there is a profile called, 'auto switch' that allows you to use similar wildcard patterns to match and select specific proxies based on IP or domain name.  More or less, you can generally apply the step in this guide to set those up to your liking as well.
