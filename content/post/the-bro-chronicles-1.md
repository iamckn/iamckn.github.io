+++
title = "The Bro Chronicles 1"
date = "2015-09-06 08:56:06 +0300"
tags = ["network security"]
+++
I was having a random conversation with [**Christian**](https://twitter.com/xtian_kisutsa). The conversation was based around Kali Nethunter and all the cool stuff that can be done with it. The discussion was revolving around "man in the middle attacks" and I suggested that having [**BRO**](https://www.bro.org/) running on Nethunter would be awesome. This got him excited and true to his nature, he couldn't rest till he'd attempted it. Long story short, he did some amazing work and had it running soon afterwards complete with a detailed "how to" guide. Here's a link to the [**guide**](http://t.co/Iz992aMCL5).

<!--more-->

With bro running on a phone, a world of exploration is opened up. You now have an enterprise level IDS running on your phone :). I hope you see where this is headed. What do you do with an IDS on your phone? You obviously investigate what your phone communicates with. 
What interested me most was the `rmnet0` interface on android. This is the interface assigned to your data connection. It gets assigned a public IP by your telecommunications provider so no NAT issues.

# The set up
This Saturday afternoon I decided to see what my phone communicates to over a one hour period while connected to the mobile data network.
This is how I set up things:

- Nethunter running on a Nexus 5 with Android 5.1
- BRO running in standalone mode monitoring rmnet0
- Mobile data connection on the Safaricom network
- ADB connection to a laptop
- shodan-cli to investigate IPs

Once everything was in place I used the phone as I would normally for about 10 minutes or so - browsed a bit and opened a few apps such as Twitter and Facebook. After that I left it idle for about an hour.

# Results and Analysis
It was then time to process and analyse the results. Nethunter runs in a chroot environment so I transferred the BRO logs to the sdcard then to my laptop, that's where adb comes in handy.
Lets start with the ports and IPs that were most active during both normal usage of the phone and when it was idle.

## Idle
```javascript
	 Ports        | Sources                   | Destinations              |
     443    38.5% | 105.57.36.93#1      99.0% | 196.201.216.21#2    20.0% | 
     53     25.5% | 196.201.216.4#3      0.5% | 216.58.223.10#4     10.0% | 
     123    24.0% | 66.196.116.132#5     0.5% | 196.49.6.67#6        9.0% | 
     993     3.0% |                           | 196.201.218.15#7     6.5% | 
     80      3.0% |                           | 197.12.0.14#8        5.0% | 
     995     1.5% |                           | 196.25.1.5#9         5.0% | 
     8996    1.0% |                           | 197.84.150.123#10    3.5% | 
     5228    1.0% |                           | 199.16.156.199#11    3.0% | 
     3       1.0% |                           | 62.210.87.71#12      3.0% | 
     55602   0.5% |                           | 216.58.223.14#13     2.0% | 
```
## Normal use
```javascript
	 Ports        | Sources                   | Destinations              |
     80     67.7% | 105.161.166.138#1   99.9% | 196.201.217.7#2     14.6% |
     53     15.8% | 66.196.116.132#3     0.1% | 173.241.248.221#4   11.4% |
     443    15.0% | 174.37.231.85#5      0.0% | 173.241.248.220#6    5.2% |
     993     0.4% |                           | 94.31.29.96#7        3.2% |
     995     0.3% |                           | 198.232.124.225#8    2.5% |
     8996    0.2% |                           | 192.0.77.2#9         1.7% |
     3       0.1% |                           | 31.13.75.8#10        1.7% |
     5228    0.1% |                           | 46.228.164.11#11     1.6% |
     123     0.1% |                           | 95.172.94.57#12      1.5% |
     57553   0.0% |                           | 46.228.164.13#13     1.4% |
```
As expected most traffic is either HTTP (80), HTTPS (443) or DNS (53). NTP traffic (123) also features prominently when the phone is idle.
Of interest is that we can confirm Safaricom uses their own DNS servers, in this case `196.201.216.21`, `196.201.218.15` and `196.201.217.7`. An OpenDNS server `208.67.222.222` was also queried once so they probably have OpenDNS as an option.
The ntp servers were distributed among African countries, Angola - `196.223.19.3`, Tunisia - `197.12.0.14`, SA - `197.84.150.123` and so on. We can therefore safely say they use the africa.pool.ntp.org pool.
Then comes the interesting bit, there were some Safaricom IPs communicating with the phone on some weird ports, one case is `196.201.216.4`, Shodan reveals port `500` and `4500` open on whatever is running on that IP. Let's leave Safaricom for now and look at the other traffic.

The BRO logs are pretty detailed and therefore I needed some way to clean them up to do my own analysis. I therefore wrote a bash script to do some filtering.
I then used Shodan-cli to get information of every unique IP and save the output. Here is the script for anyone interested.
```bash
	#!/bin/bash

	echo "Enter the name of the connection log: "
	read LOG
	echo "Getting details from shodan"
	`sed -e '/^#/d' $LOG | cut -f 5 | sort -u > filtered`
	touch output
	
	while read -r line; do
		echo $line
		`sudo shodan host $line >> output`
		echo "">**> output
	done < "filtered"
	echo "Done! Output saved to output"
	
	exit 0 	
```
I am going to cut right to the chase as this post is getting too long. The world is being taken over by analytics, advertising and digital content companies!
Let me just list a few of the companies that communicated with my phone over that one hour period, remember I left it idle most of that time. Here we go
```bash
	Mediamath.com 		 Mixpane.com 		 Criteo.com
	Fastly Inc 			AOL			   		AppNexus
	Akamai 				Chango   			Trapad
	Exelate.com 		 	Openx.com    		Adtech.de
	Lijit.com 			Crashlytics.com   	Pubmatic
	ACNielson 			Amazon  			...and several others
```
I usually am not worried about companies using analytics for their marketing purposes but this time something spooked me.
Several of these servers are vulnerable to the [**logjam attack**](https://weakdh.org/) and even more worryingly some are vulnerable to heartbleed. It would therefore not be far-fetched to assume they've already been compromised or they will be. With all the traffic going on between my phone and these servers, I had hoped they'd have their security figured out. It's often said that third parties are some of the most promising targets for hackers and you can see exactly why here.

# Conclusion
That was a long post so let's wrap it up.
There were some peculiar IPs that I'll be investigating in subsequent BRO chronicles. Other random titbits include the discovery that several Instagram servers communicating with my phone are also vulnerable to the logjam attack, Yahoo mail is particularly noisy traffic wise and so on.
When going through all that analytics and ad based traffic I remembered a quote by Jeffrey Hammerbacher, one of Facebook's first 100 employees. I therefore aptly end this post with his quote:

>## The best minds of my generation are thinking about how to make people click ads. That sucks.
