---
layout: page
sidebar: right
subheadline: 'Pentesting'
title:  "Web Application Pentesting with Burp Suite (part 1)"
teaser: "When it comes to pentesting web applications there is nothing quite like Burp Suite. Join me as we dig into Burp Suite with real world practice examples!"
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
    thumb: burptut-01/logo.jpg
    title: burptut-01/Burp-2.png
---
When it comes to hacking web applications, the possibilities are endless because the technologies to build them come in nearly unlimited flavors and stacks.  These days you can run any combination of front and backend codebase, with any storage database you want, in the cloud or on site, behind a firewall or VPN, on a desktop or a cell phone, for a bank or a toaster.....you get the point.  So with all of this variability on the web today, it might seem like you need a tool for every situation.  To some degree you are right, but the good news is there is one tool that will always come in handy: an HTTP proxy.  And when it comes to an HTTP proxy, one name stands above the REST (like an API...get it?).

Bottom Line Up Front: Burp Suite is a web application pentester's best friend.

# Burp Suite?
While it is unclear why a company would name their flagship product after a belch, one thing that is clear is the folks at <a href=https://portswigger.net/burp>PortSwigger</a> have made a tool that will stand the test of time in web application testing.  Burp Suite is a form of HTTP proxy - that is to say it sits in between your browser and the internet and forwards traffic in either direction.  Think of it as a man-in-the-middle attack on yourself, but you are happy about it.
What makes Burp SWEET, is that it will record, intercept, replay, and analyze that same traffic while also allowing you to manipulate requests and responses in ways your browser won't.  On top of that, it is extensible via third-party add-ons that can be written in Java, Ruby, or Python in order to automate testing and simplify attack techniques.  Basically, if you want to do something with a web request or response, Burp will help you - probably in a variety of ways.
# Burp Flavors
To get started, you must first download a version of Burp from the <a href=https://portswigger.net/burp>PortSwigger website.</a>  At this point in time, Professional is at v2.0.13beta and Community is at v1.7.36.  The difference in the major release update is the availability of a limited REST API and a new dashboard that separates running tasks into jobs that can be paused and started individually.  

Most of the essential tools are available in the free Community Edition, but if you really want to turn up the heat, the Professional Edition will always be updated first and comes with many additional features for a fairly reasonable $399/year (image below is out of date).  These include a full traffic history search function, faster brute forcing speeds in the Intruder tool, an automated scanner, various engagement tools, and many more things that make life easier for you.  I will be using Pro on Windows for all of the follow on tutorials and reviews.  So if you want to follow along, you'll want the newest version of Pro.

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-01/burpflavors.PNG" alt="versions">
<figcaption style="font-size:small;">
Start with Free.  You'll want Pro
</figcaption>
</figure>

There is also an Enterprise Edition for sale, but it is more of a application scanner than a penetration testing tool.  It will not be evaluated or described any more than that in this particular series.

You also want to make sure [Java](https://www.java.com/en/download/) is installed on your machine because, well, Burp is made with Java.
# Getting Started
After downloading all the required software, run the installers as you would any other application and launch the program.  The first screen you see will look a little something like this:

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-01/burp1-1.PNG" alt="Starting Screen">
<figcaption style="font-size:small;">
Welcome to Burp
</figcaption>
</figure>

If you bought professional you might see a popup to put in your license key at this time.  Please do.  I'll wait.

If you want to save your project, click new project on disk.  Otherwise click next and then "Start Burp" and you will be in the Dashboard.

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-01/burp2.PNG" alt="Dashboard">
<figcaption style="font-size:small;">
    Ok...<em>now</em> Welcome to Burp
</figcaption>
</figure>

# Setting Up Your Browser
So the thing about an HTTP proxy is you need to force your browser to use it.  Every browser does things a little differently, but the best way to do it is use a browser extension like <a href="https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/">FoxyProxy</a> for FireFox or <a href="https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=en">SwitchyOmega</a> for Chrome.  I'll go over both.
## FireFox
Go to https://addons.mozilla.org/ and search for FoxyProxy.  The one you are looking for looks like this: 

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-01/foxyproxy.PNG" alt="Add On" style="max-width:200px">
<figcaption style="font-size:small;">
    ....advanced what?!
</figcaption>
</figure>

Once it is installed (you may need to restart your browser) you can open the options by clicking on the FoxyProxy icon in the browser toolbar and selecting "Options".

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-01/foxyproxyoptions.PNG" alt="Options">
<figcaption style="font-size:small;">
    Foxy
</figcaption>
</figure>

To set up the proxy, click Add, give it a title, set 127.0.0.1 as the IP address, and 8080 as the port.  This assumes you didn't change the Burp defaults.  If you did you can go to the Proxy tab in Burp, the Options tab under Proxy, and look at the settings under Proxy Listeners to find the right informaiton.

Once everything is set correctly. Save the configuration and turn on the proxy by clicking on the icon in the toolbar again and select use "for all URLs (ignore patterns)"  I'll go over patterns at some point becasue they are very helpful, but we have enough to do right now.

Now you might think you are done, but if you try to navigate to any HTTPS site, you will see an error saying the connection is not secure.  Remember when I mentioned Burp is like a man-in-the-middle attack?  Well you have just fallen victim to yourself.  No worries though.  Burp has a way to fix this that I explain below.  If you are not interested in seeing how to set up Chrome's SwitchyOmega, feel free to skip ahead.

## Chrome
To get started installing SwitchyOmega, go to the <a href=https://chrome.google.com/webstore/category/extensions> Chrome Web Store</a> and search for SwitchyOmega.  It looks like this:

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-01/so1.PNG" alt="SwitchyOmega">
<figcaption style="font-size:small;">
    Nice rating FelisCatus
</figcaption>
</figure>

Add it to Chrome and give it the permissions it needs.  Once it is installed you can go through the tutorial or skip it.  Eventually you will need to add a New Profile, give it a name, change the protocol to HTTP, set the server to 127.0.0.1, and  set the port to 8080.  As with FireFox, if you changed the default Burp settings you can go to the Proxy tab in Burp, the Options tab under Proxy, and look at the settings under Proxy Listeners to find the right informaiton.  Finally click apply changes and make sure the new profile is selected when you click on the icon in the Chrome toolbar.

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-01/so2.PNG" alt="SwitchyOmega" style="max-height:200px">
<figcaption style="font-size:small;">
You can name it whatever you want
</figcaption>
</figure>

As with FireFox, HTTPS sites will not work because Burp is intercepting the requests.  The next section will explain how to fix that.

<h1>Setting Up Certificates</h1>
Burp has to use its own SSL certificate when attempting to proxy for sites using HTTPS because it has to strip away the encryption so it can read and display the data for you.  Unfortunately, your browser doesn't like receiving plaintext data when it asked for something encrypted, so you have to load Burp as a trusted Certificate Authority and install the applications certificate.  Luckily this is simple.

First, go to the Proxy tab in Burp and then the Options tab in Proxy.

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-01/sslsetup1.PNG" alt="SwitchyOmega" style="max-width:400px">
<figcaption style="font-size:small;">
This is what I mean
</figcaption>
</figure>

Then click "Import/Export CA Certificate", select "Certificate in DER Format" from the export options, click next, choose where to save the file, click next, see that it worked, and close.

<figure style="text-align:center; margin:1em">
<img src="/images/burptut-01/sslsetup2.PNG" alt="SwitchyOmega" style="max-width:400px">
<figcaption style="font-size:small;">
Suite Success
</figcaption>
</figure>

## FireFox Certificate Installation
To install the recently exported certificate in FireFox, go to Options, search for "certificates", and click "View Certificates".   Click on the Authorities tab and then Import.  Find the certificate you exported before (you may need to change to "All Files (*.*)" in the explorer file type dropdown), open it, and click through with default settings to the end.  When you have reached a success message you have made it! Now restart FireFox and when you browse to HTTPS sites, you should have no Insecure Connection warnings.

## Chrome Certificate Installation
To install the certificates in Chrome, go to Settings, search for "Manage Certificates", and click the highlighted section to open the Certificates dialog window.  Make sure you select the "Trusted Root Certificate Authorities" tab and then click import.  Find the proper certificate from burp (you may need to change to "All Files (*.*)" in the explorer file type dropdown), open it, and click through to the end of the wizard with default settings.  You should get a popup saying the import was successful.  Restart Chrome and once again, HTTPS will work without error warnings.

<h1>Good Job!</h1>
That was a lot of set up, but it will be worth it soon.  In the next installment of Web Hacking with Burp Suite, I will show some examples of some basic tools provided with Burp.  For now take a breath, and get ready to hack the web!
