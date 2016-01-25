---
layout: post
title: "GSM: Sniffing voice traffic"
date: 2016-01-25 06:51:00 +0300
comments: true
categories: [radio]
---
####I wrap up the GSM series with a walkthrough on how to decrypt voice traffic. Voice is the way most people interact on a telecommunications network and therefore a major componenent of GSM traffic. I've explained a lot of the background on GSM communication in the previous posts so I'll get right to it.

<!--more-->

####We will capture the traffic using the **HackRF one** and the call will take place between two Safaricom lines. The capture will take place on the downlink - that is the receiving end of the call. I'll use a Blackberry as the receiving device so that I can easily get the TMSI and Kc.

##**Capturing the traffic**

####I'll speed through a lot of these steps as they are similar to the <a href="https://www.ckn.io/blog/2015/11/29/gsm-sniffing-sms-traffic/" target="_blank">**sniffing SMS traffic scenario**</a>. I'll use the  Absolute Radio Frequency Channel Number <a href="http://www.telecomabc.com/a/arfcn.html" target="_blank">**ARFCN**</a> in specifying the radio channel. GSM uses ARFCNs to represent the various frequencies the BTS and mobile device communicate on. I'll use the ARFCN instead of the frequency in the commands I'll run for variety as I've been using frequency in the previous posts. 
####We begin by getting the ARFCN, TMSI and Kc from the Blackberry. On a Blackberry these are readily available from the engineering screen menu. The ARFCN is gotten by navigating to the **Cell Identity** submenu, the TMSI from the **Mobile Identity** submenu and finally the Kc from the **SIM Browser** submenu.

####The values I get are:

####ARFCN: **17**
####TMSI: **8D4812F8**
####Kc: **239E4C213612C000**

####I use the airprobe_rtlsdr_capture module of <a href="https://github.com/ptrkrysik/gr-gsm" target="_blank">**gr-gsm**</a> to capture the voice traffic. I begin the capture by running the following command:

```bash
airprobe_rtlsdr_capture.py -a 17 -s 1000000 -g 40 -c voice_capture.cfile -T 150
```

**-a** is the ARFCN, **-s** the sample rate in Hz, **-g** the gain, **-c** the output file and **-T** the duration of our capture in seconds.

####I then make a call while the capture is in progress.

##**Decoding BCCH**
####As explained in the previous post, in idle mode the phone has to listen on the BCCH to detect traffic to be sent to it.
####Our aim here is to identify what SDCCH (Standalone Dedicated Control Channel) is used for our call setup.

####We first start wireshark, monitor the loopback interface and then run the following command:

```bash
airprobe_decode.py -c voice_capture.cfile -s 1000000 -a 17 -m BCCH -t 0
```
####**voice_capture.cfile** is the file with the voice traffic we captured.
####We then search for traffic specific to our TMSI by searching for it in wireshark packet details.
####we look for the paging request and inspect the Immediate Assignment that follows:

{% img /images/SDCCH_Search.png %}

####Note that it's **SDCCH/8**, **Timeslot 1**.

#**Decoding SDCCH**
####We now need to identify the ciphering mode the BTS tells the phone to use. 
####We restart wireshark on the loopback interface and then run the following command specifying **SDCCH8** and **Timeslot 1**:

```bash
airprobe_decode.py -c voice_capture.cfile -s 1000000 -a 17 -m SDCCH8 -t 1
```

####We look for a Paging Response followed by a Ciphering Mode Command.

{% img /images/ciphering_mode.png %}

####We see that the algorithm in use is **A5/1**.

##**Decoding TCH**

####TCH is the Traffic Channel in GSM and is used to carry voice traffic and data. It could either be full rate TCH/F or half rate TCH/H. You can read up more on it <a href="http://www.rfwireless-world.com/Terminology/GSM-traffic-channel-TCH-FS-HS.html" target="_blank">**here**</a>.

####We now restart wireshark on the loopback interface and run the following command:

```bash
airprobe_decode.py -c voice_capture.cfile -s 1000000 -a 17 -m SDCCH8 -t 1 -e 1 -k 0x23,0x9E,0x4C,0x21,0x36,0x12,0xC0,0x00
```
####**-e 1** specifies the algorithm **A5/1**, **-k 0x23,0x9E,0x4C,0x21,0x36,0x12,0xC0,0x00** specifies the **Kc**.

####On wireshark we first look for the Call Control Setup traffic and we can actually see the calling party number as below.

{% img /images/cc_setup.png %}

####A bit down the capture we should see an Assignment command. We see that the voice call is assigned to Timeslot 7 and the Traffic Channel is full rate (TCH/F).

{% img /images/call_assignment.png %}

##**Decoding the voice traffic**

####We can now finally decode the voice traffic by running the following command:

```bash
airprobe_decode.py -c voice_capture.cfile -s 1000000 -a 17 -m TCHF -t 7 -e 1 -k 0x23,0x9E,0x4C,0x21,0x36,0x12,0xC0,0x00 -d FR -o speech.au.gsm
```
####**-m TCHF** specifies the traffic channel, **-t 7** the TCH/F timeslot, **-d FR** specifies the voice codec of the channel as full rate, and  **speech.au.gsm** specifies the output file.

####**speech.au.gsm** contains the voice traffic. We convert it to an audio file using toast as follows:

```bash
toast -d speech.au.gsm
```

####We will get a file called **speech.au** which we can play back and listen to the captured voice call :).

##**Alternative method**

####Alternatively one could use the mainstream airprobe modules instead of gr-gsm with the same results.
####The original modules had issues with the HackRF and later GNU Radio versions. I however did some patching and you can clone the patched version from my Github <a href="https://github.com/iamckn/airprobe" target="_blank">**here**</a>. 

####The equivalent commands for the whole process starting from decoding BCCH to decoding voice are:

```bash
./go.sh voice_capture.cfile 64 0B
./go.sh voice_capture.cfile 64 1S 239E4C213612C00001
./go.sh voice_capture.cfile 64 7T 239E4C213612C00001
toast -d speech.au.gsm 
```

####That concludes the GSM radio series for now. 
####Till next time, happy hacking!