+++
title = "Wireguard Vpn Portable Raspberry Pi Setup"
date = "2017-12-28 16:44:50 +0300"
tags = ["vpn"]
+++

We often use wireless networks to access the internet. This may be at home, work or even places like restaurants. When we connect to these networks, the security of our internet traffic is under the control of the owner of the wireless network. This is also the case when we connect to a wired connection on a network we don't control.

<!--more-->

With this in mind, using a VPN on foreign networks is a good idea. Configuring and remembering to turn on VPN on the several mobile devices we carry around is often a hassle.

In this post, I'll detail how you can set up a portable VPN connection on a Raspberry Pi. You can carry it with you everywhere you go and have all your devices connect to it ensuring a secure connection. This also saves the work of configuring a VPN connection on all your devices.

Before we continue, you can go through my post on setting up a typical Wireguard VPN connection [**here**](https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup/).

# Set up details

Our intended setup is as below:

![alt](/images/portable_vpn.png)

Here's how our setup will look like:

* An ubuntu 16.04 (x64) VPS as our VPN server (Gateway).
* The internet facing interface on the server is `eth0`.
* A Raspberry Pi 3 Model B running Raspbian as our portable VPN client.
* The Pi will be connected to the internet via LAN `eth0` or an external USB wireless card `wlan1`.
* We will use the `10.200.200.0/24` subnet for the network between the Pi and the VPN Gateway.
* We will use the `10.100.100.0/24` subnet for the wireless network that the Pi will host for the clients on `wlan0`.
* We will use `10.200.200.1/24` as the VPN Gateway interface IP.
* We will use `10.200.200.2/24` as the Pi VPN interface IP.
* We will use `10.100.100.1/24` as the Pi wireless network interface `wlan0` IP.
* Unbound DNS resolver for added security.

# Set up steps

1. Install WireGuard on the VPN server.
2. Prepare the Pi and install dependencies.
3. Set up Wireguard on the Pi.
4. Set up the wireless network on the Pi.
5. Set up forwarding and NAT
6. Bring up the wireless network and test the setup.

# 1. Install WireGuard on the VPN server.

This is straightforward if you have gone through my guide [**here**](https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup/). I'll therefore run through the automated ansible process.

```bash
#Clone the repo onto your computer
git clone https://github.com/iamckn/wireguard_ansible

#Move into the repo directory
cd wireguard_ansible

#Edit the hosts file in that directory to change the IP to that of your VPN Gateway

#Begin the installation process by running
ansible-playbook wireguard.yml -u root -k -i hosts

#If you're using an SSH key for authentication run this instead
ansible-playbook wireguard.yml -u root -i hosts

#Give it a few minutes and the server set up will be complete.
```

The VPN gateway will be set up to use [**unbound**](https://www.unbound.net/) to provide secure DNS to the VPN network. We will however need to modify the unbound dns configuration to account for the wireless network the Pi will host.

Edit the file `/etc/unbound/unbound.conf` and add the following two lines to the file:

```bash
#allow pi wireless network to use the unbound dns server
access-control: 10.100.100.0/24	   		allow
 
#protect the pi wireless network subnet from public internet names resolution attempts
private-address: 10.100.100.0/24
```

Restart the DNS server for the changes to take effect

```bash
systemctl restart unbound
```

# 2. Prepare the Pi and install dependencies.

We now move to the Pi to install some required dependencies. First ensure that your Pi has the latest raspbian OS installed, then update it and install the following dependencies:

```bash
sudo apt update && sudo apt-get upgrade
sudo apt-get install hostapd dnsmasq libmnl-dev linux-headers-rpi build-essential git dnsutils bc raspberrypi-kernel-headers iptables-persistent
```

# 3. Set up Wireguard on the Pi.

We then set up Wireguard on the Pi. Install Wireguard from source as follows:

```bash
git clone https://git.zx2c4.com/WireGuard
cd WireGuard/src
make
sudo make install
sudo modprobe wireguard
```

Copy the file named `wg0.conf` from the home folder of the VPN server to the Pi. Use scp or whatever other method you prefer then move it to `/etc/wireguard/wg0.conf` on the Pi.

Bring up the Wireguard interface on the Pi and enable it to start on boot:

```bash
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0.service
```

The VPN tunnel between the Pi and the VPN Server should now be up and running. You can confirm this by checking the public IP on the Pi using the following command:

```bash
curl ifconfig.co
```

# 4. Set up the wireless network on the Pi.

We now need to set up the Pi to host a wireless network through which other clients can connect. We will use `hostapd` to run the wireless network and `dnsmasq` for DNS and DHCP.

Edit the following line in the file `/etc/default/hostapd` as follows:

```bash
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

Create the following file `/etc/hostapd/hostapd.conf` and edit it as follows:

```bash
interface=wlan0
hw_mode=g
channel=10
ieee80211d=1
country_code=BZ
ieee80211n=1
wmm_enabled=1

ssid=<your ssid>
auth_algs=1
wpa=2
wpa_key_mgmt=WPA-PSK  
rsn_pairwise=CCMP
wpa_passphrase=<password of the ssid>
```

Modify the field `ssid` and `wpa_passphrase` to the name you want to use for your wireless network and the wireless password respectively.

Next we set up the various network interfaces on the Pi by editing the file `/etc/network/interfaces` and adding the following:

```bash
auto wlan0
iface wlan0 inet static
  address 10.100.100.1
  netmask 24

allow-hotplug eth0
iface eth0 inet dhcp

allow-hotplug usb0
iface usb0 inet dhcp

allow-hotplug wlan1
iface wlan1 inet dhcp
   wpa-driver wext
   wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```

`wlan0` is set to the IP `10.100.100.1/24` and is the gateway that will be used by wireless clients connecting to the Pi.
All the other interfaces are set up as possible internet facing interfaces depending on which one is connected to the internet.

You may need to connect your Pi to different wireless networks using an external wireless USB card. If not, skip the following step, otherwise edit the file `/etc/wpa_supplicant/wpa_supplicant.conf` and add the following:

```bash
network={
    ssid="network 1"
    psk="password to network 1"
    id_str="w"
    priority=1
}

network={
    ssid="network 2"
    psk="password to network 2"
    id_str="z"
}
```

You can add all the wireless networks you need to connect to to the file following the same format. Whenever you plug in an external wireless USB card, the Pi will scan for available networks and attempt to connect using the details in the file.

Let's now set up DHCP and DNS to serve the wireless network the clients connecting to the Pi will use.

We are going to use dnsmasq so let's first disable operation of the default raspbian dhcp server on the wlan0 interface. 
Edit the file `/etc/dhcpcd.conf` and add the following line:

```bash
denyinterfaces wlan0
```

We next back up the current dnsmasq configuration file:

```bash
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig 
```

We then configure dns by recreating the file `/etc/dnsmasq.conf` and editing it as:

```bash
dhcp-authoritative
interface=wlan0
listen-address=10.100.100.1
dhcp-range=10.100.100.50,10.100.100.150,12h
read-ethers
bogus-priv
domain-needed
dhcp-option=option:dns-server,10.200.200.1
```

The `dhcp-range` option determines the range of IPs clients connecting to the Pi will be allocated so you can modify it to suit your needs. Also note that the `dns-server` option is set to the VPN Server (Gateway) interface that we set up earlier.

# 5. Set up forwarding and NAT

To enable wireless clients to access the internet through the VPN connection between the Pi and the VPN Server, we need to do the following:

Uncomment the following line in `/etc/sysctl.conf`

```bash
net.ipv4.ip_forward=1
```

Then run the following commands:

```bash
sudo sysctl -p
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Finally set up the necessary NAT rules and make them persistent:

```bash
sudo iptables -t nat -A POSTROUTING -o wg0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0 -o wg0 -j ACCEPT
sudo iptables -A FORWARD -i wg0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT

sudo systemctl enable netfilter-persistent
sudo netfilter-persistent save
```

# 6. Bring up the wireless network and test the setup.

We now complete the network by starting the necessary services and bringing up the wireless network.

```bash
sudo ifup wlan0
sudo service dnsmasq start
sudo service hostapd start
sudo update-rc.d hostapd enable 
```

We are finally done!

Test the set up to ensure everything works. Start with a test of DNS operation:

```bash
dig www.google.com 10.200.200.1
```

Then check to see if the wireless network you set up is available and connect to it with a wireless client. If all went well you should have a secure VPN connection from your wireless client, to the Pi and then through the VPN server (Gateway). Don't forget to run a DNS leak test on http://dnsleak.com/.

You now have a portable secure VPN setup on your Pi that you can carry around and use. Just connect the Pi to the network through the LAN interface, external wireless USB card or even USB ethernet. Your devices can then connect to the VPN through the Pi's wireless network hosted on it's internal wireless interface (wlan0).

If you prefer to do a similar setup with everything happening over ipv6, refer to this great write-up https://danrl.com/blog/2016/travel-wifi/.

That concludes the VPN series! 

Till next time, happy hacking!
