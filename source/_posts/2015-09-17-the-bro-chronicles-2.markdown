---
layout: post
title: "The BRO Chronicles 2"
date: 2015-09-17 20:54:07 +0300
comments: true
categories: ["the network project"]
---
####In part one I analysed the data communication that takes place on a phone over a one hour period. Read the post <a href="https://www.ckn.io/blog/2015/09/06/the-bro-chronicles-1/" target="_blank">here</a> for details and background info. 
####Here, I go further and analyse what happens over a twelve hour period of normal phone usage. I ran <a href="https://www.bro.org/" target="_blank">BRO</a> on my phone from `6:40am` to `6:40pm` and ensured that data was on and WIFI off the whole time.

<!--more-->

####Let me start with figures of some standout statistics.
>####The phone communicated with `233` unique IPs from diverse parts of the globe over that period.

####I broke this down further to specific ports of interest. Note that my IP was `105.52.174.7` during that period.

##Port 21 : FTP
```javascript
	Unique IPs : 2

	Source			Destination
	105.52.174.7 		158.69.128.126
	180.97.106.37		105.52.174.7
```
####Why my phone wanted to connect to `158.69.128.126` on FTP raises some interesting questions.

##Port 22 : SSH
```javascript
	Unique IPs : 6

	Source			Destination
	180.97.106.36 		105.52.174.7
	218.65.30.23 		105.52.174.7
	42.116.163.127 	105.52.174.7
	80.82.65.123 		105.52.174.7
	89.248.168.28 		105.52.174.7
	94.102.49.102 		105.52.174.7
```

##Port 23 : Telnet
```javascript
	Unique IPs : 24

	Source			Destination
	101.29.2.192 		105.52.174.7 23
	106.115.98.232 	105.52.174.7 23
	110.195.77.199 	105.52.174.7 23
	and 21 others
```
####That translates to an average of two telnet connection attempts per hour from unique IPs!

##Summary of some other ports of interest
```javascript
	Port 		Unique IPs 	
	53			24
	25 			1
	80 			129
	123			26
	443 			218
	993 			4
	995 			1
	1900 			3
	5060 			1
	5222 			51
	5228 			7
	8080 			4
	8996 			1
```
####These results immediately direct our attention to internet wide scanning. This has slowly become a growing phenomenon over the past two years and has been aided by the release of extremely fast scanning tools. We have <a href="https://zmap.io/" target="_blank">zmap</a> created and maintained by a team from the University of Michigan which can scan the entire IPv4 space in under 5 minutes with a ten gigabit ethernet connection. Masscan, created by Robert Graham of Errata Security is fast enough to do the same in under <a href="http://blog.erratasec.com/2013/09/masscan-entire-internet-in-3-minutes.html" target="_blank">3 minutes</a>!
####Internet wide scanning is being used a lot by researchers to investigate vulnerabilities and do various internet wide studies. A case in point is that several internet wide scans were done targeting the heartbleed vulnerability within 48 hours of disclosure. A more recent case is this week's <a href="https://zmap.io/synful/" target="_blank">synful attack scan</a> done by the zmap team on September 15, 2015.
####<a href="https://www.shodan.io/" target="_blank">Shodan</a> does regular internet wide scans and hosts the data on their site which is made available to the public in an interactive way.
####The University of Michigan team (same one that maintains zmap) also does daily scans and hosts the raw data on the <a href="https://www.scans.io/" target="_blank">scans.io</a> site. They also host scans done by other researchers such as the Rapid7 team which does various scans regularly.
####The flip side of this is that a lot more people do these scans with malicious intent. They're always on the lookout for vulnerable systems on the internet and their activity spikes every time a major vulnerable is released. They mostly hide behind <a href="https://en.wikipedia.org/wiki/Bulletproof_hosting" target="_blank">bulletproof hosting providers</a> or run their scans from countries likely to be lenient, with China featuring prominently.
####Let me now relate the activity on my phone with internet wide scanning.

##Some known research scans I observed during that 12 hour period
```javascript
	IP 				Domain Name
	169.229.3.90		researchscan0.EECS.Berkeley.EDU
	216.218.206.69		scan-08.shadowserver.org
	74.82.47.20 		scan-12e.shadowserver.org
	184.105.247.204	scan-15b.shadowserver.org
	74.82.47.29		scan-12e.shadowserver.org
	188.138.9.50 		atlantic.census.shodan.io
	93.120.27.62		m247.ro.shodan.io
	71.6.167.142		census9.shodan.io
	71.6.165.200		census12.shodan.io
	198.20.70.114		census3.shodan.io 
	188.138.9.51		atlantic482.serverprofi24.de
	80.82.70.198		icsresearch1.plcscan.org
	141.212.122.181	researchscan436.eecs.umich.edu
	141.212.122.182	researchscan436.eecs.umich.edu
```
##And of course several scans from unknown actors
```javascript
	IP 				Country
	124.193.141.130	China
	223.167.14.33		China
	123.151.149.222	China
	104.245.101.253	US
	199.203.59.120		Israel
	125.253.126.75 	Vietnam
	80.82.70.238		Netherlands
	115.230.124.164	China
	177.44.69.36		Brazil
	119.235.95.174		Fiji
	125.24.223.93		Thailand
	220.134.101.246	Taiwan
	37.145.43.253		Russia
	95.59.98.19		Kazakhstan
	91.236.75.224		Polland
	46.10.74.117		Bulgaria
	186.45.49.254		Trinidad
	202.12.81.138		India
	211.247.102.117	South Korea
```
####It is evident that every day, anything you put on the internet will be poked at unless you are behind some NAT network and/or have some form of firewall filtering traffic. 
####The takeaway is that you really should keep your internet facing systems updated and with as good a security posture as possible. Watch out for newly released vulnerabilities and patch them as fast as possible. Past cases indicate that 48 hours later may already be too late. 
####Implement defences on the edge of your network and monitor your logs regularly and diligently. Try out the awesome BRO IDS (I am not being paid to promote it incase you're wondering).
####Some of these scanners are extremely aggressive. To demonstrate just how aggressive, I leave you with this Netherlands IP that was scanning port numbers that have the digits 443.
```javascript
	Source IP 		Destination IP 	Destination Port
	80.82.70.238 	105.52.174.7 	1443
	80.82.70.238 	105.52.174.7 	2443
	80.82.70.238 	105.52.174.7 	3443
	80.82.70.238 	105.52.174.7 	4431
	80.82.70.238 	105.52.174.7 	4435
	80.82.70.238 	105.52.174.7 	4436
	80.82.70.238 	105.52.174.7 	4438
	80.82.70.238 	105.52.174.7 	4439
	80.82.70.238 	105.52.174.7 	4443
	80.82.70.238 	105.52.174.7 	5443
	80.82.70.238 	105.52.174.7 	6443
	80.82.70.238 	105.52.174.7 	7443
	80.82.70.238 	105.52.174.7 	10443
	80.82.70.238 	105.52.174.7 	11443
	80.82.70.238 	105.52.174.7 	22443
	80.82.70.238 	105.52.174.7 	33443
	80.82.70.238 	105.52.174.7 	44310
	80.82.70.238 	105.52.174.7 	44322
	80.82.70.238 	105.52.174.7 	44366
	80.82.70.238 	105.52.174.7 	55443
```	


