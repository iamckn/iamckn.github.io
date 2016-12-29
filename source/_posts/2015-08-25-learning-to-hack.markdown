---
layout: post
title: "Learning to hack"
date: 2015-08-25 14:27:24 +0300
comments: true
categories: [ramblings]
---
Becoming a skilled hacker or the politically correct "information security specialist" doesn't have a straightforward defined path. Here I will try to layout what constitutes a good way to go about it.

<!--more-->

#Bits and Pieces - Digital Logic

Everything starts here. Every computer process is just sequences of simple logic operations.
Learn about AND, OR, XOR and so on. Go through the full course material if you're already being taught this in school. If not, don't skim through this, dedicate enough time to it. This is interesting stuff for the hacker mind so it shouldn't be too taxing.
Ideally take it deeper to the circuit level implementations. Study NAND gates and so on. If you're so inclined go even further to diodes and transistors. With this you'll be primed for the next area - Assembly Language.

#POP POP RET - Assembly Language

A lot of people will avoid Assembly due to the false impression that it's very complex. It really isn't, infact it's a lot more fun and rewarding. If you took time on your digital logic and you aren't afraid of a bit of math, this will be a lot easier to grasp.
You'll get to learn how the processor runs at a low level. You'll get to learn about instructions, registers and programming logic. Once you've got this figured out, higher level languages will be a breeze. You'll get to understand how your programs interact with different hardware. Best of all you'll spare yourself the agony of trying to figure it out when you try your hand at things like exploit development, reverse engineering and hardware hacking.
There are free emulators out there that will help you along the process and make the experience fulfilling. Try lighting some LEDs or programming some hardware. Once you're done with this, you'll be able to hold your own any in almost any information security field you decide to pursue.

#PCAP or it never happened - The Network

Next you need to learn how devices and services communicate. Start from the basics and build up from there. Study the OSI model, network devices and related services. Learn as much as you can about IPv4, IPv6, subnetting, routing, switching, DNS, DHCP and so on. From there move on to network security and the various attacks and countermeasures. Master the theory behind attacks such as MAC flooding, DNS and DHCP hijacks and so on. Learn about common defenses and network setups, Firewalls, IPS's and IDS's, DMZ's, VLANS, VPNS and so on.
A straightforward way would be to go through the Cisco Network Certification material, you can even get yourself certified along the way :)

#The Box - *nix

You need a system that is powerful and flexible. This is where linux and the other variants of the *nix family come in. If you're just beginning, set up a virtual machine or dual boot your computer. I would suggest you begin with the latest stable version of Ubuntu to get yourself going and make the transition from Windows smooth. Learn how to set it up and how to do various tasks on the system. Learn about various commands and how to automate process using scripts. All this is still at a high level so the learning curve is still manageable. Once you're comfortable, you can transition to using it as your primary OS. 
Next dive into the guts of the system. Learn about the building blocks of the system and how everything fits in together. One way is to build your linux system from the ground up. Distros like ArchLinux and Gentoo that require you to build from an initial shell interface are an option you can use in the learning proces. The Arch Wiki for one is amazingly comprehensive and structured for both the beginner and the advanced user. 
Kali Linux then comes in. Once you understand Linux well enough, you can let the good people at Offensive Security do the work for you. Kali Linux is a powerful and well built system that will give you access to tools that you'll use a lot in your information security work.

#Explore the world - The organisation

The real world is made up of people and the decisions they make. It is therefore highly important that you understand how they operate in various environments. 
I therefore suggest that you find a way of working in an organisation and experience how information security is approached. Intern as a system admin or whatever position that allows you to interact with systems and people in IT departments. Learn the common practices and errors that are made along the way. Get an internal perspective so that your thought process takes into account what goes into building and setting up systems. 
You don't want to gain temporary access to a network and start wondering how a domain controller ties in to the network. 
Being part of an organisation also will give you insight into the behaviour of various people in the organistation. Understanding how everyone from the tech savvy to the management react to various situations will come in handy as you progess.

#Tying up the pieces

There are a lot of areas you can focus your attention on along your information security journey and that is up to you. Experiment, change, adapt, and tweak what you need to. 
Some things will remain throughout though. 
You will need to be creative to see beyond the black and white. 
You'll need passion to take you through hours of research and reading through RFCs...hehe. 
You'll need some code to live by or you may get lost along the way.

Being a hacker isn't about writing the most complicated exploit, it is about figuring out elegant solutions to some problem - easy or challenging. 
In the end it's about the thrill of discovery and creation.

I leave you with this book by James Gleick that I believe every information security person should read at least once http://www.amazon.com/The-Information-History-Theory-Flood/dp/1400096235