---
layout: post
title: "VPNs! An OPSEC Primer"
date: 2017-11-12 12:05:24 +0300
comments: true
categories: [vpn]
---

The internet is a glorious place. This gloriousness does often come with some pitfalls here and there. Being probably the greatest source of information ever, various actors are constantly devicing ways to acquire and make use of all this data. 

<!--more-->

Businesses want to mine this data for monetisation purposes through targeted ads and the like. Criminals want it for malicious intent and profit. Governments and intelligence agencies want it for law enforcement and often censure and control. ISPs want it for monitoring, traffic filtering and so on. The list is endless and the motives ranging from good to bad are almost always controversial.

A lot of the internet community is ok with the status quo and unlikely to be overly concerned with all this. Whether they should be, is a debate that has been explored extensively and therefore something this post will not dwell on. 

There are however a lot of us that prefer some form of control over our data as we interact with the World Wide Web. There are those who may have reasons to bypass restrictions placed upon their internet activity as prescribed by various entities mainly ISPs. Then there are those whose work demands operational security as they go about their business. These could include <a href="https://en.wikipedia.org/wiki/Red_team" target="_blank">**red teams**</a>, law enforcement, activists and even criminals. 

Here, I will try to cover the various ways these groups can use <a href="https://en.wikipedia.org/wiki/Virtual_private_network" target="_blank">**virtual private networks**</a> and the merits and weakness of each scenario. I will do so by providing a rating of the operational security each scenario affords them.

# Typical internet connection

Let's begin by describing the typical way most users access the internet.

{% img /images/typical.png %}

This is usually the basic internet connection. The user connects to the network through a router which connects them to their ISP and out into the internet.

### OPSEC Rating: **Low**
* ISP observes all connections between client and the internet
* ISP can view all clear text communication
* ISP can see all destination and source IP/port info between client and the internet
* ISP can block or filter certain types of traffic

As can be seen, the typical internet connection places almost or the control with the ISP.

# Typical VPN connection

A typical VPN setup involves the client connecting to a VPN gateway/provider over an encrypted tunnel. Their internet traffic therefore goes through this tunnel before being directed out to the internet by the VPN gateway/provider.

{% img /images/typical_vpn.png %}

### OPSEC Rating: **Medium**
* ISP can only see destination and source IP/port info between client and VPN Gateway
* ISP blocking and filtering of traffic is bypassed by use of the VPN tunnel
* No clear text communication as traffic is encrypted in the VPN tunnel
* ISP can correlate connection details between the Client and it's other clients
* Hoster of VPN Gateway can correlate traffic between Client and the internet 

This VPN connection will meet the needs of most users. It however fails if the intention is good anonymity. 
The first problem is the following scenario: let's say the client wants to anonymously connect to a server owned by another client hosted on the same ISP. They will connect to the VPN Gateway which will then forward them to the server. I hope you see the problem here. First, the ISP can see that the client is connected to the VPN Gateway. Secondly, the ISP can also see the VPN Gateway connecting to the server as this server is owned by another client hosted by them. The ISP can therefore correlate the two connection details even though a VPN gateway was involved.

The second problem is that the VPN Gateway can basically see all the connection details between the client and the internet.

# Chained VPN connection

To overcome some of the challenges identified in the typical VPN connection, a chained VPN connection can be set up as follows.

{% img /images/chained_vpn.png %}

### OPSEC Rating: **Good**

* ISP can only see destination and source IP/port info between client and first VPN Gateway
* ISP blocking and filtering of traffic is bypassed by use of the VPN tunnel
* No clear text communication as traffic is encrypted in the VPN tunnel
* ISP cannot correlate connection details between the Client and it's other clients 
* Hoster of VPN gateway one cannot correlate traffic between Client and the internet
* Hoster of VPN gateway two can correlate normal internet browsing and traffic to the sensitive network

This vpn connection provides relatively good anonymity as opposed to the typical one described earlier. It can be seen that the ISP never sees the connection between the two VPN Gateways and therefore cannot link the client to VPN Gateway Two. As all internet traffic goes out through VPN Gateway Two, VPN Gateway One provides an added layer of security against correlation weaknesses of the typical VPN connection.

It should however be noted that VPN Gateway two could possibly identify the client through correlation of metadata information should the client perform personally identifiable internet activity. A likely scenario where this would occur is the client accessing their personal social media or email accounts (metadata such as urls can give away this information). If the client then uses the same VPN connection for other purposes such as accessing the sensitive network shown in the diagram, VPN Gateway two can correlate this to potentially identify the client.

# Chained VPN connection with selective routing

A variation of the chained VPN connection could be set up as follows.

{% img /images/chained_selective.png %}

### OPSEC Rating: **Good**

* ISP can only see destination and source IP info between client and first VPN Gateway
* ISP blocking and filtering of traffic is bypassed by use of the VPN tunnel
* No clear text communication as traffic is encrypted in the VPN tunnel
* ISP cannot correlate connection details between client and sensivite network 
* Hoster of VPN gateway one cannot correlate normal internet browsing and traffic to the sensitive network
* Hoster of VPN Gateway two cannot observe Client's normal internet browsing

In this setup, normal internet traffic from the client is forwarded directly to the destination by VPN Gateway One and isn't forwarded to VPN Gateway Two. Select traffic that may be considered sensitive is however explicitly forwarded to to VPN Gateway Two.

What this achieves is to ensure that the client's normal traffic such as normal web surfing doesn't go out the same VPN Gateway as traffic they wouldn't want linked to them. VPN Gateway One will only see normal internet traffic while VPN Gateway Two will only see traffic to the sensitive network.

# Takeway notes

* The typical vpn connection should suit most users who just need basic anonymity and security.
* The two chained options provide improved anonymity.
* Use of any of the VPN setups described alone isn't to be assumed to guarantee operational security. Opsec hygiene should be observed and complementary solutions and techniques employed.
* An advanced adversary such as a nation state will defeat the setups described with a high likelihood.

# Addendum

I'll be doing a series on how to implement the various setups using my favorite VPN solution, the extremely simple yet fast solution VPN called <a href="https://www.wireguard.com/" target="_blank">**Wireguard**</a>.
As part of the series, I'll also detail how to setup the portable Wireguard VPN solution shown below using a Raspberry Pi.

{% img /images/portable_vpn.png %}

Here are some useful links related to this post.

* Some great opsec rules from https://grugq.tumblr.com/post/60463307186/rules-of-clandestine-operation 

* The awesome Wireguard VPN https://www.wireguard.com/ 

* My go to network diagram tool https://www.draw.io/


A typical VPN connection using Wireguard follows next.

Till next time, happy hacking!