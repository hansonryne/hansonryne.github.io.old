---
layout: page
sidebar: right
subheadline: 'Pentesting Tools'
title:  "Web Application Pentesting with Burp Suite (Part 2)"
teaser: "Continuing the Burp Suite Tutorial Series with a look at Scope, Scanning, and Spidering"
breadcrumb: true
categories:
    - infosec
tags:
    - blog
    - pentesting
    - 'web applications'
    - burp
    - tutorial
image:
    thumb: burptut-02/spider-scope.jpg
    title: burptut-02/spider-scope.jpg
---

So you made it through [Part 1](https://www.rynehanson.com/infosec/burp-tutorial-01/)? Congrats on your success - that was the most boring part.  Now it's time to hit the web and hack some apps.  But where do we start?  Well, I don't recommend hitting the open information highway and testing whatever you see.  That might cause some problems for you.  Thankfully, there are many apps to test with online that are free to use in a safe environment.

This part in the series is going to go over some basic uses of Burp to get you on the road to your first finding.  To start, we need to set up a good practice target.  After that, we will take a look at scoping our testing and using Burp's built in crawl and scan tools to try and find some low hanging fruit.  Maybe even squeeze some "Juice" out of it. (You'll understand that pun in a second).  All of this is just the beginning of what will eventually lead to a mastery of Burp Suite, so now is a good time to be excited.


## Finding a Target
The best and easiest way to get started with practicing web hacking (that I have found) is to use <a href="https://attackdefense.com">attackdefense.com</a>.  At the time I'm writing this, the site is free to use, and all you need is a Google account.  For the purposes of the rest of this series the specific section of attackdefense that we will use is the <a href="https://www.owasp.org/index.php/OWASP_Juice_Shop_Project">OWASP Juice Shop Vulnerable Web Application</a> (see pun from previous paragraph).  It is listed on the site under the 'Deliberately Vulnerable' -> 'Web Apps' navbar option, and it is called (drumroll.....) <a href="https://attackdefense.com/challengedetails?cid=452">'Juice Shop'</a>.

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-02/juiceshop.PNG" alt="Juice Shop">
<figcaption style="font-size:small;">
A nice juicy app to test 
</figcaption>
</figure>

The best thing about this setup is you can start and stop the application as many times as you want and it will reset it to its original state.  If you mess something up or want a completely fresh start, you can do it in less than a minute.

## Get the Proxy Up and Running
Once the application is running and you click on the lab link (should be something like http://sdfadgagaweff.public.attackdefenselabs.com) to navigate to the web page, you will finally be able to set up Burp to intercept traffic.  Depending on the browser proxy extension you set up in Part 1, set it up to send all traffic through Burp on its IP and port, and start browsing the lab.
<figure style="text-align:center; margin:1em">
<img src="/images/burptut-02/foxyproxy.PNG" alt="FoxyProxy">
<figcaption style="font-size:small;">
If you don't recognize this you skipped <a href='https://opensecurity.io/resources/beginners-guide-to-web-app-pentesting-burp-suite/'>Part 1</a> .... shame on you
</figcaption>
</figure>


## Finally Using Burp
Hopefully you will see stuff showing up in the 'Site map' section of the 'Target' tab.  In some cases, like if you have other browser tabs open, you might see WAY too much stuff showing up.
<figure style="text-align:center; margin:1em">
<img src="/images/burptut-02/lotsoftargets.PNG" alt="lotsoftargets">
<figcaption style="font-size:small;">
TOO MANY TARGETS (redacted)
</figcaption>
</figure>

Looking at the pane on the left side, you can see there are a lot of domains going through Burp.  Most of the time, you aren't going to want to test more than a couple sites at a time, and it can get very annoying having to sift through all those options to find what you are looking for.  This is where scoping your Burp session comes in.

## Declutter Your Life
If you are in the 'Site map' tab, move one tab over to the right and you will be in the 'Scope' area for burp.  It looks a little something like this.

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-02/scopetab.PNG" alt="scope">
<figcaption style="font-size:small;">
360NoScOp3?
</figcaption>
</figure>

As you can see, mine is blank right now, which means everything that goes through the Burp proxy is being recorded and presented to you.  How helpful!!  Except when it isn't.

Paring down results can be done in a variety of ways.  First, you can just click 'Add' and type in a site by hand.  If you are using Juice Shop on attackdefense.com, you might want to add something like, "http://e2dnmsfarrd76w1v0zgh7weig.public1.attackdefenselabs.com/" (a link like that will show up when you start the lab).

If typing or copy/pasting a site in by hand just doesn't do it for you, Burp also allows you to add items to the target scope by right clicking on the domain in the 'Site map' tab and then choosing "Add to scope" in the dropdown.

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-02/addtoscope.PNG" alt="add target">
<figcaption style="font-size:small;">
Right click menus are the best place to learn about a tool like Burp
</figcaption>
</figure>

Once you add an item to a scope, Burp will ask if you want to stop paying attention to out-of-scope items.  Usually, "Yes" is the best option.  When you are all done, Burp will add the site to the scope list, and all other traffic will be ignored by Burp so you won't have to see it.  

But you aren't quite done decluttering your 'Site map' tab yet.  The last thing you need to do is filter out all the other sites that were already captured before the scope was set.  To do that, go back to the 'Site map' tab, click on the filter section that runs across the top of the window, and choose the box that says "Show only in-scope items."

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-02/scopefilter.PNG" alt="filter options">
<figcaption style="font-size:small;">
You can filter by a lot of different things
</figcaption>
</figure>

Now when you click off that menu, Burp will hide everything that hasn't been added to the scope, and you will have a much cleaner space to work in.

###Bonus Blog: Advanced Proxy Setup
[Filter traffic that gets sent to Burp from the browser proxy settings](https://www.rynehanson.com/infosec/foxyproxy-patterns/)

##Take a Quick Breath, Then Get to Spidering and Scanning

So now that Burp is proxying, logging, and scoping your web traffic, it's time to DO THE DAMN THING and start looking for vulnerabilities.  The quickest and easiest way to do that is to use Burp's automated crawl and scan tools.  Remember I am using the Burp professional version 2.0.xbeta, so if you are using an older version like 1.X or the free Community Edition and your options don't look exactly the same, that is why.

From the 'Site map' tab, right click on your target and search the dropdown menu for the 'Scan' option.  Clicking on that will open a new window where you can select scanning options.  In this case, we will go with the default options, but feel free to poke around and try different configurations.

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-02/newscan.PNG" alt="New Scan">
<figcaption style="font-size:small;">
Don't be afraid to spend some time poking around.  I'm not going anywhere.
</figcaption>
</figure>

When you click 'Ok' and start the scan, head back to the 'Dashboard' tab and look in the Tasks pane.  You should see a new box with a request counter incrementing.  This is the task that is scanning the site you selected for issues.  

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-02/scan.gif" alt="running scan">
<figcaption style="font-size:small;">
And we are off to the races
</figcaption>
</figure>

If an issue is detected, it will show up in the pane just to the right of the running task with a description of the issue and which request or response got flagged.

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-02/issues.PNG" alt="issue pane">
<figcaption style="font-size:small;">
You've got some issues....
</figcaption>
</figure>

Nice!  Looks like we have an 'Open redirection' issue in the Juice Shop application.  Congratulations!  You have found your first vulnerability.  Typically you would then manually investigate to find out if Burp gave a false positive, but I'll just tell you it's the real deal this time.

The other thing that happened while the scan was running, was your 'Site-map' was populated by the crawling (also known as spidering) function.  This might help you uncover interesting areas like the 'private' directory, that you might miss while navigating the site manually.

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-02/crawl.PNG" alt="sitemap">
<figcaption style="font-size:small;">
Things you might have missed doing it on your own.  'Private' looks..well...private.</figcaption>
</figure>

Burp will crawl pages looking for links and try to follow them only if they fall within the scope of the project.  THIS IS WHY SETTING SCOPE IS VERY IMPORTANT.  If you don't have a scope set, it will crawl all the links recursively and go FOREVER, and you may end up in places you aren't allowed to be.  Remember Spider Man, with great power comes great responsibility.

## Wrap Up
So now you have a target, a scope, a scan, and your first finding under your belt.  At this point, all you have to do is dig dig dig to find as much as you can.  This will involve doing things that the application developers didn't anticipate or expect in many cases.  To help with this, the next installment in the series will focus on the intercepting proxy that makes Burp such an effective tool for manipulating requests and responses in interesting ways.
