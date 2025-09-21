+++
title = "The Hackrf One First Steps"
date = "2015-10-11 20:18:10 +0300"
tags = ["radio"]
+++
The wireless world is an area I've been interested in for a long time. From the the more common applications such as Wi-Fi, bluetooth and FM to the lesser explored such as radar, satellite and GSM, radio frequency is an area I plan to explore extensively. How awesome is the concept of [**electromagnetic pulses**](https://en.wikipedia.org/wiki/Electromagnetic_pulse) in this age that is driven by electromagnetism. I digress so let me get back on track, there will be several more posts to explore the possibilities.

<!--more-->

I recently acquired the awesome [**HackRF One**](https://greatscottgadgets.com/hackrf/), a Software Defined Radio peripheral capable of transmission or reception of radio signals from 1 MHz to 6 GHz. I added the [**Ham It Up v1.3 - RF Upconverter**](http://www.nooelec.com/store/ham-it-up.html) to boost performance in the lower frequency ranges. A telescopic antenna and various connectors complete the setup.

# The Hardware
![The HackRF One, telescopic antenna (ANT 500), some connectors and a Ham It Up v1.3 - RF Upconverter](/images/hackrf_hardware.JPG)

# Software setup
I did my setup on archlinux but it should be just as easy on other OS's. Here are the instructions from the [**HackRF Github wiki page**](https://github.com/mossmann/hackrf/wiki/Operating-System-Tips).
On my system I followed the following steps:

## Installation of various software

```bash
sudo pacman -S hackrf gnuradio-osmosdr gnuradio gnuradio-companion gqrx
yaourt -S gr-fosphor-git
```
## Creation of udev rules to allow access to the HackRF

```bash
sudo nano /etc/udev/rules.d/52-hackrf.rules
	ATTR{idVendor}=="1d50", ATTR{idProduct}=="6089", SYMLINK+="hackrf-one-%k", MODE="660", TAG+="uaccess"
	ATTR{idVendor}=="1fc9", ATTR{idProduct}=="000c", SYMLINK+="hackrf-dfu-%k", MODE="660", TAG+="uaccess"
```
## Testing the HackRF
Plug in the HackRF and run
```bash
hackrf_info
```
You should get output similar to:
```bash
Found HackRF board 0:
USB descriptor string: 000000000000000014d463dc2f45a7e1
Board ID Number: 2 (HackRF One)
Firmware Version: 2015.07.2
Part ID Number: 0xa000cb3c 0x0069435b
Serial Number: 0x00000000 0x00000000 0x14d463dc 0x2f45a7e1
```
If you get an error, recheck your udev rules and ensure the HackRF USB connection is ok.

## Updating the SPI Flash Firmware
Download the latest firmware [**here**](https://github.com/mossmann/hackrf/releases/tag/v2015.07.2).
```bash
wget https://github.com/mossmann/hackrf/releases/download/v2015.07.2/hackrf-2015.07.2.zip
unzip hackrf-2015.07.2.zip 
cd hackrf-2015.07.2/firmware-bin/
hackrf_spiflash -w hackrf_one_usb_rom_to_ram.bin
```
## Updating the CPLD
```bash
cd ..
hackrf_cpldjtag -x firmware/cpld/sgpio_if/default.xsvf
```
Press the reset button, then run the `hackrf_info` command and confirm the firmware updated successfully.
We are then ready to try the "Hello World" of Software Defined Radio, implementing an FM receiver.

# An FM Receiver
We will use the gnuradio-companion for this.
Download and save this [**file**](https://raw.githubusercontent.com/rrobotics/hackrf-tests/master/fm_radio/fm_radio_rx.grc), then open it using gnuradio from the terminal.
```bash
gnuradio-companion fm_radio_rx.grc
```
You can then tune in to your preferred radio frequency and listen in.
![alt](/images/fm_receiver.png)

With that out of the way, you can try building the FM receiver yourself using the gnuradio-companion.
With the HackRF's transmission capability, an FM transmitter can also be implemented. I won't say I haven't thought of jamming certain frequencies in the vicinity, but that would be illegal, wouldn't it :)?
In the next post, I'll show how to sniff gsm traffic.

Till then, happy hacking :).
