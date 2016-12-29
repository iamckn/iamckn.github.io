---
layout: post
title: "GSM: Sniffing SMS traffic"
date: 2015-11-29 10:14:48 +0300
comments: true
categories: [radio]
---
In the previous <a href="https://www.ckn.io/blog/2015/11/01/sniffing-gsm-traffic/" target="_blank">**post**</a>, I explained how GSM traffic can be sniffed with the <a href="https://www.ckn.io/blog/2015/10/11/the-hackrf-one-first-steps/" target="_blank">**HackRF One**</a>. GSM traffic carries a lot of information, from system information to the actual voice and data we are familiar with. The traffic that the normal user of a telecommunication network is concerned with is voice and data. With this in mind I'll do a two part series to demonstrate how voice and data can be sniffed using the HackRF. I start with SMS traffic which falls under the data category. Let's get right into it!

<!--more-->

I am going to send an SMS from a Safaricom line to an Orange line, capture the traffic over the <a href="https://en.wikipedia.org/wiki/Um_interface" target="_blank">**Um (air) interface**</a> and decrypt the data to retrieve the SMS. I own both lines steering clear of the legality issue of decrypting other people's traffic.

##**Identifying the BTS**
The specific point at which I'll capture the traffic is as it's being sent by the BTS to the Orange line. The technical term for this is the downlink. I therefore need to identify the BTS that my Orange line is connected to. A BTS is uniquely identified using an assigned cell identity (Cell ID). The cell identity combined with the <a href="https://en.wikipedia.org/wiki/Location_area_identity" target="_blank">**location area identity (LAI)**</a> which uniquely identifies the country, mobile network and location area code is what we need to get. There are various ways to get this information such as the engineering menu on blackberries. The phone I am using is an android phone and there are several android apps that will give you this information. In my case I use the awesome <a href="https://github.com/SecUpwN/Android-IMSI-Catcher-Detector" target="_blank">**Android IMSI Catcher Detector**</a>.
Next we sniff the GSM frequencies our mobile operators use and identify the specific frequency the BTS is operating on. Follow the previous <a href="https://www.ckn.io/blog/2015/11/01/sniffing-gsm-traffic/" target="_blank">**post**</a> on how to do this. We will accomplish this by searching the traffic being captured on wireshark for the LAI and Cell ID our phone is on until we have a match. In this case the frequency the Orange BTS was operating on is 949.2MHz.

##**Capturing SMS traffic**
I'll use the airprobe_rtlsdr_capture module of <a href="https://github.com/ptrkrysik/gr-gsm" target="_blank">**gr-gsm**</a> to capture the SMS traffic. I begin the capture using the following command:

```bash
airprobe_rtlsdr_capture.py -f 949200000 -s 1000000 -g 40 -c capture.cfile -T 60
```

**-f** is the frequency in Hz, **-s** the sample rate in Hz, **-g** the gain, **-c** the output file and **-T** the duration of our capture in seconds.

I then send an SMS reading "This is a demo of GSM decryption" to the Orange line. 

##**Getting the TMSI and Kc**
We now have the traffic captured and saved in a file called **capture.cfile**. Before we get into the decryption process, we need some information specific to our SIM card.
First we need the Temporary Mobile Subscriber Identity (**TMSI**). This is a random assigned identity assigned to the SIM when it connects to a BTS. Its purpose is to avoid the subscriber from being identified and tracked by eavesdroppers on the air interface as I am trying to do :). 
We will then need to get the **Kc**, which is the key used to encrypt the traffic between the phone and the BTS over the air. I will get into the details of how the **Kc** is calculated in a later post but for now you can read up on the **A8** algorithm.
There are various ways to get the **TMSI** and **Kc**. These guides <a href="http://domonkos.tomcsanyi.net/?p=369" target="_blank">**here**</a> and <a href="http://openbsc.osmocom.org/trac/wiki/A5_GSM_AT_tricks" target="_blank">**here**</a> are great references.
I used AT commands on a connected Samsung S3 mini as follows:

```bash
AT+CRSM=176,28448,0,0,9
+CRSM: 144,0,0E10EAF30299F2C404
```
The **Kc** from the output is **0E10EAF30299F2C4**

```bash
AT+CRSM=176,28542,0,0,11
+CRSM: 144,0,38256E5136F9702B5D0000
```
The **TMSI** from the output is **38256E51**

##**Decoding BCCH**
BCCH is the Broadcast Control Channel that the BTS uses to communicate system information messages to the mobile device on. 
In idle mode the phone has to listen on the BCCH to detect traffic to be sent to it.
In this case the BTS sends two paging request messages which inform the phone of an incoming SMS.
This will then be followed by an Immediate Assignment where an SDCCH is established for the SMS transfer.
SDCCH is a Standalone Dedicated Control Channel used for most short transactions, including initial call setup step, registration and SMS transfer.
We therefore need to identify what SDCCH is used for the SMS transfer. 
We first start wireshark and monitor the loopback interface and then run the following command:

```bash
airprobe_decode.py -c capture.cfile -s 1000000 -f 949200000 -m BCCH -t 0
```
**capture.cfile** is the file with the traffic we captured earlier.
We then search for traffic specific to our TMSI by searching for it in wireshark packet details.

{% img /images/tmsi_search.png %}

we look for the two paging requests and inspect the Immediate Assignment that follows:

{% img /images/sdcch_channel.png %}

Note that it's **SDCCH/8**, **Timeslot 2**.

##**Decoding SDCCH**
We now need to identify the ciphering mode the BTS tells the phone to use. 
We restart wireshark on the loopback interface and then run the following command specifying **SDCCH8** and **Timeslot 2**:

```bash
airprobe_decode.py -c capture.cfile -s 1000000 -f 949200000 -m SDCCH8 -t 2
```

We look for a Paging Response followed by a Ciphering Mode Command.

{% img /images/cipher_mode.png %}

We see that the algorithm in use is **A5/1** (more on this in a later post).

##**Decrypting the SMS traffic. Finally!**

We confirm the most recently used **Kc** as shown earlier, then we restart wireshark on the loopback interface and run the following command:

```bash
airprobe_decode.py -c capture.cfile -s 1000000 -f 949200000 -m SDCCH8 -t 2 -e 1 -k 0x0E,0x10,0xEA,0xF3,0x02,0x99,0xF2,0xC4
```
**-e 1** specifies the algorithm **A5/1**, **-k 0x0E,0x10,0xEA,0xF3,0x02,0x99,0xF2,0xC4** specifies the **Kc**.

On wireshark we look for the GSM SMS traffic and we can see the text of our SMS :).

{% img /images/decrypted_sms.png %}

##**Wrapping up**
If you've followed through keenly you'll note that every variable is available readily from just sniffing the traffic except the **Kc**. Everything else is exchanged between the BTS and Mobile device, from the TMSI and BTS details to the data traffic though encrypted. The question becomes how do you get the **Kc** in use by another mobile device? The good people at <a href="https://srlabs.de/decrypting_gsm/" target="_blank">**SRLabs**</a> figured out how to crack the A5/1 encryption used and created a tool called **Kraken** and 2TB rainbow tables to find the **Kc**.
They also automated a lot of the steps in this post making decryption of gsm voice and data almost trivial.

In the next post I'll demonstrate how to decrypt voice traffic :) as I continue explaining GSM communication.
Till then, happy hacking. 
