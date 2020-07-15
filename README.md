# wifi-to-eth-rpi
Code and instructions to use RPi as a wifi entender.

# GOAL at the end:
![alt text](https://github.com/mehulwarade/wifi-to-eth-rpi/blob/master/goal.png?raw=true =100x20)

#Tested on:
OS: Dietpi v6.31.2
Hardware: RPi 3 Model B+ (armv7l)

## Dependencies Required:

1. dnsmasq [```sudo apt-get install dnsmasq```]
2. net-tools [```sudo apt install net-tools```]   <= required to use the "route option"
3. iptables [```sudo apt install iptables```]   <= Used for creating a bridge and routing traffic from wlan to eth

## Instructions (Easy):

#### Set your ethernet adapter to a static IP address
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

```conf
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig 
sudo nano /etc/dnsmasq.conf
```

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

#### Use iptables to configure a NAT setting to share the Wifi connection with the ethernet port
NAT stands for Network Address Translation. This allows a single IP address to server as a router on a network. So in this case the ethernet adapter on the RPi will serve as the router for whatever device you attach to it. The NAT settings will route the ethernet requests through the Wifi connection.

Run the following command per line in terminal:

```s
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT
```

#### Configure the dnsmasq settings
Edit /etc/sysctl.conf and uncomment this line:

net.ipv4.ip_forward=1

```sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward```

#### Next set up ip routing:
```conf
sudo ip route del 0/0 dev eth0 &> /dev/null
a=`route | awk "/${wlan}/"'{print $5+1;exit}'`
sudo route add -net default gw 192.168.2.1 netmask 0.0.0.0 dev eth0 metric $a
```

#### Restart the dnsmasq service

```sudo systemctl restart dnsmasq```

#### That's it! Plug ethernet cable into other device and it should start having internet connection


## References:
https://www.instructables.com/id/Share-WiFi-With-Ethernet-Port-on-a-Raspberry-Pi/ 

https://raspberrypi.stackexchange.com/a/50073 

https://github.com/arpitjindal97/raspbian-recipes/blob/master/wifi-to-eth-route.sh  

https://vpsfix.com/community/server-administration/no-etc-rc-local-file-on-ubuntu-18-04-heres-what-to-do/ 
