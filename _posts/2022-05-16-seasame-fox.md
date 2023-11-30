---
title: Project Seasame Fox 🦊
date: 2022-06-16 00:00:00
categories: [CYBER, WIFI]
tags:      # TAG names should always be lowercase
---

**TLDR:** I made a device that logs all the wifi traffic and maps it out. It can also do some hacking stuff.

Intro
-----

I knew I wanted to make something. I love making stuff. Making a physical thing would be awesome, something that was small and discrete, something that would make me feel like a spy hiding in plain sight. At the time of this spell of inspiration, I was learning about wifi and its capabilities at uni. I combined these ideas together to make a device that can log and and map any device that is using wifi. This means that I can track any phone or laptop even if it is not connected to wifi, the wifi just needs to be turned on on the device. The making of this project can be broken up into four parts. **1.** Create the physical device that is
 capable of searching for wifi devices and capturing network traffic while also knowing its physical location **2.** Create a script that will allow the device to log information about wifi devices acompanied by information about its physical location. **3.** Create a simple user interface to plot the information recorded by the device. **4.** Add some hacking features to feel cool. Simple.  
  
A discovery I figured out about wifi protocol was that heaps of information is constantly being transmitted to everyone. Whenever the wifi on your phone is turned on, your phone is constantly sending a signal to anyone that will listen, all about itself including it’s MAC address (unique id) and all the wifi connections it has made in the past. Because of this, we can listen in and take this data.  

Part 0: Research and planning
-----------------------------

This first bit was quite difficult. I had to find tools that could help me achieve the goals mentioned above while also being small and discrete.  
  
I needed a small computer that would have the capability to connect to the internet and scan for networks. There was only one real option. The raspberry pi, more specifically, the Raspberry Pi Zero W. Not only was this device smaller than a credit card, it also had an antenna that could search and find wifi and with a bit of tweaking (using Kali Linux and the re4son kernel), it could capture network traffic as well.  
  
I also had to have the device’s physical location at all times, the one solution to this is a GPS. The cheapest, most compact solution was the U-Blox Neo 6M GPS.  
  
The last thing I needed was a power source. The device needed to be small and I didn’t want to have to connect it to a powerpoint or a computer, I really wanted it to be self contained. A cheap solution that I found which was also very accesible was the Keji 2000mAh Powerbank.

Part 1: Setup
-------------

This process took an extremely long amount of time. I had to find all the equipment and items that I needed for the device and I had to setup the raspberry pi to correctly work with everything.  
  
I created a sketch of where all the parts would go, this would allow me to visualise where everything would go and help create dimensions for the device’s housing.  
![Drawing schematic](/assets/img/seasame_fox/drawing%20schematic.jpeg) I had to boot an operating system called Kali Linux onto the raspberry pi to allow for the tools neccessary in wifi monitoring and hacking. On top of that I had to download a kernel that allowed for me to use wifi myself while scanning other network traffic.  
  
Connecting the GPS to the raspberry pi was quite simple, all it required was soldering some pins from the GPS to the pi.  
  
The housing for the device went through many iterations. I decided to use Fusion 360 to design it since I had quite a bit of experience using that. Throughout the project I created multiple iterations that fixed past mistakes and made some improvements.  
_Here is a photo of the iterations up until now_. ![All housing](/assets/img/seasame_fox/all%20housings.jpg) _Here is the components combined inside its housing_ ![All together](/assets/img/seasame_fox/all%20together.jpg)

Part 2: Writing the script
--------------------------

My script had to have the following capabilities:  
\- wait until GPS data is collected  
\- capture wifi data in a certain time interval  
\- add gps data to wifi data  
\- save to file  
  
\*\*\* this bit is a bit more technical \*\*\*  
  
Initially I was planning write all of this in bash, bash is a scripting language and allows execution of commands you would usually write on the command line. Since I was executing “airodump-ng” as the main command, I thought that using bash would be easy. Nope. Trying to parse data from the GPS and also append and edit wifi and gps data was a nightmare. I tried desperately to get it to work to no avail. Instead I switched over to python, a language I was familiar with. Using a library called subprocess, I was able to execute lines of code as if I was writing in the terminal. I was also easily able to manipulate the data, allowing the gps data to be added to the wifi data and then appending this information to the end of a file.  
  
Next, I had to find a way to automate this process. Since I can’t run the device constantly as turning it off would lead to all the data being lost, I instead decided to run the script for 2 minutes at a time and just re-run the program every 2 minutes. A program that runs in the background is called a daemon and to make your program into a daemon, you have to use a feature called “cron”. By editing the crontab, you can make programs run at certain intervals, every minute, every hour etc. I edited the crontab file to run my program every 2 minutes.  
  

Part 3: Mapping the information
-------------------------------

My goal for this part was to have a simple interface that allows a user to see all the wifi devices that have been logged and also display its physical location. To achieve this goal, I used some simple HTML, Javascript and the Google Maps API. Once I had the data in a file on my computer, I converted the csv file into a json file (a format more friendly for the API), separated the data based on their location and then plotted in on an interactive map. When navigating the interface, clicking a marker makes all the information about that location pops up, including all the wifi devices, their device name, privacy protocol etc. ![All Access Points on map](/assets/img/seasame_fox/all%20aps.png) 

Also, next to this is a table that contains every single wifi device (routers and clients). Not only this, but you can also search through all these devices to find specific information about the dataset i.e how many devices are Optus devices, which area has the most wifi devices or which devices have bad privacy protocols. ![Table of all APs](/assets/img/seasame_fox/blurred%20sc.png)

Part 4: Actual hacking
----------------------

\*\*\* Disclaimer: All the stuff below was not actually performed. It is illegal to break into another person’s device without consent. Don’t do it \*\*\*  
  
Now that all the creepy stalker, mass surveillance stuff is done, lets do some hacking!  
  
The next part will describe things that could potentially easily be executed with only a few lines of code. This was never actually tested out because of its illegal nature but this is just to illustrate the capabilties of the device.  
  
The first thing I could use this device for was WPA/WPA2 handshake capturing. Let me briefly explain.  
  
Most wifi routers use WPA/WPA2 protocol which is relatively secure. However, if you can capture the initial connection (handshake) between a device (such as your phone) and the router, you can extract a lot of information. If you can capture the handshake, you can get the hashed password of the router (pretty much an encrypted password).  
  
With the hashed password, you can brute force this using a wordlist of common passwords to find the password for the wifi router. What my device can do is disconnect devices from a router and then “listen” to the handshake between the device and the router. When the handshake is captured, later on, it can be attempted to be cracked. Don’t be too scared though. This attack is only vulnerable for people who have their password as “password123” or “123456” so just make sure you don’t have a very common password.  
  
There is a program called “besside-ng” which can do this whole process for me. The disconnecting, the capturing, the logging. All I had to do was create a daemon to run this every couple of minutes.  
  
Another quite simple attack I could do with my device is wifi jamming (via deauthorising). This is another attack that is extremely easy to do (but quite illegal). All I would have to do is scan for all the wifi routers and then constantly send deauthorisation packets. This would constantly be disconnecting all the devices from the router. This pretty much means that I could prevent people from using wifi. There are definitely some nefarious use cases you could use this for.  
  

Part 5: Future possibilities and improvements
---------------------------------------------

There were a few things that could be improved about the device. For example, when inside buildings, the GPS antenna is not good enough to pick up a signal, this means that no information will be logged. Another thing, the box has to be opened and then plugged in to the power bank and then closed. If I wanted the device on for a long time, this would be tedious.  
  
Redesigning the box to make it be able to be on for hours at a time and getting a better GPS system could make the device better.  
  
This is a fun little project but it could be easily scaled up as a mass surveillance technology. Having a swarm of these that could work together, having a stronger wifi antenna and making the device able to be left on constantly could make this extremely powerful.  
  
From the data analysis side, a lot more could be done as well. Finding the density of areas depending on the amount of devices there are, looking for patterns in recurring device ids etc.  
  

Conclusion
----------

I was very happy with how the project turned out. Don’t be fooled by the seemingly smooth flowingness of everything. At every turn there was another setback, another thing that didn’t work. The whole process required a lot of perserverence, I just neglected to mention these because it would make the post overly long and boring.  
  
I tested this device out in public and I was absolutely stunned with the amount of data I received. The device was only turned on for about 20 minutes and I was walking through a suburb at night when there weren’t that many people and in this time period I logged 1000 access points (routers/hotspots) and 500 clients (devices such as phones) (1500 data points). It is crazy to think that this was done with a budget of about $30 ($10 raspberry pi + $15 gps + $5 power bank). Let’s suppose I had it on for 12 hours instead and I had 10 of these scattered around the place. Every day, I would collect over 375,000 (1500 \* 25 \* 10) data points, 2.5 million per week. This is all with a budget of $300 (10 devices).  
  
This project was done with one of my courses at university all about security and privacy and this really opened my eyes about this issue. If someone with a budget of $300 could collect 136 million data points in a year, how much could massive companies with billions of dollars of net worth or massive governments collect?  
  
Some people would think that this information is useless. What am I learning from this? Who cares if you can see what devices are on in a small radius? However, the data really starts to get powerful once you have linkage. Once I can start to link data together it gets really dangerous. If I had this device on for a year, I could track moving patterns of people since each device has a unique id. I can get information on how busy certain places are at different times of the year. I can see which devices are using outdated security and target them. I can find what brand certain routers are and exploit vulnerabilities. If I could link the unique device id with an individual, I could essential track where they are at all times. All this information is valuable but can also be dangerous. Stay safe kids.
