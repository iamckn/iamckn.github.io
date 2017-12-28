---
layout: post
title: "Wireguard VPN: Chained Setup"
date: 2017-12-28 07:33:38 +0300
comments: true
categories: [vpn]
---

Moving on from my previous <a href="https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup/" target="_blank">**post**</a> about setting up a typical Wireguard VPN connection, let's go through how to do a chained setup. I will show how to do both the typical chained Wireguard VPN connection and the one with selective routing as described in my earlier post <a href="https://www.ckn.io/blog/2017/11/12/vpns-an-opsec-primer/" target="_blank">**here**</a>

<!--more-->

# Set up details

Let us start with the typical Wireguard VPN chained connection.

{% img /images/chained_vpn.png %}

Here's how our set up will look like:

* An ubuntu 16.04 (x64) VPS as our first VPN server which we will refer to as the middleman (VPN Gateway One as shown in the diagram above).
* An ubuntu 16.04 (x64) VPS as our second VPN server which we will refer to as the gate (VPN Gateway Two as shown in the diagram above).
* An ubuntu 16.04 (x64) computer as the client.
* The internet facing interface on both VPN servers is eth0.
* We will use the 10.200.200.0/24 subnet for the network between the client and the middleman.
* We will use the 10.100.100.0/24 subnet for the network between the middleman and the gate.
* We will use 10.200.200.1/24 as the middleman client facing interface (wg0) IP.
* We will use 10.200.200.2/24 as the VPN client interface (vpn0) IP .
* We will use 10.100.100.1/24 as the gate VPN interface (wg0) IP.
* We will use 10.100.100.2/24 as the middleman gate facing interface (gate0) IP.
* Unbound DNS resolver for added security.

# Set up steps

1. Install Wireguard on the middleman.
2. Install Wireguard on the gate.
3. Set up a Wireguard VPN tunnel between the client and the middleman.
4. Set up a Wireguard VPN tunnel between the middleman and the gate.
5. Configure policy routing on the middleman to route traffic from the client to the gate.
6. Update the middleman gate facing interface (gate0) to allow all traffic from the gate to be allowed in the tunnel.
7. Confirm everything works as desired by doing a traceroute to the internet from the client.

# 1. Install Wireguard on the middleman.

I already went into the specifics of a typical Wirguard VPN setup in my previous post, so I'll not go into the details here.

I have shared ansible scripts on <a href="https://github.com/iamckn/chained-wireguard-ansible" target="_blank">**Github**</a> for the automation so I'll just run through the process here.

```bash
#Clone the repo on the client
git clone https://github.com/iamckn/chained-wireguard-ansible

#Move into the middleman folder
cd chained-wireguard-ansible/middleman/

#Edit the hosts file in that directory to change the IP to your middleman VPS

#Begin the installation process by running
ansible-playbook wireguard.yml -u root -k -i hosts

#If you're using an SSH key for authentication run this instead
ansible-playbook wireguard.yml -u root -i hosts

#Give it a few minutes and the server set up will be complete.
```

# 2. Install Wireguard on the gate.

The installation process is similar with the process as below. 

```bash
#Clone the repo on the client
git clone https://github.com/iamckn/chained-wireguard-ansible

#Move into the gate folder
cd chained-wireguard-ansible/gate/

#Edit the hosts file in that directory to change the IP to your gate VPS

#Begin the installation process by running
ansible-playbook wireguard.yml -u root -k -i hosts

#If you're using an SSH key for authentication run this instead
ansible-playbook wireguard.yml -u root -i hosts

#Give it a few minutes and the server set up will be complete.
```

# 3. Set up a Wireguard VPN tunnel between the client and the middleman.

We then set up Wireguard on our client. 

We begin by installing wireguard on the client depending on what platform we're on.

```bash
add-apt-repository ppa:wireguard/wireguard
apt-get update
apt-get install wireguard-dkms wireguard-tools linux-headers-$(uname -r)
```

If you are on Kali Linux, you may have to install **resolvconf** if you don't have it already.

Our Wireguard client config was generated during the middleman install process. The file is named **wg0.conf** and is in the home folder of the middleman. Use scp or whatever other method you prefer to copy it to your client. 

I'll rename mine to **vpn0.conf** and move it to **/etc/wireguard/vpn0.conf**.

We finally bring up our VPN interface by running the command:

```bash
sudo wg-quick up vpn0
```

We have our Wireguard VPN tunnel in place.

```bash
vpn0      Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  
          inet addr:10.200.200.2  P-t-P:10.200.200.2  Mask:255.255.255.255
          UP POINTOPOINT RUNNING NOARP  MTU:1420  Metric:1
          RX packets:110 errors:0 dropped:0 overruns:0 frame:0
          TX packets:241 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:46064 (46.0 KB)  TX bytes:51540 (51.5 KB)
```

You should now have a secure VPN connection in place. You can confirm this by checking your IP on sites such as https://whoer.net/. Your public IP should be that of the middleman.


# 4. Set up a Wireguard VPN tunnel between the middleman and the gate.

As it stands we have set up a standard VPN connection between the client and the middleman. The next step is now to set up a Wireguard VPN tunnel between the middleman and the gate.

In our scenario the gate will act as the server and the middleman as the client in that tunnel.

During the gate install process, a file named **gate0.conf** was generated and you should find it in the home directory of the gate. Copy it to the middleman using scp. 

Move the file to **/etc/wireguard/gate0.conf** on the middleman.

Bring up the VPN interface by running the command:

```bash
wg-quick up gate0
```

We now have our Wireguard VPN tunnel between the middleman and the gate in place.

```bash
gate0     Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  
          inet addr:10.100.100.2  P-t-P:10.100.100.2  Mask:255.255.255.255
          UP POINTOPOINT RUNNING NOARP  MTU:1420  Metric:1
          RX packets:1 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:92 (92.0 B)  TX bytes:180 (180.0 B)
```

# 5. Configure policy routing on the middleman to route traffic from the client to the gate.

We now have all the tunnels in place but traffic from the client will still go out to the internet through the middleman.

Normally all we need to do is modify our routes on the middleman to route traffic from the client to the gate. Wireguard however, using the **wg-quick** tool employs a variant of Rule-based routing using <a href="https://www.linux.org/docs/man8/tc-fw.html" target="_blank">**fwmark**</a>.

We are therefore going to configure policy routing to ensure traffic from the client is passed on to the gate by the middleman.

Do the configuration on the middleman as follows:

```bash
echo "1 middleman" >> /etc/iproute2/rt_tables
ip route add 0.0.0.0/0 dev gate0 table middleman 
ip rule add from 10.200.200.0/24 lookup middleman
```

# 6. Update the middleman gate facing interface (gate0) to allow all traffic from the gate to be allowed in the tunnel.

Next we need to allow all traffic coming from the gate to be allowed through the tunnel between the middleman and the gate. This is necessary to allow internet traffic that will be passed through the tunnel. Wireguard interfaces are strict in inspecting the origin of traffic that can be allowed to participate in the encrypted tunnel.

On the middleman run the following command:

```bash
wg set gate0 peer <peer_public_key> allowed-ips 0.0.0.0/0
```

Note that the **peer_public_key** in the command above can be gotten by running *wg show* on the middleman and checking the peer key on the **gate0** interface. 

# 7. Confirm everything works as desired by doing a traceroute to the internet from the client.

Before celebration, do various tests to confirm traffic from your client is going out through the gate and not the middleman.

Below is a traceroute to 4.2.2.2 confirming my traffic passess through the middleman and goes out through the gate to the internet.

```bash
traceroute to 4.2.2.2 (4.2.2.2), 30 hops max, 60 byte packets
 1  10.200.200.1 (10.200.200.1)  257.422 ms  258.769 ms  258.783 ms
 2  10.100.100.1 (10.100.100.1)  258.772 ms  258.752 ms  258.878 ms
 3	104.131.0.253 (104.131.0.253)  258.877 ms 104.131.0.254 (104.131.0.254)  259.595 ms  260.021 ms
 4  138.197.251.20 (138.197.251.20)  260.028 ms 138.197.251.26 (138.197.251.26)  260.029 ms 138.197.248.36 (138.197.248.36)  260.005 ms
 5  138.197.244.20 (138.197.244.20)  260.430 ms 138.197.244.28 (138.197.244.28)  260.841 ms  261.134 ms
 6  * * lag-103.ear4.Newark1.Level3.net (4.14.218.29)  259.196 ms
 7  * * *
 8  b.resolvers.Level3.net (4.2.2.2)  259.679 ms  260.234 ms  260.117 ms
```

Notice that **10.200.200.1** is the the middleman client facing interface IP and **10.100.100.1** is the gate VPN interface IP.

Don't forget to confirm your public IP on sites such as https://whoer.net/ and run a DNS leak test on http://dnsleak.com/.

We are done! 

#Selective Routing

What if we want to allow only certain traffic to go out to the internet through the gate and all the rest to go out through the middleman? The scenario is as below:

{% img /images/chained_selective.png %}

Let's put an example scenario to test. We want all our internet traffic to go out through the middleman and not be forwarded to the gate. We however desire to have an exception for traffic to 4.2.2.2, which we want to go out through the gate.

The only thing we will need to do is modify the policy routes we set up on the middleman as follows:

```bash
#Delete the route that forwarded all traffic to the gate
ip route del 0.0.0.0/0 dev gate0 table middleman 

#Add a route to forward 4.2.2.2 to the gate
ip route add 4.2.2.2/32 dev gate0 table middleman 
```

Doing traceroute tests should confirm our results.

```bash
#Normal internet traffic
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  10.200.200.1 (10.200.200.1)  256.786 ms  257.063 ms  257.094 ms
 2  104.131.0.253 (104.131.0.253)  257.827 ms 104.131.0.254 (104.131.0.254)  257.830 ms  258.201 ms
 3  138.197.248.52 (138.197.248.52)  258.208 ms  258.196 ms 138.197.248.32 (138.197.248.32)  259.771 ms
 4  138.197.244.34 (138.197.244.34)  259.040 ms  259.439 ms 138.197.244.32 (138.197.244.32)  259.409 ms
 5  72.14.194.62 (72.14.194.62)  259.692 ms  260.107 ms 162.243.191.255 (162.243.191.255)  260.110 ms
 6  * * *
 7  108.170.238.197 (108.170.238.197)  258.633 ms 108.170.238.195 (108.170.238.195)  258.223 ms 108.170.235.179 (108.170.235.179)  258.646 ms
 8  google-public-dns-a.google.com (8.8.8.8)  259.006 ms  259.652 ms  259.583 ms
```

Note that our traffic doesn't get routed through the gate interface (**10.100.100.1**).

```bash
#Traffic to 4.2.2.2 
traceroute to 4.2.2.2 (4.2.2.2), 30 hops max, 60 byte packets
 1  10.200.200.1 (10.200.200.1)  256.622 ms  256.789 ms  256.787 ms
 2  10.100.100.1 (10.100.100.1)  257.783 ms  258.323 ms  258.317 ms
 3  * * *
 4  * * *
 5  * * *
 6  * * *
 7  * * *
 8  b.resolvers.Level3.net (4.2.2.2)  259.457 ms  259.321 ms  259.270 ms
```

Note that traffic to 4.2.2.2 gets routed through the gate interface (**10.100.100.1**).

Modify the policy routes as need be to suit your needs.

I'll wrap up the VPN series in the next post by showing how to set up a portable VPN solution on a Raspberry PI.

Till then, happy hacking!