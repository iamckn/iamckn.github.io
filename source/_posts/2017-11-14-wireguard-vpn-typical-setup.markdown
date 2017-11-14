---
layout: post
title: "Wireguard VPN: Typical Setup"
date: 2017-11-14 07:51:06 +0300
comments: true
categories: [vpn]
---

I recently discovered the awesome <a href="https://www.wireguard.com/" target="_blank">**Wireguard**</a> VPN tunnel and I was sold. Wireguard is a simple, kernel-based, state-of-the-art VPN that also happens to be ridiculously fast and uses modern cryptographic principles that all other highspeed VPN solutions lack.

<!--more-->

Openvpn used to be my VPN solution of choice but after a few weeks with Wireguard, things changed. See the performance comparision charts done by the Wireguard author, Jason Donenfeld.

{% img /images/wireguard_comparisions.png %}

Here are just a few of the reasons why Wireguard blows away the competition:

* It aims to be as easy to configure and deploy as SSH.
* It is capable of roaming between IP addresses (especially useful to prevent dropped connections when you have flaky internet).
* Uses state-of-the-art cryptography.
* It is meant to be easily implemented in very few lines of code, and easily auditable for security vulnerabilities.
* A combination of extremely high speed cryptographic primitives and the fact that WireGuard lives inside the Linux kernel means that secure networking can be very high-speed.
* Stealth - does not respond to any unauthenticated packets and both peers become silent when there's no data to be exchanged.

Hopefully you too have been sold so let's get into the set up process.

# Set up steps

1. Install WireGuard on the VPN server.
2. Generate server and client keys.
3. Generate server and client configs.
4. Enable WireGuard interface on the server.
5. Enable IP forwarding on the server.
6. Configure firewall rules on the server.
7. Configure DNS.
8. Set up Wireguard on clients.

# Set up details

We will be setting up the typical VPN connection described in the previous post.

{% img /images/typical_vpn.png %}

Here's how are set up will look like:

* An ubuntu 16.04 (x64) VPS as our VPN server (Gateway).
* The internet facing interface on the server is eth0.
* An ubuntu 16.04 (x64) computer as the client.
* We will use 10.200.200.1/24 as the VPN server interface IP.
* We will use 10.200.200.2/24 as the VPN client interface IP.
* Unbound DNS resolver for added security.

# 1. Install WireGuard on the VPN server

 Comprehensive details on Wireguard installation can be found on the official site <a href="https://www.wireguard.com/install/" target="_blank">**here**</a>.
 For our Ubuntu case the process is:

```bash
add-apt-repository ppa:wireguard/wireguard
apt-get update
apt-get install wireguard-dkms wireguard-tools linux-headers-$(uname -r)
```

# 2. Generate server and client keys

We will generate the following four files: server_private_key, server_public_key, client_private_key, client_public_key.

```bash
Umask 077
wg genkey | tee server_private_key | wg pubkey > server_public_key
wg genkey | tee client_private_key | wg pubkey > client_public_key
```

# 3.1 Generate server config

Create a file called **/etc/wireguard/wg0.conf** on the server and add the following content.

```bash
[Interface]
Address = 10.200.200.1/24
SaveConfig = true
PrivateKey = <insert server_private_key>
ListenPort = 51820

[Peer]
PublicKey = <insert client_public_key>
AllowedIPs = 10.200.200.2/32
```

**wg0.conf** will result in an interface named **wg0** therefore you can rename the file if you fancy something different.  

**AllowedIPs = 10.200.200.2/32** provides enhanced security by ensuring that only that a client with the IP **10.200.200.2** and the correct private key will be allowed to authenticate on the VPN tunnel .

**ListenPort** is the udp port to listen on. A different one can be used.

# 3.2 Generate client config

Create a file called **wg0-client.conf** on the client and add the following content.

```bash
[Interface]
Address = 10.200.200.2/32
PrivateKey = <insert client_private_key>
DNS = 10.200.200.1

[Peer]
PublicKey = <insert server_public_key>
Endpoint = <insert vpn_server_address>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 21
```

Similar to the server case, **wg0-client.conf** will result in an interface named wg0-client so you can rename the file if you fancy something different. 

**AllowedIPs = 0.0.0.0/0** will allow and route all traffic on the client through the VPN tunnel. This can be narrowed down if you only want some traffic to go over VPN.

**DNS = 10.200.200.1** will set the DNS resolver IP to our VPN server. This is important to prevent DNS leaks when on the VPN.

# 4. Enable the WireGuard interface on the server.

We will bring up the Wireguard interface on the VPN server as follows:

```bash
chown -v root:root /etc/wireguard/wg0.conf
chmod -v 600 /etc/wireguard/wg0.conf
wg-quick up wg0
systemctl enable wg-quick@wg0.service #Enable the interface at boot
```

After this confirm you have a new interface named wg0 by running **ifconfig**.

```bash
wg0       Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  
          inet addr:10.200.200.1  P-t-P:10.200.200.1  Mask:255.255.255.0
          UP POINTOPOINT RUNNING NOARP  MTU:1420  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

# 5. Enable IP forwarding on the server

Edit the file **/etc/sysctl.conf** and set the following line as:

```bash
net.ipv4.ip_forward=1
```

Then also do the following to stop having to reboot the server

```bash
sysctl -p
echo 1 > /proc/sys/net/ipv4/ip_forward
```

# 6. Configure firewall rules on the server

We will need to set up a few firewall rules to manage our VPN and DNS traffic.

Track VPN connection

```bash
iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

Allowing incoming VPN traffic on the listening port

```bash
iptables -A INPUT -p udp -m udp --dport 51820 -m conntrack --ctstate NEW -j ACCEPT
```

Allow both TCP and UDP recursive DNS traffic

```bash
iptables -A INPUT -s 10.200.200.0/24 -p tcp -m tcp --dport 53 -m conntrack --ctstate NEW -j ACCEPT
iptables -A INPUT -s 10.200.200.0/24 -p udp -m udp --dport 53 -m conntrack --ctstate NEW -j ACCEPT
```

Allow forwarding of packets that stay in the VPN tunnel

```bash
iptables -A FORWARD -i wg0 -o wg0 -m conntrack --ctstate NEW -j ACCEPT
```

Set up nat

```bash
iptables -t nat -A POSTROUTING -s 10.200.200.0/24 -o eth0 -j MASQUERADE
```

We also want to ensure that the rules remain persistent across reboots.

```bash
apt-get install iptables-persistent
systemctl enable netfilter-persistent
netfilter-persistent save
```

# 7. Configure DNS

A major issue with a lot of VPN set ups is that the DNS is not done well enough. This ends up leaking client connection and location details. A good way to test this is through the great http://dnsleak.com/ site.

We are therefore going to ensure that our DNS traffic is secure. After some research I came to the conclusion that the unbound DNS solution is a very good option to use. Some of its merits include:

* Lightweight and fast
* Easy to install and configure
* Security oriented
* Supports DNSSEC

We’ll set it up in a way to counter DNS leakage, more sophisticated attacks like fake proxy configuration, rogue routers and all sorts of MITM attacks on HTTPS and other protocols.

We first do the installation on the server

```bash
apt-get install unbound unbound-host
```

We then download the list of Root DNS Servers

```bash
curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.cache
```

Next we edit the following file **/etc/unbound/unbound.conf**

```bash
server:

  num-threads: 4

  #Enable logs
  verbosity: 1

  #list of Root DNS Server
  root-hints: "/var/lib/unbound/root.hints"

  #Use the root servers key for DNSSEC
  auto-trust-anchor-file: "/var/lib/unbound/root.key"

  #Respond to DNS requests on all interfaces
  interface: 0.0.0.0
  max-udp-size: 3072

  #Authorized IPs to access the DNS Server
  access-control: 0.0.0.0/0                 refuse
  access-control: 127.0.0.1                 allow
  access-control: 10.200.200.0/24	   		allow

  #not allowed to be returned for public internet  names
  private-address: 10.200.200.0/24

  # Hide DNS Server info
  hide-identity: yes
  hide-version: yes
     
  #Limit DNS Fraud and use DNSSEC
  harden-glue: yes
  harden-dnssec-stripped: yes
  harden-referral-path: yes

  #Add an unwanted reply threshold to clean the cache and avoid when possible a DNS Poisoning
  unwanted-reply-threshold: 10000000

  #Have the validator print validation failures to the log.
  val-log-level: 1

  #Minimum lifetime of cache entries in seconds
  cache-min-ttl: 1800	

  #Maximum lifetime of cached entries
  cache-max-ttl: 14400
  prefetch: yes
  prefetch-key: yes
```

I have commented the config file explaining the specific configuration details.

Finally we set some permissions, enable and test the operation on our DNS resolver.

```bash
chown -R unbound:unbound /var/lib/unbound
systemctl enable unbound
```

Here are some sample test results

```bash
nslookup www.google.com. 10.200.200.1

	Server:		10.200.200.1
	Address:	10.200.200.1#53

	Non-authoritative answer:
	Name:	www.google.com
	Address: 172.217.7.228

#Testing DNSSEC
unbound-host -C /etc/unbound/unbound.conf -v ietf.org

	[1510648646] libunbound[1739:0] notice: init module 0: validator
	[1510648646] libunbound[1739:0] notice: init module 1: iterator
	ietf.org has address 4.31.198.44 (secure)
	ietf.org has IPv6 address 2001:1900:3001:11::2c (secure)
	ietf.org mail is handled by 0 mail.ietf.org. (secure)
```

With that, our server setup is done :)


# 8. Set up Wireguard on clients

We can now finally set up our client. 

We begin by installing wireguard on the client depending on what platform we're on. The installation process is the same as the server's.

```bash
add-apt-repository ppa:wireguard/wireguard
apt-get update
apt-get install wireguard-dkms wireguard-tools linux-headers-$(uname -r)
```

If you are on Kali Linux, you may have to install **resolvconf** if you don't have it already.

We had already generated the **wg0-client.conf** client config in step **3.2**.
All we need to do is to move it to **/etc/wireguard/wg0-client.conf**.

We finally bring up our VPN interface by running the command:

```bash
sudo wg-quick up wg0-client
```

And voila, we have our Wireguard VPN tunnel in place.

```bash
wg0-client Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  
          inet addr:10.200.200.2  P-t-P:10.200.200.2  Mask:255.255.255.255
          UP POINTOPOINT RUNNING NOARP  MTU:1420  Metric:1
          RX packets:95 errors:0 dropped:0 overruns:0 frame:0
          TX packets:177 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:14236 (14.2 KB)  TX bytes:31516 (31.5 KB)
```

The wg command is a great Wireguard utility that you can use to view connection status.

```bash
sudo wg show
	
	interface: wg0-client
	  public key: FwdTNMXqL46jNhZwkkzWsyR1AIlGX66vRWe1HFSemHw=
	  private key: (hidden)
	  listening port: 39451
	  fwmark: 0xca6c

	peer: +lb7/6Nn8uhlA/6fjT3ivfM5fWKKQ2L+stX+dSq18CI=
	  endpoint: 165.227.120.177:51820
	  allowed ips: 0.0.0.0/0
	  latest handshake: 49 seconds ago
	  transfer: 11.41 MiB received, 862.25 KiB sent
	  persistent keepalive: every 21 seconds
```

You should now have a secure VPN connection in place. You can confirm this by checking your IP on sites such as https://whoer.net/.

Ensure you also run a DNS leak test on http://dnsleak.com/.

If you want to disconnect from the VPN you have to bring the VPN interface down.

```bash
sudo wg-quick down wg0-client
```

# Wrapping up

There are some points about the use of Wireguard that should be noted. 
First, your VPN connection will remain persistent across networks. This means that unless you specifically bring the interface down or shutdown the computer, you will always be on the VPN. Disconnecting and reconnecting to the same or a different network maintains the connection. Also note that a sleep or suspend of the computer will maintain the interface for when the computer becomes active again.

This is mostly a good thing as you'll maintain your VPN connection even when your internet is unstable or you switch networks. 

Another thing to note is that you can set up the VPN interface to be persistent across reboots by enabling it as a service.

```bash
sudo systemctl enable wg-quick@wg0-client.service
```

This means that you'll be always on VPN from the moment you boot up the computer.

You may also want to add new clients and the way you do this is as follows:

Generate new client keys on the server

```bash
wg genkey | tee new_client_private_key | wg pubkey > new_client_public_key
```

Then register them on the server

```bash
wg set wg0 peer <new_client_public_key> allowed-ips <new_client_vpn_IP>/32
```

Finally generate the new client config as described in step **3.2** and you can then set up your clients as per step **8**.

# Automation

I realised that having to go through all the steps when setting up a new VPN server will probably be a tedious process. 
I therefore automated the whole process using <a href="https://www.ansible.com/" target="_blank">**Ansible**</a>.

You can therefore quickly spin up a new Wireguard VPN on a new VPS in a few minutes.

You can access the details here https://github.com/iamckn/wireguard_ansible

Here are some useful links that have guided this post.

* Official Wireguard site https://www.wireguard.com/
* Unbound – Your own DNS Server https://freedif.org/unbound-your-own-dns-server/
* Installing Wireguard https://research.kudelskisecurity.com/2017/06/07/installing-wireguard-the-modern-vpn/
* Wireguard IPv6 set up https://danrl.com/blog/2016/travel-wifi/

The next post will be about how to up a chained Wireguard VPN connection.

Till then, happy hacking! 