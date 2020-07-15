# wifi-to-eth-rpi
Code and instructions to use RPi as a wifi entender.

# GOAL at the end:
![alt text](https://github.com/mehulwarade/wifi-to-eth-rpi/blob/master/goal.png?raw=true)

#Tested on:
OS: Dietpi v6.31.2
Hardware: RPi 3 Model B+ (armv7l)

## Dependencies Required:

1. dnsmasq [```sudo apt-get install dnsmasq```]
2. net-tools [```sudo apt install net-tools```]   <= required to use the "route option"
3. iptables [```sudo apt install iptables```]   <= Used for creating a bridge and routing traffic from wlan to eth

## Instructions (Easy):

We need to configure interfaces. We will assign a static IP address to  eth0 which will be used as gateway. Open the interfaces file:
```sudo nano /etc/network/interfaces```

Edit the eth0 section like this:

```s
allow-hotplug eth0  
iface eth0 inet static  
address 192.168.2.1
netmask 255.255.255.0
network 192.168.2.0
broadcast 192.168.2.255
```

Next, we will configure dnsmasq. The shipped dnsmasq config file contains a lot of information on how to use it. So, I will advise to move it and create a new one.

```sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig ```
```sudo nano /etc/dnsmasq.conf```

Paste the following into the new file

```conf
interface=eth0      # Use interface eth0  
listen-address=192.168.2.1 # listen on  
# Bind to the interface to make sure we aren't sending things 
# elsewhere  
bind-interfaces
server=8.8.8.8       # Forward DNS requests to Google DNS  
domain-needed        # Don't forward short names  
# Never forward addresses in the non-routed address spaces.
bogus-priv
# Assign IP addresses between 192.168.2.2 and 192.168.2.100 with a
# 12 hour lease time
dhcp-range=192.168.2.2,192.168.2.100,12h 

```

git clone https://github.com/mehulwarade/wifi-to-eth-rpi
cd wifi-to-eth-rpi
sh wifi-eth.sh

#### That's it! Plug ethernet cable into other device and it should start having internet connection