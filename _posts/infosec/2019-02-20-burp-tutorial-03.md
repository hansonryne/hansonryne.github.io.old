Welcome to Part 3 of the Burp Suite tutorial series - where you learn to use one of the most powerful tools in web application pentesting effectively and efficiently.  This is a pretty exciting edition of the series because, unlike Part 1 and Part2, you are finally going to start doing some manual manipulation of HTTP traffic and find vulnerabilities that a typical automated scanner will fail to detect.

This post focuses on the core function of Burp Suite: the intercepting proxy.  This is the tab where a web application penetration tester will spend a good deal of their time in Burp whether it be to manually inspect traffic as it leaves and returns to the browser, to look back at the history of requests, or to manage rules and filters that change requests and responses on the fly.  This is the core functionality of Burp Suite, so it is critical that you have a good working knowledge of this fundamental block of the application in order to take advantage of everything that is to follow.  With that, let's dive in.

## Why Use a Proxy?
As I've mentioned before, the use of an intercepting proxy for web app pentesting is incredibly important.  Burp Suite gives you this functionality in the 'Proxy' tab of the interface.

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/proxytab.PNG" alt="Proxy Tab">
<figcaption style="font-size:small;">
Descriptive...if not creative.
</figcaption>
</figure>

But why is the ability to see all your web application traffic and mess with it so important?  Well the truth is, web apps do so much more than they appear to on the surface.  A single click can generate dozens of requests in the background and submit information you never see just by browsing a site.  For a typical user, this is a convenience.  For a security researcher, this can't be ignored.

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/google.PNG" alt="One request?">
<figcaption style="font-size:small;">
All this just to see the Google homepage?
</figcaption>
</figure>

Any one of the requests in the screenshot above could have a security flaw in it, but if you only used your browser, you wouldn't even know they were occuring.  Being able to inspect each request and response individually and on your own schedule is very helpful.  Once you have a chance to look at them, you get to see what looks important or interesting, mess with what would be considered "normal," and then see the results.

Not sure exactly what I mean?  Let's fix that by digging into the details a little more.

## The Options Tab
<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/options.PNG" alt="Options">
<figcaption style="font-size:small;">
We've got options...
</figcaption>
</figure>

If you have been following along as you went through parts [1](https://opensecurity.io/resources/beginners-guide-to-web-app-pentesting-burp-suite/) and [2](https://opensecurity.io/resources/beginners-guide-to-web-hacking-with-burp-suite-suite-part-2/) of the series, you have already come across the 'Options' subtab.  Here is where you configure everything that facilitates the basic functionality of Burp Suite from the proxy address itself to the automatic manipulation of web traffic.

Generally speaking, you won't mess with the 'Proxy Listeners' section of this tab, as the default address and port of 127.0.0.1:8080 are fine 99% of the time.  The most important parts here are the buttons that manage the CA certificate files, which you should have installed in your browser while you were setting up Burp in [part 1](https://opensecurity.io/resources/beginners-guide-to-web-app-pentesting-burp-suite/) of this series.  You did that right...?  Of course you did!

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/you-rock.jpg" alt="you-rock">
<figcaption style="font-size:small;">
Because you rock!
</figcaption>
</figure>

One section down are the 'Intercept Client Requests' rules.  When you are intercepting traffic, you can configure this section to ensure only the traffic you care about is being intercepted for manual inspection.  Normally you won't change these rules, but it is nice to be able to if you have to.

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/options-1.PNG" alt="intercept-rules">
<figcaption style="font-size:small;">
Request and Response Interception Rules
</figcaption>
</figure>

Right below the 'Intercept Client Requests' rules are the 'Intercept Server Responses' rules.  The manipulation of server responses is not as popular as the manipulation of client requests, but in some cases, such as with Javascript-heavy Single Page Applications, modifying a response can be the most effective way to find vulnerabilities.

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/options-2.PNG" alt="match-replace-rules">
<figcaption style="font-size:small;">
Match and Replace automates the manual changes you make all the time
</figcaption>
</figure>

The last section we will focus on in this tab is the 'Match and Replace' ruleset.  If you find yourself constantly intercepting a request or response and manually changing the data to the same values, this section will help by automating that process by matching strings or Regex values and then changing the content of the traffic on the fly.  Another important thing to note is that the default rules included with Burp may give you some ideas of things to try when testing an application.

## HTTP History

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/http-history.PNG" alt="intercept-rules">
<figcaption style="font-size:small;">
Time for a history lesson
</figcaption>
</figure>

The next subtab that we will go through is 'HTTP history'.  When you are testing an application, it is absolutely crucial that you keep an organized log of everywhere you have been, and how you got there in the first place.  Burp takes care of this for you and does so much more.

The 'HTTP history' subtab provides a list of every request that Burp has logged in the project that is sortable by time, order, type, size, length, method, host, IP...

![3hourslater](/resources/content/images/2019/03/3hourslater.jpg)

...internal temperature, population density, and the diamond's clarity (I may be exaggerating).  Basically, it is very helpful for finding the requests and responses you want to look at in more detail.

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/google-1.PNG" alt="history">
<figcaption style="font-size:small;">
So much information
</figcaption>
</figure>

Taking a look at the image above, you can see all the information that Burp stores for a request to Google.  In the top section is each detail of a request/response pair broken down into columns.  With only one request shown, it doesn't look like much, but when there are hundreds of requests in the log, it is nice to be able to sort by method and see (for example) all the POST requests in line with each other.

Below the top section is the raw request and response data.  If you don't want raw data, Burp will also parse the information and present it in a variety of other ways based on what you like.  Typically, I keep it in raw format, but sometimes just looking at headers or parameters allows me to focus on what I need.  Play around with it and see what works for you.

The last thing to mention in this section is the filter dialog.  Right below the subtabs near the top of the Burp window is a white box that runs the full width of the interface.  Clicking on it will bring up the filter dialog which allows you to filter in just about any way you can imagine.  

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/filters.PNG" alt="filters">
<figcaption style="font-size:small;">
When you have hundreds of requests, a quick filter makes life a lot better
</figcaption>
</figure>

An important note when it comes to filters is to make sure that the defaults are not filtering out things you care about.  For example, if you want to see binary files in the history, Burp will not show them by default.  If you are ever confused about why something isn't being logged, it might be that Burp is indeed logging the traffic, but the filter isn't showing it to you.

## The Intercept Tab

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/intercept-tab.PNG" alt="history">
<figcaption style="font-size:small;">
Where the magic happens
</figcaption>
</figure>

The final subtab that this post will cover is 'Intercept,' and we will do it with a live example that you can try on the Juice Shop Vulnerable Wep Application yourself.  The 'Intercept' tab is what allows manual inspection and manipulation of web traffic in real time for both requests and responses.  When you are testing a web application, the understanding of this tab is critical to everything that follows.  Even if you favor other tools in Burp over manual interception, it is important to realize that the core technology that runs Burp is based off the function that the 'Intercept' tab provides.

Before the demonstration, I have a few quick notes that might make your life easier.  First, if you like hotkeys, turning interception on and off can be done via CTRL-t.  Other hotkeys are listed in 'User options' -> 'Misc', but I use the one for interception more than almost anything else.  

Second, a clean Burp installation turns interception on every time you start Burp by default.  This can be annoying if you don't want to start intercepting right away, but it can be turned off by going to 'User options' -> 'Misc' and scrolling down to the section called 'Proxy Interception'

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/proxy-interception.PNG" alt="off-by-default">
<figcaption style="font-size:small;">
You want to do this...trust me
</figcaption>
</figure>

## Let's Get Intercepting
This example is going to show one of the most important reasons someone would want to intercept and manipulate traffic in the first place: To bypass client-side security controls.  Have you ever been to a site where you had to put in an email address and the form would not submit until you entered an address in the correct format?  That is an example of a client-side control.  

As a general rule, client-side controls should not be the only methods used to validate user input (also known as "unsafe input").  However, some developers will use them as the sole security mechanism in an application.  When this happens, an attacker (played by you in this scene) can bypass that security control and send unsafe input (played by an XSS payload in this scene) to the application.  If that doesn't make sense, sometimes it is easier to understand with a walkthrough.

### Step 1: Find a Client Side Security Control
The Juice Shop application has a client side security control when users register for an account.  To get to the 'User Registration' page, you can either navigate to the '/#/register' path in the URL bar of your browser or click on 'Log In' and then 'Not yet a customer' in the interface.  

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/register.PNG" alt="User registration">
<figcaption style="font-size:small;">
Let's sign up for an account
</figcaption>
</figure>

Once you are on the page, try to sign up for an account.  As you are typing in your email address, you may notice a red bar at the top of the screen that gives an error message stating: "Email address is not valid." 

First, submit a valid email address to create an account.  I'll use me@me.me as an example.  Now, when you successfully submit a new user registration to the application, the administration page will display that email address in a list of registered users.  

POTENTIAL SPOILER: If you want to see this for yourself, login to the administrator account (User: admin@juice-sh.op, Password: admin123 on attackdefense.com instance) and navigate to the '/#/administration' path.

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/me-registered.PNG" alt="Registered">
<figcaption style="font-size:small;">
There is me at the bottom
</figcaption>
</figure>

So your user email is displayed in the interface of the administrator page.  Cool.  But what if you could get something else to render in the page?  Something like malicious JavaScript perhaps?  "That's quite impossible," some may say.  "There is a control in place to make sure you can only submit a valid email address."  This is where our intercepting proxy comes in to play.

Log out of the application and navigate back to the user registration page.  Then turn on Burp interception by either clicking the button in the tab or using the CTRL-t hotkey combination.  When it is on, the button should appear to be pressed in and it should read, "Intercept is on."

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/intercept-on.PNG" alt="intercepting">
<figcaption style="font-size:small;">
Prepare to intercept!
</figcaption>
</figure>

Again, fill out the form.  You can even use the same exact information you put in the first time because we are going to change it before it gets to the server.  When you submit, your Burp application should start flashing and when you look at it, you should see something like this in the interceptor window.

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/orig-burp-1.PNG" alt="Intercepted">
<figcaption style="font-size:small;">
INTERCEPTION!!!!
</figcaption>
</figure>

Notice the parameters that are being sent to the application, and also that when you click into the window, you are able to change the text that is being sent.  Imagine if you changed the email to something like what is in the photo below.  

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/modified-1.PNG" alt="modified">
<figcaption style="font-size:small;">
    Payload: &lt;script>alert('Stored XSS')&lt;/script>
</figcaption>
</figure>

Then stop imagining it and do it.

After the request is modified, you can either click forward or just turn interception off and the request will be forwarded for you.  Once it is on its way, check out how Burp logged the request in the 'HTTP history' tab.  

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/history.PNG" alt="edited">
<figcaption style="font-size:small;">
See the checkmark in the 'Edited' column and the extra 'Edited request' tab at the bottom?  You did that.
</figcaption>
</figure>

Notice how Burp shows the original request and the edited request for your records.  Thanks Burp!

Ok so the request has been submitted and based on the response from the server, it looks like the JavaScript payload was accepted as a valid email despite the client-side controls in the browser.  Take a look at the 'Response' tab to verify this for yourself.

Even more importantly, if you log back in as an admin and go to the administration page of the application you might notice a popup appears with your XSS message!  Great job.  Using an intercepting proxy, you have bypassed a client-side security control and sent a "malicious" payload to the application.  

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2019/03/xss.PNG" alt="xss">
<figcaption style="font-size:small;">
This is just a simple, and benign, example.  XSS is bad news bears.
</figcaption>
</figure>

## Impact
So what is the point of this?  Well those familiar with cross-site scripting attacks know that this kind of attack can lead to keylogging, website defacement, information disclosure, denial of service, session hijacking, etc.

Even better, since this is a cross-site scripting attack on the admin page, it is likely you can steal an administrator's session and log in with their account.  Not bad huh?

And this is all because the only security protecting against this is a client-side check for the proper format of an email address.  A properly secured application *may* have client-side checks like that, but ***must*** have server-side validations of every user-supplied input.  Going even further, all data that is displayed in a page should encode special characters like '<' and '>' so they don't render as code in the page.

Relying on the client to send safe information to a web application is trivial to bypass - as you have just proved with a single request modification.

## Conclusion
And that wraps up this installment of the Burp Suite tutorial series.  It is important that you understand what happened in this post as all the other tools in Burp use the same priciples to work.  The most important thing to get out of this is that Burp allows you to manipulate requests and responses as they transit between your browser and a web server, allowing attackers to send anything they want to an application.

The next part of the series will focus on the 'Intruder' tab - a handy tool for brute forcing and fuzzing application inputs for vulnerabilities in a semi-automated fashion.

And for those just joining in this journey through Burp Suite, be sure to check out [Part 1](https://opensecurity.io/resources/beginners-guide-to-web-app-pentesting-burp-suite/) and [Part 2](https://opensecurity.io/resources/beginners-guide-to-web-hacking-with-burp-suite-suite-part-2/).
