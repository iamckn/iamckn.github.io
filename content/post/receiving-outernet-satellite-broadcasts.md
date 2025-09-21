+++
title = "Receiving Outernet Satellite Broadcasts"
date = "2016-12-29 10:18:31 +0300"
tags = ["radio"]
+++

Receiving satellite data is something I've been hoping to try out for a while. It's an area that captures the magic of long range radio communication and has become easier to experiment in as time has gone by. 
<!--more-->

This is where [**Outernet**](https://outernet.is) comes in. They are a global data broadcasting company that provides access to content all over the world through geostationary and low earth orbit satellites. They broadcast their signal over three Inmarsat satellites at a bitrate of about 2kbps or 20MB of content per day. Full information on the technology aspects can be found [**here**](https://outernet.is/tech/).

They provide a Do-It-Yourself kit for $99 that one can get up and running in a short amount of time. I already had the RTL-SDR V3 dongle and a Pi 3 so all I needed was their portable L-Band patch antenna and the Low Noise Amplifier. They've documented the set up process extremely well so I'll not dwell on it here. Their detailed guide is available [**here**](https://static1.squarespace.com/static/55ccfcb0e4b0c5b275895965/t/58325122579fb38c5719f2fb/1479692584995/Outernet+L-Band+Manual+v03.10.pdf).

Here's my assembled hardware set up.

![alt](/images/outernet_hardware.jpg)

Once the hardware is powered up, a wireless hotpspot called Outernet is created which you connect to and visit the page **my.outernet.is** on your browser. You'll be prompted for various initial configuraton details which you can also modify later to your liking. Of importance is the choice of the satellite to use, with three available. In my case. the one serving my region is **Alphasat** on the 1545.94 MHz frequency.

The next step is to point the antenna directly at the satellite for good signal reception. A good way to find the direction of the satellite using the  [**Satellite AR**](https://play.google.com/store/apps/details?id=com.agi.android.augmentedreality&hl=en)  Android app. Check the Tuner Settings on the web dashboard while positioning the antenna until you get a Signal to Noise Ratio of at least 3dB.

![alt](/images/outernet_tuner.png)

Data will start streaming in at the rate of 2kbps/20MB per day so give it some hours before you start exploring the downloaded content. Below is a sample of what the content includes after a week.  

## Amateur radio content from APRS and AMSAT  
News and messages from APRS and AMSAT.

![alt](/images/outernet_amateur.gif)

## Community content contributed by the public  
Content uploaded to outernet servers by the community. The public is allowed to submit content which may be incorporated after review.

![alt](/images/outernet_community.gif)

## Games
Browser based games.

![alt](/images/outernet_games.gif)

## Offline Wikipedia pages
Select offline Wikpedia pages. 

![alt](/images/outernet_wikipedia.gif)


## News
Up to date news from around the world.

![alt](/images/outernet_news.gif)


## Interactive weather data
And finally my favorite, a beautiful interactive 3D weather map that is simply amazing.

![alt](/images/outernet_weather.gif)

The Outernet service is constantly evolving and I am looking forward to more exciting content and functionality.

Till next time, happy hacking!
