+++
title = "Sniffing Gsm Traffic"
date = "2015-11-01 09:30:27 +0300"
tags = ["radio"]
+++
I have been playing around with the HackRF for the past couple of weeks and progressively exploring the Radio Frequency spectrum. In this post I'll take you through how to sniff GSM traffic. I'll be specifically monitoring the [**Um interface**](https://en.wikipedia.org/wiki/Um_interface). This in the air interface between the Mobile Station (MS) and the Base Transceiver Station (BTS). The MS in this case will be the mobile phone while the BTS is what the phone connects to on the Mobile network. The BTS is usually hosted on towers which you can spot in various locations. Here is what a typical one looks like.

<!--more-->

![alt](/images/BTS.jpg)

We first ensure that our HackRF is well set up as detailed in [**this earlier post**](https://www.ckn.io/blog/2015/10/11/the-hackrf-one-first-steps/).
This setup is tailored to Archlinux but other linux distros closely follow the same process.
Next we install [**gr-gsm**](https://github.com/ptrkrysik/gr-gsm) which consists of gnuradio blocks and tools for receiving GSM transmissions.

# Setting up gr-gsm

Install libosmocore libraries
```bash
yaourt -S libosmocore	
```

Install gr-gsm
```bash
git clone https://github.com/ptrkrysik/gr-gsm
cd gr-gsm
mkdir build
cd build
cmake ..
make
sudo make install
sudo ldconfig
```

Add the path of the gr-gsm blocks to gnuradio
```bash
nano ~/.gnuradio/config.conf 	
	[grc]
	local_blocks_path=/usr/local/share/gnuradio/grc/blocks
```

Move gr-gsm library and python modules to correct location (depends on your environment).
```bash
sudo mv /usr/local/lib/python2.7/site-packages/grgsm /usr/lib/python2.7/site-packages/
sudo mv /usr/local/lib/libgnuradio-grgsm.so /usr/lib/
```

Next we will set up kalibrate-hackrf which we will use to find what GSM frequencies to sniff on.

# Setting up kalibrate-hackrf
```bash
git clone https://github.com/scateu/kalibrate-hackrf.git
cd kalibrate-hackrf
./bootstrap
./configure
make
sudo make install
```

# Finding GSM frequencies
With our prerequisites done, the next step is to find GSM frequencies to listen on. First we consult [**this site**](http://www.worldtimezone.com/gsm.html) to identify GSM bands used by our local providers. 

![alt](/images/gsm_bands.png)

Let's concentrate on the 900MHz band for now.
We use kalibrate-hackrf (kal) to find the frequencies as below:

```bash
kal -s GSM900 -g 40 -l 40
```

`-s` is the band to scan, `-g` is the baseband gain and `-l` the interface gain
Let it run for a minute or so and you should get output similar to:

![alt](/images/kal.png)

# Sniffing!

Let us now use gqrx to confirm the frequencies by tuning in to the first one. In this case **936.4MHz**

![alt](/images/gqrx_gsm.png)

We can see that the activity is centered around **936.8MHz** so we adjust accordingly and use that as the frequency.
We now start capturing gsm traffic using gr-gsm :). What we do is launch gnuradio companion and open `the airprobe_rtlsdr.grc` block that is part of gr-gsm and run it.

```bash
gnuradio-companion airprobe_rtlsdr.grc
```

Tune to **936.8MHz** and if all went well we should get data streaming through the console window.

![alt](/images/gr_gsm_traffic.png)

We finally launch wireshark and monitor the loopback interface to see the decoded traffic :)
Here is a sample showing BTS Location details.

![alt](/images/gsm_wireshark.png)

So what next after sniffing the traffic? 
In subsequent posts I'll explain and show how to interpret the decoded traffic as I explain how GSM communication takes place.
From there I'll demonstrate how to decrypt the traffic. That is where it gets really interesting as we start reading SMS's, listening to calls and so on :).
Till then, happy hacking.
