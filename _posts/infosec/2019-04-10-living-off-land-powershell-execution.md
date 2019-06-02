---
layout: page
sidebar: right
subheadline: 'Pentesting'
title:  "Living Off the Land: Opening Powershell When You Can't Open Powershell"
teaser: "When you aren't allowed to open Powershell via normal start menus or links, how do you do it anyway?"
breadcrumb: true
categories:
    - infosec
tags:
    - blog
    - infosec
    - post
    - pentesting
    - powershell
    - windows
image:
    thumb: lol-powershell-bypass/03spgpowershell-a-smart-persons-guidehero.jpg
    title: lol-powershell-bypass/03spgpowershell-a-smart-persons-guidehero.jpg
---
Fairly recently, my team found ourselves with Remote Desktop Protocol (RDP) connections into a network as a low level user.  It was a good spot to be right off the bat, since it was a reliable foothold in the network, but when I said it was a low level user I mean LOW.

The environment we found ourselves in had a lot of security restrictions in place.  There was no access to any files outside the user libraries via the file explorer.

<figure style="text-align:center; margin:1em">
<img src="/images/lol-powershell-bypass/file-explorer.PNG" alt="file-explorer">
<figcaption style="font-size:small;">
Look at the left sidebar.  Not even "This PC"
</figcaption>
</figure>

We could not open a command prompt or a PowerShell window through the start menu.

<figure style="text-align:center; margin:1em;">
<img src="/images/lol-powershell-bypass/cmd-click.gif" alt="no clicking" style="max-height:400px;">
<figcaption style="font-size:small;">
Can you at least give me an "Access Denied" or something?
</figcaption>
</figure>

We couldn't even use "run."

<figure style="text-align:center; margin:1em">
<img src="/images/lol-powershell-bypass/run.PNG" alt="Proxy Tab">
<figcaption style="font-size:small;">
Walk don't run...actually don't even walk
</figcaption>
</figure>

It seemed like everything was fairly well locked down, but it wasn't time to give up.  If there is one thing Windows gives us a lot of, its options.  

So without even the permissions to **right click(!?!)**, we set off to find a way to move forward, and before long we had three different ways to open PowerShell and keep testing.  With the help of some handy tools like BloodHound and Rubeus and our gained PowerShell access, it wasn't long until we were Domain Admin.

<h2> Method #1: Drop a .bat</h2>

Hey, you want to know a secret?  Internet Explorer is still around, and in a big way.

<figure style="text-align:center; margin:1em">
<img src="/images/lol-powershell-bypass/market-share.PNG" alt="market share">
<figcaption style="font-size:small;">
Thanks w3counter.com
</figcaption>
</figure>

Want to know another secret?  Internet Explorer will run batch files for you.

The process is fairly simple.  Especially if your goals are small - like opening a command prompt.  

1. Open a notepad document.  I'd be surprised if your organization didn't allow that.  
2. Type in "powershell.exe"
3. Save it as a batch file to your Desktop.
4. Open an Internet Explorer window.
5. Drag the batch file into the Internet Explorer window.
6. Click through all the prompts.
7. ...
8. Profit

<figure style="text-align:center; margin:1em;">
<img src="/images/lol-powershell-bypass/notepad.gif" alt="batch file" style="max-height:400px;">
<figcaption style="font-size:small;">
Oh yeah.....
</figcaption>
</figure>

<h2>Method #2: Excel Macros</h2>

In an office environment, you probably have access to Excel, which makes this method pretty handy in most situations.  Macros are not new, and the security risks associated with them are well documented, so usually an organization will disable them.

That was not this case in this test (it will not be the case in every test) so it pays to have a quick macro handy.  Let's walk through how to set it up.

1. Enable the Developer tab on the Excel ribbon.

<figure style="text-align:center; margin:1em;">
<img src="/images/lol-powershell-bypass/devtab.PNG" alt="devtab" style="max-height:400px;">
<figcaption style="font-size:small;">
File -> Options -> Customize Ribbon
</figcaption>
</figure>

2. Open the new macro dialog.

<figure style="text-align:center; margin:1em;">
<img src="/images/lol-powershell-bypass/newmacro.PNG" alt="new macro" style="max-height:400px;">
<figcaption style="font-size:small;">
Developer tab -> Macros
</figcaption>
</figure>

3. Name your macro and click "Create."
4. Enter your macro script.

<figure style="text-align:center; margin:1em;">
<img src="/images/lol-powershell-bypass/vbascript.PNG" alt="vba script" style="max-height:400px;">
<figcaption style="font-size:small;">
Shell "CMD /K powershell.exe", vbNormalFocus
</figcaption>
</figure>

5. Save the macro.
<figure style="text-align:center; margin:1em;">
<img src="/images/lol-powershell-bypass/savemacro.PNG" alt="save macro" style="max-height:400px;">
<figcaption style="font-size:small;">
Save as a macro-enabled workbook
</figcaption>
</figure>

6. Run the macro.
<figure style="text-align:center; margin:1em;">
<img src="/images/lol-powershell-bypass/runmacro.PNG" alt="run macro" style="max-height:400px;">
<figcaption style="font-size:small;">
Developer tab -> Macros
</figcaption>
</figure>

7. Grab shell bro.

<h2>Method #3: Dynamic Data Exchange</h2>

If you read the previous [Living Off the Land](https://www.rynehanson.com/infosec/living-off-land-dde/) post I wrote, you'll probably already know this method as a way to get shells through an Excel email attachment.  But if you are already in the target network, have Excel, and macros *are* disabled (rendering method 2 useless), this is a useful way to bring up a local shell as well.

1. Open a normal blank Excel workbook.
2. Enter the DDE payload into a cell.
3. Hit enter and click through prompts.
4. Hey look!  Another shell.

<figure style="text-align:center; margin:1em;">
<img src="/images/lol-powershell-bypass/dde.gif" alt="dde" style="max-height:400px;">
<figcaption style="font-size:small;">
It's like a turtle colony out here.
</figcaption>
</figure>

<h2>How Do You Stop This?</h2>

So of course it's important to note that this can be secured.  In all of these cases you will essentially have a powershell process running with a parent in Internet Explorer or Excel, and frankly, that should "never" happen (I know never say never).  The best way to identify this is to monitor for programs running from parent processes that make no sense.  Then block it.  

<figure style="text-align:center; margin:1em;">
<img src="/images/lol-powershell-bypass/processids.PNG" alt="processes" style="max-height:400px;">
<figcaption style="font-size:small;">
Bottom line: If you see a powershell running from Internet Explorer, <br>there better be a really good reason for it.
</figcaption>
</figure>


<h2>That's Hardly All</h2>

That's just three ways to bypass restrictions on shells in a Windows environment - it's hardly an exhaustive list.  But really, you only ever need one.  Play around on your own machine and see if you can find more ways to open applications that you never dreamed existed.  You'll be surprised what you find.
